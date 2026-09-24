# Step 5 report — the final revision on the local backend

Plan: [plan.md](plan.md) step 5. 2026-09-24 (JST; the trials ran 03:00–05:58 UTC), by the Omni Agent
as the Developer's stand-in and as trial operator. Observer stayed on its local backend
throughout.

## Version sets

The trials found defects that needed code. The version set therefore changed twice, and each
change is followed by the repetition it affects.

| Set | Deployed | What | Trials on it |
|---|---|---|---|
| S1 | 02:54Z | pyagag `023224a` in all seven consumers; agobserver `ec32e59` (p3 steps 2–4); agautolab `9586b17`; agfront `167f0de` | A, B, C |
| S2 | 04:28Z (Observer), 05:05Z and 05:29Z (autolab) | agobserver `74c881f`: triage reads the request's own conversation and keeps its exact input (step 6's input fix, below). agautolab `0d99c04`, then `0421c95`: the last close-out addresses the requester; a task closes only on the requester's agreement | D, E, F, G, H |
| **Final** | 05:39Z | pyagag **`0ef3c61`** in all seven consumers (a clearer `agentchat accept` refusal); agfront **`99ad106`** (a guide line on recording a whole acceptance); agautolab **`13f242b`**; agobserver **`74c881f`** (pj-agdev `7080e76`); agdevworld `2891ab2`; archsage `e4488c2`; cagent `030cf70`. Every listener and the relay were restarted on it and checked for a harness child first. Installed commits checked from each venv's `direct_url.json` | I, J, K; the final follow-up of H |

Models, unchanged: Front, autolab and forge on Sonnet 5 through claude_code. Observer's `triage`
(and `observe`, `intake`) on `qwen3.8:27b-mxfp8` through agcode on this host's ollama. Monitor
settings unchanged from p2: a look every 120 s, a 12 h discovery window, `resolved_live` grace
60 s, `undelivered` 300 s, `silent` 2,700 s, retry 600 s, two requests, `REJUDGE_SECONDS` 3,600.
Watchdog every 30 s, with alerts by DM.

## The cases

All requests went through `#front` on the disposable project `pj-robustp1`.

