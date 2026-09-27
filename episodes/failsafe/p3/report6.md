# failsafe p3 — step 6: the consolidated path, proven live

All trials ran on agstudio on 2026-09-27 (UTC). They used the real Front,
autolab (Claude Code supercoder and superdirector) and Observer, on the test
bed `pj-robustp1`. Before any change, `nctl status` was ok and
`nctl drift --host agstudio` converged; after the last rollout, it still
converged.

The Omni Agent stood in for the Developer at the requester's points and
injected and removed the faults. Where it did more than that, the tables
below say so.

## Settings

| | operational | accelerated (trial C only) |
|---|---|---|
| look interval | 60 s | 30 s |
| `probe_after` | 120 s | 60 s |
| `ask_after` | 180 s | 60 s |
| `escalate_after` | 600 s | 180 s |
| `wait_idle` | 900 s | 120 s |

- The accelerated values were written to `agobserver/.local/timing.json`
  at 05:17Z and removed at 05:31Z.
- Trials A, A2, B and D ran on the operational values. The five-minute
  target was measured only on those.

Trial aids, all removed afterwards (`faults/` directories empty, no
`timing.json`):

- autolab: `silent-exit` (A, A2);
- Observer: `probe-slow` = 8 (A–A2), and `review-exit` (C);
- Front: `reply-unusable` = 1 (A) and = 2 (B).

## Results against the plan's table

