# Robust workflow p3 — phase report

Plan: [plan.md](plan.md). Steps: [1](report1.md) gaps reproduced · [2](report2.md) waiting vs
terminal decisions · [3](report3.md) mission acceptance · [4](report4.md) receipts on every route ·
[5](report5.md) the final revision on the local backend · [6](report6.md) the Sonnet decision ·
[7](report7.md) rollout. Executed 2026-09-24 (JST) by the Omni Agent. Observer stayed on its local
model (`qwen3.8:27b-mxfp8`, agcode) throughout. Front, autolab and forge ran on Sonnet 5 through
claude_code, unchanged.

## What changed

| Area | Before | Now | Where |
|---|---|---|---|
| Waiting | "No recovery is needed" and "this work is finished" were one verdict, `legit`. A request waiting on the human left tracking when its ✔ was judged legitimate | A request is tracked until everything under it is `done`/`cancelled` **by record**. A dismissal stops nudges and ends nothing, and closes as *finished* or *cancelled* only on the work's own record | agobserver `monitor` |
| Judgments | A verdict was used whatever had changed while the local model thought (30–130 s). A kept `stall` drove the second request and the report | Each verdict carries its evidence's snapshot and is used only for that state. A superseded verdict is discarded and the current state judged, with the allowance kept. Backlog and churn show in the health record and the watchdog | agobserver `monitor`; agdevworld `watchdog` |
| Triage input | The stalled conversation and a one-line trace of the origin, with each post cut at 600 characters from the start | Plus the request's own conversation (its newest spoken posts), with long posts keeping both ends. One guide paragraph on "work waiting for a decision", one on the listener's ack. Each judgment keeps its exact `prompt.md`/`input.json` | agobserver `triage`, guide |
| Mission close | Nothing recorded a mission `done` on the normal path. Relaying the acceptance into the plan bought two runs that only said "noted" | **`agentchat accept <mission> --evidence <post>`** (`agag.acceptance`): tasks `accepted`, whose decision on which post, `done`, ✔, all selfnotes and idempotent. `accept.flag` is the same operation for a person in the plan, and `mission_done` and the completion door write the same record. autolab's close-out tells the requester it's their step. **A task closes only on the requester's agreement**, never on autolab's own start note | pyagag; agautolab; agdevworld `close`; agfront guide |
| Receipts | Only the mention route's trigger, or an answer Observer named with `[owed]`. The owner route read answers and left them owed, so the answer was served again for "nothing new" | Every thread a serving is handed is recorded (`note_input`). After a confirmed delivery the newest post naming the agent in each span gets its receipt, on every route; a restart before the receipt writes it without a rerun. The parent hop (a task started for this agent) is a thread. Callbacks under ✔ are recovered at startup, newer than a per-listener horizon | pyagag `serving`, `listen`, `topics`, `zulip`; agfront `evidence` |

## Final revision and its verification

pyagag `0ef3c61` in all seven consumers; agobserver `74c881f`; agautolab `13f242b`; agfront
`99ad106`; agdevworld `2891ab2`; archsage `e4488c2`; cagent `030cf70`. Running since 05:39Z,
verified from the venvs and the process start times.

Eleven live requests (A–K), including multi-task missions, three forge → autolab handoffs,
concurrency (up to three at once), the plan's eight cases, two held judgments, three listener
stops and a 24-minute real connection loss with queue expiry. **On the final code: no false
recovery, no unnecessary request, no lost unfinished request, no duplicate completed work, and no
Omni Agent rescue.** The earlier sets produced one needless request (trial C: input gap, fixed) and
five defects, all fixed and re-verified (report5). Fixture-only: the 13 h window boundary, an
interrupted acceptance, a strictly mid-run arrival, and queue expiry on the ✔-callback path.
143 runs, $13.12 for the step.

## Before and after

