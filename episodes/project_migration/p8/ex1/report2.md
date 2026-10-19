# project_migration p8 ex1 — Step 2 report: normal waits durable and resumable

Date: 2026-10-19. Source and fixture work only; nothing deployed yet (Step 5).

## What changed

| Where | Commit | Change |
| --- | --- | --- |
| Front guide (`pj-front/agent/guide.md`) | pj-front `15b2448` (local; pushed in Step 5) | *Waiting* states the run-027 fact: a node of Front's still running at the attempt's end holds the run interrupted; `talk` ends where its own part ends (`wfe run adjust --completing talk`), and what is done with the child's result is a node after the delegate node |
| Common run contract (agautolab `wfrun` guide) | agautolab `551b9ed` | the delegation section: the wait is the run's record, not the process; end the attempt with the other started nodes completed, failed or waiting |
| `wfe delegate request` (help and output) | agdevworld `c26fa5e` | the help says what ending the attempt with another node running does; the output names nodes still recorded as running and where result handling belongs |
| Child completion relay (`shared/delegation.ts` `childFacts`) | `c26fa5e` | a completed execution is told **once**, when the child's attempt that completed it has ended (at once if completed outside an attempt), under the receipt of the entry that completed it last. Documentation steps no longer produce one waking fact per step. A child reopened and completed again after that is a new result. Old parent records are unaffected: the last completing entry's receipt is already recorded there. `wfe observe` `unrelayed[].at` is now when the fact became tellable |
| Structured hold cause (`shared/run.ts`) | `c26fa5e` | `attempt.end` written by the executor carries `explain: cause`; the hold it derives keeps `cause` beside the sentence: `left-running`/`left-ready`/`nothing-recorded` (`nodes`), `missed-event` (`events`), `process` (`detail`), `lost`. Entries without `explain` replay exactly as before (tested), so stored records stay consistent |
| `nothing-recorded` | `c26fa5e` | with `explain`, an exit after which the run's work is still `not-started` (nothing started, asked or delegated) is an interruption — Step 1 finding 5; recovery side in Step 3 |
| Contract docs | `c26fa5e` | `docs/runs.md` (waits are the record's; the cause table), `docs/delegation.md` (wait duration and sessions, the graph around the wait, single completion), `docs/observation.md` |

No new operation was needed: the corrected shape is expressed by
`wfe run adjust --completing`, `wfe delegate request` and the node graph.
The active-work check in `attempt.end` is unchanged.

## Lifecycle tests (`test/delegatewait.test.ts`, all pass)

| Test | Proves |
| --- | --- |
| run-027 reproduced | a talk node left running beside a dispatched delegation → `interrupted`, cause `left-running: [talk]`; genuinely active work still interrupts |
| corrected shape | talk completed at the adjustment, `deliver` after the delegate node → `exited`, no hold, **no open slot**, the child runs in the released slot, one continuation, two parent attempts in all |
| documentation-step child | the child completes its execution several times in one attempt; the parent gets **one** `completed` fact, after the child's attempt ended, and one queued continuation, even after repeated relays and reconciles |
| structured causes | `left-running`, `process` (exit code 1), `lost` (process gone → `unknown`) |
| missed vs during | a fact recorded before the attempt and not taken up → `missed-event` with its identity; a fact recorded *during* the attempt → normal exit and a continuation queued at the end (the race between event persistence and process exit) |
| several children | two delegations: one child's question wakes the parent while the other child still works; the other child's failure reaches the parent too — neither suppressed by the other wait |
| hours-long wait | the parent waits through two coordinator restarts (every store closed and reopened) and repeated reconcile/relay passes: no hold, no job, no slot; the child's fact is relayed once, one continuation, never two attempts of one parent |
| person's stop | a stopped parent records the child's result and is continued by nothing — not the relay, reconcile or a restart |
| nothing recorded | an attempt that records nothing → `interrupted` / `nothing-recorded`; the same history without `explain` replays as a plain exit |

Already covered and still passing (`test/delegateflow.test.ts`): dispatch by
receipt, a post whose acknowledgment was lost, a bind that failed after the
child was created, answers applied twice, a lost continuation requeued once,
parent hold, cancellation leaving the child, failure and explicit
replacement.

**Session independence** (`test/sessions.test.ts`, unchanged, passing): under
`run` the next attempt resumes the retained session by id after the answer;
under `fresh` every attempt starts a new one. Both end the attempt at the
wait; nothing about the wait depends on the session.

Full WFE suite after the change: 430 tests, all passing after one expected
adjustment — `test/io.test.ts` ended an attempt that recorded nothing and
called it "a plain exit"; it now records a wait first.

## What this does not yet do

- Front's live behaviour changes only once the guide is pushed and deployed
  (Step 5), and is then a matter of the agent following it; the tool output
  is the safety net.
- Status words and reasons in views: Step 4.
- Recovery of malformed runs and missing delivery/queue rows: Step 3.