| Case | Expected | Result | Where |
|---|---|---|---|
| Legitimate human wait after a ✔ | No nudge; tracked across inactivity and restart | **S1: failed.** The judge ruled C's ✔ a `stall` twice (117 s, then 49 s after Front answered Observer), and one needless request reached Front, which answered "a deliberate wait". Root cause: an input gap (step 6). **S2: passed.** D's ✔ was ruled `legit` (51 s, citing Front's question to the human). Nothing was asked, and D stayed tracked across an Observer restart and a 24.6 min outage, then finished (`finished`: "the wait ended with the work's own record, and nobody was asked"). The 13 h boundary is the W1 fixture (controlled clock) | C; D |
| Acceptance or cancellation during triage | Old verdict rejected or re-evaluated | **Three live instances.** C (S1, unplanned): a `legit` about `awaiting_requester … o10203` came back after the acceptance had landed. It was discarded ("the verdict … judged evidence that has changed … judging the current state instead") and judged again. E (S2, `triage-stall` held): the acceptance arrived while the judgment was held, and the returned `legit` was discarded at the next look (`o10364 → o10370`). F (S2, held): a cancellation. The held verdict was never applied, and the recorded `cancelled` closed the incident as "a decision, not a recovery". No request was made on a superseded verdict | C, E, F |
| Last task accepted, mission pending | Task closes; mission waits for its decision | **Passed** (A on S1, G on S2, J on final). Each time the task closed, the mission stayed `started` and tracked, and Front asked the human about the mission. A caveat: Front twice tried to record a mission on a *task* acceptance, and the operation refused both (task still open: A #10048, I #10702) | A, G, J |
| Mission acceptance repeated or interrupted | One evidenced completion; request leaves tracking | **Passed.** Every recorded mission has exactly one acceptance note. The human's repeat in A was declined by Front after reading the trace, and a live repeat of the operation wrote nothing (6 notes before and after). Each request left tracking on the look after its record (for example A at the 03:18 look; J and K at 05:57:54). An *interrupted* acceptance: fixture only | A; fixture `test_an_interrupted_acceptance…` |
| Answer handed to an owner-route serving | Receipt after delivery; no redundant mention serving | **Passed twice.** Front was stopped by the operator, the report landed, the human asked at home, and Front restarted. A: the receipt for #10012 came 16 s after restart, and the mention read "nothing owed now; skipped". B: the same for #10106, with the stop also interrupting an in-flight mention serving (below) | A, B |
| Answer mid-serving or outside its input | Stays owed; handled once | **Passed** (outside the input). #10012 (serving 243's thread stopped at #10007), #10074 (249 → 251) and #10106 (253 → 256) each stayed owed and were taken exactly once. *Strictly mid-run*: fixture only | A, B; fixture |
| Owed callback in a ✔ topic at restart | Taken once, no twin, no rerun | **Passed.** Front was down 10 min 51 s while C's closing report (#10217) landed in a ✔ topic. At restart: "recovery: #10217 in '✔ workrun-task1-m10161' names us and has no receipt". Observer's owed request was served first and wrote the receipt, and the mention was then skipped. The old index pass reads open topics only, so it would not have found it. Front's event queue did **not** expire (it resumed), so queue expiry on this path is fixture-only (S2) | C |
| Real mirror connection loss and resync | Staleness visible; nothing concluded; tracking and receipts survive | **Passed, with a real outage and a real queue expiry.** Observer's Zulip connection went through a local forwarding proxy, which was killed for 24 min 25 s (04:38:09–05:02:34). The mirror went `stale` with a real `stale_since` (04:38:09.9), and the watchdog reported `unable_to_observe` and DM'd the owners at +5 min 10 s. There were no posts, judgments or index changes while blind, while D moved on (accepted, committed, task closed). At recovery: "mirror queue … is gone; re-registering and resyncing", 139 calls, 2.6 s, and the watchdog `ok` 1 min 45 s later. The monitor then closed D's wait as `finished`. **Queue expiry was tested** (Observer's) | D |

## Normal cycles

11 requests: A (2 tasks), B (forge → autolab), C, D, E, F (cancelled), G (2 tasks),
H (forge → autolab), I, J (2 tasks), K (forge → autolab). Concurrency ran as A∥B, E∥F, G∥H and
I∥J∥K. On the final set: I, J and K, the multi-task mission, the forge handoff and three concurrent
requests. Each ended with its mission recorded `done` with evidence (F `cancelled`) and the request
out of tracking. p1–p3's leftovers are unchanged (five standing requests, report3).

## Measured

| Measure | Target | Final set | Whole step |
|---|---|---|---|
| False recovery (a rescue without the transition) | 0 | 0 | 0. C's incident was rescued on the receipt Front wrote, which is a real receipt |
| Unnecessary recovery requests | — | 0 | **1** (C, S1's input gap) |
| Lost unfinished requests | 0 | 0 | 0. The index never dropped a request with work open |
| Duplicate completed work | 0 | 0 | 0. B's interrupted mention serving was requeued and then skipped, and nothing ran twice |
| Redundant servings (the trigger was inside an earlier serving's input) | — | 0 | **0 of 43** delivered mention servings, counted exactly from the journal's inputs (p2 baseline 4 of 34 by heuristic) |
| Retained completed requests | 0 | 0 | 0. Each left on the next look after its record |
| Omni Agent rescue | 0 | 0 | 0 |
| Elapsed | — | acceptance → `done` 17 s (J, K); `done` → out of tracking ≤ 1 look | receipt after restart 13–16 s; stale → DM 5 min 10 s; outage end → `ok` 1 min 45 s |
| Cost | — | I, J, K and H's follow-ups inside D–K: 91 runs, $8.70 | **143 runs, $13.12** (Front 85, $7.91; autolab 40, $4.18; forge 9, $1.03; Observer 9 local judgments, $0) |

**Operator actions, counted apart:** four listener stops (Front ×3: A, B, and C for over 10 min;
the B stop also killed an in-flight serving, see below), two Observer restarts for cases 1 and 8, one
proxy outage, two `triage-stall` holds, and the deployments. **Ordinary human decisions:** every
acceptance, the icon reviews, one commit-scope answer (E), one cancellation (F), one "yes,
unresolve it" (D), and the follow-ups asking Front to record a mission it had not recorded (B, D, H).

## Defects found, and what was done

| # | Found in | Defect | Fix | Verified |
|---|---|---|---|---|
| 1 | C | The judge saw only the stalled conversation and a one-line trace of the origin, so a ✔ on work waiting for the human read as a stall | agobserver `455c2b3`/`74c881f` (step 6) | D (legit, no nudge); replays |
| 2 | B, D (S1/S2) | Front passed `agentchat accept` on to the human, even when their words had closed the mission, because the close-out line did not say whose step it is | agautolab `0d99c04`: the line addresses the requester and says when | E: Front recorded it itself |
| 3 | G | autolab's worker committed and wrote `report.md` in a task's first serving, and the close-out claimed "task 1 was accepted (#<its own start note>)" and started task 2 | agautolab `0421c95`: no close-out unless the processed input holds the requester's post (a start note never counts) | fixture; I and J closed only on the human's word |
| 4 | G | Front passed the accepting post as the mission, read the refusal as "cannot record", and relayed the acceptance by post, spending a planning run (the record then read "by Front") | pyagag `0ef3c61`: the refusal says how to name the mission | fixture; J and K recorded with the right arguments |
| 5 | H | Front acknowledged an explicit mission acceptance and recorded nothing | agfront `99ad106`: one guide line. Acknowledging records nothing; record the acceptance where the other agent's introduction says | H's follow-up, J, K |

**Front's mission recording, across trials.** Recorded without a follow-up: A, C, E, I, J, K. After the
human's follow-up: B, D, H. Via a workplan post: G. On the final set, three of three. Every
unrecorded mission stayed visibly `started` and tracked until the record existed. None was lost,
and none was recorded without evidence. The refusals worked as a net four times: a task still open
(A, I), the wrong argument (G), and evidence older than the mission (K).

## Found and not fixed (for the report)

- **Concurrent missions share one working tree.** E∥F both edited `wordcount.py`, and autolab had to
  ask which lines to commit. A cancelled mission's uncommitted edits stay in the tree: F's
  `--lower` remained until G's request asked autolab to discard them.
- **autolab's worker commits before the requester agrees** (G, H, K). The contract and the guide say
  otherwise. The new guard stops the task *closing*, not the commit.
- **Front's replies carry stray `</parameter>` lines at times** ("continuation block unreadable", a
  "stray closing tag" apology). Once (A #10015) the delivered reply was only that apology, and the
  status report the human asked for came one serving later.
- **The operator's stop in B hit an in-flight run** (a mention serving already acked). The journal
  marked it `interrupted` and requeued it, and the owner route's receipt then made it unnecessary.
  Nothing was lost or repeated, but the check before the stop was mistimed.
- **A verdict held in the worker's memory is lost with the process** (D: judged, restart 8 s later,
  judged again). That costs one extra local judgment and asks nobody.
- **Front agreed to a task itself on the human's behalf** after an asset acceptance (H, K). The
  delegation was reasonable, but in B, with the same words, Front asked the human. Recorded as
  inconsistency, not a breach.

## What this does not prove

- Eleven requests on one small project, by one requester.
- Queue expiry of *Front's* mirror on the ✔-callback path, a strictly mid-run arrival, an
  interrupted acceptance, and the discovery-window boundary are fixture-only.
- The mission-recording behaviour is three of three on the final set, but it is a model's behaviour
  under a guide line, not a guarantee. The safety comes from the record being required, not from
  Front getting it right.
