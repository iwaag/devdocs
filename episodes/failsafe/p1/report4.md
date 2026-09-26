# failsafe p1 — step 4: bounded failure trials

All trials ran live on agstudio (2026-09-26 UTC) with real in-system agents:
Front, autolab (Claude Code supercoder) and Observer with its local triage
model. They used disposable missions in `pj-robustp1` (`wordcount.py`).
The Omni Agent stood in for the Developer at the requester's two points: the
opening request and the final "I accept the mission". It injected faults and
restarted Observer. It did nothing to rescue any trial.

Before changing services: `nctl status` ok and `nctl drift --host
agstudio` converged (twice, 15:47 and 16:03).

## Fault injection

The m11741 shape (a serving ends on "the rest is still running, I'll report"
with nothing running) is made by two one-shot autolab fault files, created
only by a person (`agautolab` `zulip_listener`):

- `faults/stop-mid-task`: the next task prompt gets a trial instruction.
  Do the first real step, do not finish, and reply as **progress** that the
  rest is still running.
- `faults/stop-mid-task-unmarked`: the same, but with a reply that declares
  **no intent**, so it reads as the task's answer. This is the
  misclassification trial, in place of Zulip's truncation, which `compose`
  now prevents.

The cause differs from m11741 (an instruction, not the harness). The
recovery does not depend on the cause.

## Detection targets

Set from the 120 s look interval before the trials (`DETECTION_TARGET`):

| kind | target |
|---|---|
| `unheld` | 420 s |
| `quiet` | 2070 s (1800 s quiet + interval + the 150 s judgment ceiling) |
| `silent` | 2970 s |

## Results

| trial | stall (last word) | Observer asked Front | detection | resumed | work evidence | rescue recorded | mission `done` |
|---|---|---|---|---|---|---|---|
| **T1** worker ends after a progress post (m11804) | 16:17:55 #11818 | 16:23:25 #11828 (`unheld`, no judgment) | **330 s** ✓ | 16:23:36 (Front posted into the task) | 16:24:05 completed, integrated | 16:25:25 | 16:25:48 |
| **T2** healthy work longer than the review interval (m11895) | — (one serving, 16:27:31 → 16:39:47, 6-minute silences) | never | no incident ✓ | — | 16:39:47 report | — | 16:40:28 |
| **T3** progress misclassified as an answer (m11866) | 16:26:41 #11880 (unmarked) | 17:01:15 #11949 (`quiet`, judged `stall`) | 2074 s ✗ by 4 s (a duplicate mission incident's failed judgment queued first) | 17:01:24 | 17:02:53 completed (after a `returned` overlap with T2, merged) | 17:03:45 | 17:04:05 |
| **T3b** repeated after the fixes (m12016) | 17:14:25 #12031 (unmarked) | 17:47:30 #12041 (`quiet`, judged `stall` in 64 s) | **1985 s** ✓ | 17:47:40 | 17:48:08 completed, continued from the partial helper | 17:49:31 | 17:49:44 |
| **T4** Observer restarts | restarted 16:27:13, during tracking (T3 stalled, not yet due; T2 running) and 17:01:44, during recovery (T3 asked 1/2, not yet rescued) | — | T3 still detected after the first restart | no second request after the second; the retry schedule and the attempt count survived | — | 17:03:45 "after 1 request" | — |

Totals over the trials:

| measure | value |
|---|---|
| false interventions | 0 — no request about T2; one extra, needless incident in T3 (see D1), which asked nobody |
| duplicate executions or side effects | 0 — each stop was resumed by one serving; T2's long serving was never touched |
| retained outputs | T3b's resumed serving kept the stopped one's `is_dotted` and did not redo it; T3's stopped serving had left nothing (it only read code), so the work was done afresh |
| human interventions | none beyond the requester's own posts (request, mission acceptance) |
| Omni Agent rescue | none |
| cost | autolab supercoder runs 0442–0452: **$1.10** in total; Front and Observer runs not summed |

## Defects found, fixed, and re-tried

- **D1: `quiet` opened incidents on both the mission and its task** (T3).
  The mission's judgment failed (exit 2, `unclear`) and held the task's
  judgment for 130 s. Fix (pyagag `f5c4359`): `quiet` asks only about the
  deepest unfinished unit. In T3b there was one incident and the target was
  met.
- **D2: Front handed its replies to Observer** (T1, T3). The listener hands
  a reply to the last other speaker, which was Observer's request. So Front
  asked Observer (`to=23`) to confirm acceptance. That pending "question to
  a person" also read as a human wait, which suppresses `quiet` and
  `unheld`, a failsafe hole. Fix:
  - Observer's request declares `intent=report answer=none`;
  - `requester_of` never hands a reply to a post that answers nothing
    (pyagag `f5c4359`, agobserver);
  - in T3b Front addressed the Omni Agent (`to=9`).
- Both fixes were deployed to every pyagag consumer. The consumers' tests
  all pass, and every listener was restarted at 17:13 before T3b.

## Observations kept open

- **A resume post can also close the task.** In T1 the resumed supercoder
  took Front's "continue and report" as the requester's agreement: it
  wrote `report.md` and the task closed in the same serving. Front had been
  authorised to agree, so the outcome was right, but the agreement was
  inferred rather than given.
- Front's first `agentchat send` in T1 was refused (`to=` without a request),
  and its retry in the same serving succeeded (`[opfail]` recorded).
- Plain conversations never get a terminal record, so their requests stay
  tracked indefinitely (o11771, the aborted first T1 whose feature already
  existed). This is harmless (nothing is asked) but unbounded.