| trial | required outcome | measured | met |
|---|---|---|---|
| **Nested code fences and text after them** (A, A2) | the complete intended reply reaches its recipient | #12695: a bare-fenced test tail, a ```` ```diff ```` block, and the summary after both, complete. #12885 (A2): the same shape. The first attempt, #12680, was complete up to Zulip's size limit and then cut by the poster: 200+ verbose test lines. The cut was visible and Front asked for a repost. Fixed at the boundary: the reply guide now states the post size (pyagag `cd9cce3`), and the repost fitted | ✓ after the size guidance |
| **Front ends without a usable reply** (A: repaired; B: unrepairable) | repaired or escalated within a stated bound; no silent completion or duplicate work | **A** (fault = 1, on the serving that recorded the mission acceptance): acceptance recorded 05:13:43 (#12733); failure line 05:13:49 as `progress` without `end=`; re-served 05:14:09; reply 05:14:17 (#12739). **28 s from failure to repaired reply.** The acceptance was not recorded twice. **B** (fault = 2): question 05:14:45; failures 05:15:11 and 05:15:42 (the last closing, `report`); owners named by Observer 05:16:45 — **63 s after the last failure, 120 s after the question**; one incident, one review occurrence; Front was not asked a third time | ✓ |
| **Agreement reaches the plan** (A) | correct routing or explicit correction; no unsupported closure; actual acceptance completes close-out | agreement posted in the plan (#12700). The planner, reading `status.md`, said it closes nothing and pointed to the task topic; the listener's line from the record followed (#12702). Agreement in the task topic (#12703) closed task 1 at 05:09:09; task 2 started by itself; mission acceptance recorded on #12729 | ✓ |
| **Quiet live wait, and a stuck wait with repetitive activity** (A task 2; C) | live `waiting` is observed; healthy work continues; the stuck case gets bounded investigation | **Live `waiting`** for the first time: A's task 2, a 200 s foreground command, probed at 147 s and 207 s: "waiting … on Bash …, within its bound" (0.02 s CPU). No suspicion; normal completion. **C:** a subagent's 420 s wait with only `tool_progress` housekeeping. It was `waiting` at 73–164 s, then uncertain once nothing advanced for 120 s. Front was asked at +61 s (#12812) and investigated without starting anything (#12815). The developer was told at +181 s (#12817). The wait ended at 05:29:05 and the incident read "moving again" | ✓ |
| **Several probes time out** (A, A2) | an unrelated stopped task still meets its detection target | 8 probes per look taking their whole 10 s timeout ran beside the real ones. The looks lasted **0.3–0.6 s**. The stopped task reached Front in **168 s (A)** and **144 s (A2)**, against the 240 s detection target and the 300 s goal | ✓ |
| **Reported incident later recovers, with restart during update** (C, also A and B) | its review shows the later outcome once, with original evidence retained | C: `review-exit` armed after the "not recovered" occurrence 4 (#12819). Observer exited right after posting "Occurrence 4 — later: … moved again" (#12826), before recording it. Restarted by launchd, it recorded the update with **no second post** (count 1). The final `done` was added once later (#12839). A's occurrence 3 and B's occurrence 1 also gained "moved again" and final updates, once each. The original posts are unchanged | ✓ |
| **Retention and cleanup followed by restart** (D) | completed trials stay retired, active/held work survives, new requests are discoverable | Before and after an Observer restart the tracked set was the same `o11711`, `o12741`, `o8512`: two holds, and B, whose question was still owed. Retirements (`o11450`, `o11522`) stayed. A, C and A2 (done) were not tracked. A new request (A2) was discovered and handled. B left tracking at the first look after its question was answered (05:50:26) | ✓ |

## The end-to-end silent-exit recovery (operational)

**Trial A did not meet it.**

- Exit at 04:48:56, Front asked at 04:51:44 (168 s).
- Front then answered that the work was "moving again", and did not
  resume it (#12665). It had read its own acknowledgement in its own
  conversation (#12664) as the task's new serving.
- Observer's request made this easy: next to "health check: **stopped**
  … process exited" it also said "a serving is open … nothing has said it
  ended" and "it may be a long job … do not start a second run beside
  it".
- The bounded fallback worked: the developer was told at 05:01:45 (+601 s
  from the first suspicion) in the incident topic and the review. The
  stand-in developer then asked Front to resume (#12672). This is a
  developer intervention after escalation, not a rescue.

Fixes:

- agobserver `920b651`: a health-checked request states the check's fact
  ("the serving acknowledged at #… is over — the health check found its
  run's process gone") and no longer adds the conversation-only doubt.
  Test: `test_a_confirmed_stop_is_not_contradicted_in_the_request`.
- Front's three guides: work has moved again only if its owner posted in
  that conversation after the stop; your own acknowledgement is not that.

**Trial A2, after the fixes: met.**

| stage | time | from the exit |
|---|---|---|
| harness killed (`silent-exit`, pid 22252) | 05:40:54 | — |
| Observer: probe `stopped`, Front asked (#12874) | 05:43:18 | **144 s** |
| Front resumed the task in its own topic (#12879), started nothing else | 05:43:51 | **177 s** |
| resumed serving's result (#12885) | 05:44:56 | 4 min 02 s |
| requester's agreement in the task topic (#12887); close-out, fast-forward integration and push | 05:45:29 | |
| incident rescued | 05:46:18 | |
| mission acceptance recorded by Front on the requester's post (#12904 → #12907) | 05:46:52 | |
| developer follow-up: `review-autolab-stopped` occurrence 4 (#12901), recovered, marked as a trial with its assessment; final `done` appended (#12912) | 05:46–05:47 | |

No Omni Agent rescue; explicit acceptance; accurate follow-up. One wording
issue: Front said "this task is closed" 13 s before the close-out record
existed (#12889), inferring it from the agreement. Its guides now say a task
is closed when its record says `completed`.

## Other findings during the trials

- **Duplicate work (C).** After the agreement, the task serving re-ran the
  whole 420 s wait (05:31:06–05:38:13) before writing `report.md`. Its
  first run had written the file outside the repository. Nothing external
  was repeated, but a serving was spent. The supercoder guide now says an
  agreement closes what was shown, never asks for the work again, and
  only a wrong part is fixed and shown.
- **A bare `sleep` is refused by Claude Code's Bash tool.** C's first
  three task servings stopped on it and asked. The task text was adjusted
  in place through the planner (#12801 → #12805). This was a trial-design
  issue, not a product defect.
- **Front's first `agentchat accept` in C named the wrong id** and was
  refused with the reason (#12841). The retry recorded it correctly.
- **The incident text for `unanswered`** showed only the mention (B). It
  is fixed in pyagag `975895e` (the notice's own line).
- **Recovery timing in the review.** A2's occurrence says "recovery 5 min
  23 s from onset". That is when the monitor's look confirmed it; the
  resumed serving's first work was at 05:43:52, its result at 05:44:56.

## Measures

| measure | value |
|---|---|
| silent exit → Front (operational) | A 168 s, A2 **144 s** (goal 300 s) |
| silent exit → resumed without rescue | A2 **177 s**; A not resumed by Front (defect, fixed) |
| failed reply → repaired | 28 s (A) |
| last failed reply → owners told | 63 s (B) |
| look duration with 8 timing-out probes | 0.3–0.6 s |
| false interventions (asking about healthy work) | **0 operational.** In C, asking about a quiet unbounded wait is the policy under test (accelerated `wait_idle`), not a false positive; operationally it applies after 15 min |
| duplicate actions | recovery requests 0, review posts 0 (across the injected exit), acceptances 0; **1 duplicate task execution** (C's re-run wait; guide fixed) |
| human / Omni Agent interventions | stand-in requests, agreements and acceptances; the post answering A's escalation (#12672); in C, the command corrections (#12784, #12797, #12801, #12808); fault injection and removal. **No rescue before an escalation** |
| model cost | trials (04:48–05:51Z): autolab supercoder 14 runs $1.77, superdirector 5 runs $0.60, Front 26 runs $2.90: **$5.27**. Cleanup cancellations: $0.16. Observer: 0 judgments |

## Tests after the last change

| suite | passed |
|---|---|
| pyagag | 969 |
| agautolab | 318 |
| agobserver | 169 |
| agfront | 181 |
| agforge | 265 |
| archsage | 37 |
| cagent | 204 |

## Rollout and residue

- pyagag `975895e` everywhere a conversational role runs. All listeners,
  the gateway, the forge service and cagent-api were kickstarted at
  05:56:15Z with no run in flight.
- Trial missions m12639, m12762 and m12855 are `done`. The trial
  conversations `front-failsafe-p3-{a,b,c,a2}` are left open as the record
  of this step. All their requests are finished or answered, and none is
  tracked.
- Review statuses for this phase's occurrences were recorded with
  `review_status` by the Omni Agent. The reviews stay open for the
  Developer's ✔:
  - `review-autolab-stopped`: 4 occurrences;
  - `review-autolab-uncertain`: 4;
  - `review-front-unanswered`: 1.