| Measure | Start of p3 | End of p3 |
|---|---|---|
| Tracked requests | 15, every one held by a mission that could never close | **5**, each with something genuinely unfinished (a planning twin ✔'d with no record, an open p3 task, the p1 setup conversation, trials B and B3 of p2 with no acceptance on record). Every p3 request left on the look after its record |
| Missions `started` with every task finished | 22 | **4**: m7601, m7732 and p2's B and B3 missions, all without acceptance evidence, left visibly pending. 18 were recorded on the requester-side post that accepted them (report3) |
| Redundant servings | 4 of 34 mention servings since p2's monitor (heuristic) | **0 of 43** in p3's trials, counted exactly from the journal |
| Answers without a receipt (p2's rule) | 1 (#9368, p2 B) | 1: the same historical post, not re-served. The ✔ horizon deliberately does not replay history |
| Unnecessary recovery requests | — | 1 on S1 (C); 0 on the final code |

## Remaining limits

**Identity and recovery**
- The listener queue and the recorded thread inputs are keyed by name. Receipts resolve by message
  id.
- The parent hop is one hop, found through the newest 1,000 root notes (302 in the realm today).
- A callback under ✔ older than a listener's horizon is not recovered by that listener. Observer's
  owed request remains that case's path.
- A verdict held only in the worker's memory is lost with the process, and the question is asked
  again.

**Completion**
- Recording a mission's acceptance is Front's act. On the final code it did so 3 of 3 times. Across
  the phase's 10 completed missions it used the tool unprompted 6 times, G fell back to a planning post,
  and B, D and H needed the human's follow-up. What is guaranteed is that an unrecorded mission stays
  visibly pending and tracked, and that no record exists without evidence. Four refusals caught
  Front's mistakes: a task still open (twice), the wrong argument, and evidence older than the
  mission.

**Work isolation (new, from the trials)**
- Concurrent missions share autolab's working tree, and a cancelled mission's uncommitted edits
  stay in it (E∥F).
- autolab's worker commits before the requester agrees (G, H, K). The close-out guard stops the
  task closing, not the commit.

**Judgment and discovery**
- Judgments rest on a 27B local model. It is consistent on the reviewed shapes after the input fix
  (report6), with an occasional 130 s timeout that reads `unclear`.
- Discovery is still `#front` only.
- nctl sees listener liveness, not the monitor's health.
- Front's replies sometimes carry a stray `</parameter>` line.

## Sonnet comparison: deferred (report6)

Every wrong local verdict in p3 had a non-model cause: a missing piece of evidence, a cut that
dropped the question, a definition overlap, or an unknown system fact. After those were fixed, the
same model judged every replayed and live case correctly. Reconsider when:
- a wrong verdict with the evidence in its prompt causes a needless request or report;
- more than one live verdict in ten is wrong;
- `unclear` or timeouts exceed a fifth of judgments;
- wider discovery brings harder judged kinds.

If it is run, it is a bounded 15–20-snapshot `triage` comparison with expected outcomes fixed
first. The harness has to be fixed first too (`--deadline-s` is agcode-only), or it is reported as a
backend comparison.

## Next action

1. **Work isolation in autolab**, the one correctness hazard the trials found and did not fix:
   concurrent missions editing one working tree, cancelled missions leaving edits, and commits
   before agreement. The fix is structural (a tree per mission or per task, and a commit only in the
   close-out), not a guide line.
2. **Wider discovery** (routines and argues first). Completion and receipts are now sound, so a
   request found there can end; the p2 report's reason for waiting is gone.
3. The bounded backend comparison only if its condition is met.

## Omni Agent work for in-system agents

- Did record eighteen p1–p3 mission acceptances for agent Front, with Front's credential, each on
  the requester-side post that gave it (report3). A one-time backfill of records that predate the
  operation; not a handoff candidate.
- Did the requester's part in every trial (requests, acceptances, reviews, the commit-scope and
  cancellation decisions, follow-ups asking Front to record a mission) for Front. That is the
  Developer's role, not a handoff candidate.
- Did the operator's fault injection: listener stops, Observer restarts, the forwarding-proxy outage
  and `triage-stall`. Operator trial work, not a handoff candidate.
- Did refresh agstudio's nctl observation once, after trial stops left a stale `service_missing`.
  The six-hourly job does the same; not a handoff candidate.
- Did fix the defects the trials found (report5), which is the Omni Agent's job in a development
  episode. The concurrent-tree hazard is left to autolab's next phase (above).
