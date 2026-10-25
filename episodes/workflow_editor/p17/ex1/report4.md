# p17 ex1 step 4 — focused verification and required checks

Plan: [plan.md](plan.md), step 4. Tests: agdevworld `93d183b`, `ae7d050`.

## Coverage against the plan

| Plan item | Where | Result |
| --- | --- | --- |
| Exact incident shape: same-run reused page, idle execution, approval, verified stop, cleanup successor claimed once, completion, parent delivery, no `run.recover` | `reviewRelease` *incident shape* | pass: 1 job, 2 attempts, no recover, parent gets the `completed` event |
| Final review without successors stops and completes with no extra attempt; deferred parent review kept | `reviewFlow` (existing flow, now also asserts no job before or after reconcile); `reviewReleaseState` *stop that failed* (`unlocked: []`) | pass |
| Approval during an open attempt, before its end is recorded: one later attempt, never concurrent | `reviewRelease` *approval while the attempt is still open*; `reviewReleaseState` *approval during an attempt* | pass: nothing queued beside it, normal end, one job |
| Crash after completion before enqueue, repaired by reconcile; no duplicates | `reviewRelease` *lost continuation … restart* | pass |
| Duplicate callback, repeated reconcile, multiple rounds, unrelated ready work, holds, cancellation | `reviewRelease` *holds, cancellation and duplicates*; `reviewReleaseState` *requested changes … next round*, *approval during an attempt* (`extra`) | pass |
| Stop failure keeps the approval, retries, only then releases | `reviewHost` *own reused page: a failed stop …*; `reviewReleaseState` | pass |
| Retained page / another review's usage preserved and shown | `reviewHost` existing *kept on request*, *two rounds on one page*; `reviewReleaseState` text | pass |
| Different unfinished run's page not stopped, ownership visible | `reviewHost` *different unfinished run* | pass |
| Already-stopped page: cleanup succeeds | incident test's explicit cleanup (`stop all` → ok), `reviewHost` (`stop all` → `[]`) | pass |
| Assigned remote execution | `reviewRelease` *remote worker*: job `worker = pc`, local claim idle | pass |
| Parent/child identity | receipts bound to round uid (run creation hash); existing `reviewHost`/`reviewFlow` identity checks pass | pass |
| Requested-changes continuation, same-child follow-up, cancellation cleanup unchanged | `reviewFlow` deferred flow, `reviewHost` cancellation, `correction`, `humanReview` | pass |
| Browser/API and CLI routes | `reviewRelease` *browser route* (HTTP start/decide, retransmitted receipt, view `release`); *remote worker* uses `wfe review start/approve/show`; `reviewHost` CLI/HTTP parity | pass |
| Old histories replay unchanged | `reviewReleaseState` *entries written before p17 ex1* | pass |

## Required checks

- Affected files while implementing (review*, humanReview, holds, exec,
  reconcile, recover, observe, status, correction, delegateflow,
  assessment, present, sessions, continuation, workers): 117/117.
- WFE `npm run check` once at the end: **600/600 tests**, `tsc`, both
  vite builds. The UI source (`src/reviewView.ts`) changed, so the builds
  are relevant. agautolab (executor) and aghangout did not change; their
  checks were not run.

## Compatibility note for step 5

`readRun` validates stored state against a replay. A run whose
`review.cleanup` carries `release: true` stores `reviews.<rv>.release`.
An older WFE replays no such field and would refuse that record, so the
agpc worker pin must move with the coordinator. Older records replay
identically under the new release, since the new behavior is keyed on the
entry flag.
