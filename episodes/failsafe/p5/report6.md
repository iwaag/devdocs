# failsafe p5 — step 6: autonomous completion, validated and consolidated

Date: 2026-09-27 13:00Z – 2026-09-28 16:10Z (UTC; the realm's clock). The
Omni Agent stood in for the Developer: it posted the requests, gave the
reserved approval in trial G, and answered Front's question there.

## The trials

| trial | code (pyagag) | what | result | Observer posts | human input after the requests | cost |
|---|---|---|---|---|---|---|
| C | `a44850a` | growbox + worldtrend at once | **both completed** 13:17:58 / 13:23:10 (15 / 20 min) | 0 | none | $8.95 |
| D | `ab8c98a` | the pair again | both completed 14:00:53 / 14:03:09 | **3 false `undelivered`** → fixed | none | $8.43 |
| E | `ab8c98a` + Observer fix | the pair again | worldtrend completed 14:27:42; growbox **assisted**: developer fix `1c7937d` deployed mid-run, completed 14:36:25 | 1 correct (2 asks, then reported) | none | $9.83 |
| F | `1c7937d` | the pair again | **both completed** 15:02:10 / 15:02:36 | 0 | none | $8.38 |
| G | `1c7937d` | reserved approval, a stop, a long queue | reserved: waited 30 min for the person, recorded their words; stop → Front in 129 s; queue: 1 **false `unacknowledged`** → fixed | 1 correct (stop), 1 false | planned approvals; one explanation Front asked for | $4.77 |
| H | `e9e6229` (final) | the pair again | **both completed** 16:06:09 / 16:06:57 (19 / 20 min) | 0 | none | $8.84 |

The two concurrent-study trials on the final implementation are **F and
H**. F ran one commit earlier than H, and that commit (the roster owner)
affects only a first post that names nobody, which never occurs in a study
pair. H passed on the exact final code. C also passed, on earlier code.
The whole of step 6 cost **$49.20**. Each trial is an autolab + Front +
archsage sum over its time window; Observer's judgments run on the local
model.

### The required-results table

| scenario | required result | evidence |
|---|---|---|
| Healthy serial queue beyond five minutes | panel explains the wait; Observer issues no false recovery request | H: growbox's agreement (#14915, 15:55:58) waited ~6 min behind worldtrend's task. Observer's own record `q14853`: **behind, confirmed** at 344 s ("queued 344 s (position 1 of 1) behind work-m14876/workrun-task1-m14876, whose serving is running"), no post. D: worldtrend's task start waited 8 min 43 s, `q14094` behind, confirmed at 495 s, no post. The panel said the same, e.g. C "queued 256 s … behind work-m13901/workrun-task1-m13901, whose serving is running" (board.jsonl) |
| Blocker stops, or finishes but queued work is not picked up | bounded detection names the actual problem; no indefinite exemption or competing run | **accelerated/test only** (labelled as such): `probe_queue` over a real listener queue (idle executor → `stopped` after 90 s; passed over by 3 servings; blocker stopped → `unknown` with the blocker named) and Observer `test_p5_queue.py` (blocker stopped in another request: that request asked, the queued one not, `unserved` reported after `escalate_after`; failed check → plain rule; `unknown` then `queued` → `excused`). Live, the stopped *task* of G2 was detected directly (below); no live blocker stopped with a post queued behind it |
| Front vs autolab records the same acceptance | same decision and final record; repeats do not repeat work | tests: Front then autolab on the same post, the second writes nothing. Live: every mission of C–H was recorded by Front on its own agreement (`[acceptance] #<Front's post> by 15 (Front) after=#<shown>`), with no planning run spent to note it, except E worldtrend, where Front posted into the workplan first (one extra planning run) |
| Human explicitly reserves approval | work reaches that wait and stays open until the human agrees | G1: Front wrote `[selfnote][approval] reserved 9 (Omni Agent) #14608` itself; after the result (#14653) it showed it and asked (#14655); the panel read "waiting — #14655 asks Omni Agent" 15:13–15:43; nothing was recorded until the stand-in's #14808; the record is `#14808 by 9 (Omni Agent) after=#14653` |
| Routine continuation and interrupted close-out | real serving resumes when needed; end and delivery complete once without a nudge | every run of C–H ended with its end record, one report and one `[delivered]` note, none nudged. D worldtrend continued its run from the desk with `agrun continue` (#14213/#14214, the start note served). Growbox in D, E and F ended from the desk with `agrun finish` (D, E) or by the run itself (F, H). Interruption: step 4's tests at each write; startup recovery completed B growbox's pre-p5 close-out (once, after the fix) |
| Two runs of the same study | refresh evidence and replies reach the correct run and satisfy the correct revision | growbox ran five times (C–H) and worldtrend five times. Ten refreshes went to ten topics of their runs' own, and no topic received a later run's post. Each `sagesync` records `for=` its run (E growbox: its desk, which asked) and `includes=` its integrated commit. Every card's `knowledge_refreshed` completed on its own record. Reordered and overlapping refreshes: `test_two_runs_of_the_same_study_each_need_their_own_refresh_in_any_order` |

### Stop timing (operational, health-covered owner)

G2's task serving was killed by the one-shot `silent-exit` fault at its
first tool call:

| time | event |
|---|---|
| 15:24:31 | harness killed (exit −9) |
| 15:26:40 | Observer's request to Front: **129 s** after the stop (goal: 300 s) |
| 15:26:55 | Front's `recheck` → STOPPED, resume posted in the task's own topic |
| 15:26:55 | autolab acknowledged |
| ~15:34 | the task finished |

Mission m14673 was completed and accepted. There was no second run beside
it, and review occurrence 8 is marked as a trial.

## What failed, and what was changed during the step

| seen in | what happened | fix | commit |
|---|---|---|---|
| C | a not-started task shown "behind" another serving on the panel | the queue reading applies only to a post awaiting its ack | pyagag `ab8c98a` |
| C–H | Front tried `--to archsage-agstudio1` (an instance name) up to three times per run | the refusal names the account the name is part of | `ab8c98a` |
| D | **3 false `undelivered`**: close-out answers waited in **Front's** serial queue behind Front's run of the other study (up to 5 min), and the archsage answer likewise. Each bought Front a desk serving | Observer reads an answer awaiting its requester like a queued post (conversation evidence of the requester's open serving) | agobserver `QUEUED_KINDS` |
| D | a late refresh answer reached an already-ended run; relayed to the desk, one more desk serving (14:10) | consequence of the false requests (the desk ended the run early); none since | — |
| E | Front posted its task agreement to `#pj-growbox › work-m14270/workrun-task1-m14270`, a new topic nobody serves. `recheck` then said ASKED, so Front waited on nobody | `agentchat send` refuses a `<channel>/<topic>`-shaped topic that names a real channel; `recheck` says UNOWNED where no agent ever served; guides say what to do | pyagag `1c7937d`; agfront guides |
| E | the refresh request began `sage:worldtrend …`, which addresses the *sage*; it cannot sync, so one extra round | study guides v3 (#14400–#14402) and archsage's guide: address archsage itself | archsage guide; routine guides |
| G | Front's first plan request named nobody, so the trace had no owner; the queue could not be read, and Observer asked about a healthy 6-minute wait | the owner of such a post is the agent whose `#agents` roster serves the topic (`MirrorReader.roster_owner`) | pyagag `e9e6229` |
| G | Front asked why the stand-in wanted an unexplained 7-minute step | answered; a reasonable question, not a defect | — |

Every fix was deployed to every consumer on the pin it needed (final
`e9e6229`; the relay `fb61ea3`, one display string later) and all
services restarted with nothing in flight. Test totals on the final pins:
pyagag 1050, agfront 193, agautolab 331, agobserver 176, relay 363,
agforge 265, archsage 39, cagent 204. `nctl drift` converged=46.

## Interventions, counted

- **Human (stand-in) input**: the requests; in G, the planned reserved
  approval (#14808), G2's go-ahead (#14659) and its explanation (#14664).
  None after the requests in C, D, E, F and H.
- **Developer repairs during trials**: the fixes above. E growbox's
  completion depended on one of them (deployed at 14:32, before Observer's
  second request), so it is an assisted, diagnostic run, not an autonomous
  success.
- **False interventions by Observer**: 4 in total (D ×3 `undelivered`, G ×1
  `unacknowledged`), each fixed; 0 in F and H.
- **Correct interventions**: E's stray topic (asked twice, then reported:
  the stray post can never be served, and Front had meanwhile posted in the
  right topic) and G2's injected stop.
- **Duplicates**:
  - no duplicate execution of any task;
  - one duplicate report delivery, caused by the developer (step 4's first
    startup, B growbox, one extra desk serving);
  - no duplicate acceptance: every repeat of `agentchat accept` wrote
    nothing.
- **Other paid extra servings**: the "Nothing new here" desk serving after
  each delivered report (one per run, by design of `continue_deliveries`);
  Front's opfail retries of `--to`.

## Review occurrences

Recorded with `agobserver.review_status`. The ✔ on each review topic is
left to the Developer:

- `review-front-unheld` 1–2 fixed (pyagag `a30e09a`), 3 fixed (`agrun`
  adopt/continue, p5 step 4);
- `review-autolab-undelivered` 1–2 fixed (`03f8fc9`), 3–4 fixed (trial D,
  requester queue);
- `review-archsage-undelivered` 1 fixed (trial D);
- `review-autolab-resolved-live` all fixed (`a30e09a`);
- `review-autolab-unacknowledged` 1 fixed (the queue reading, p5 step 2);
- `review-unknown-owner-unacknowledged` 1 fixed (stray topic refusal,
  UNOWNED), 2 fixed (roster owner);
- `review-autolab-stopped` 8 reviewed (G2's injected trial stop).

Unchanged from earlier phases, not part of this one:
`review-autolab-stopped` 1–7, `review-autolab-uncertain`,
`review-front-unanswered`. Observer residue: o14251 retired (the stray
topic owes nothing).

## Cleanup

- Faults: autolab `silent-exit` was consumed by its one use; no
  `timing.json`; no other fault file.
- `agrunfinish` is removed (script, grants, guides, docs). The fixed
  refresh topic is gone from all three study guides. The completion door
  was left as it was, and why is in report3.
- Documentation: README_DEV has *Waiting and finishing studies* (the adopted
  semantics), the reworked routine close-out, the progress panel's records
  and limits, and autolab's acceptance paragraph. The host notes are in
  `pj-agdev/.local/devenv.md` (*failsafe p5 on this host*).
- The trial loggers are stopped. Their boards stay in the ignored
  `agdevworld/.local/pg/p5{c…h}/` as the record.
