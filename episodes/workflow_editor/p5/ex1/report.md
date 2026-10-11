# Workflow editor p5/ex1 — final report

This follow-up closes the two reliability findings of the [p5 final report](../report.md) and the backend-display gap it lists. Plan: [plan.md](plan.md). Step reports: [1 baseline](report1.md), [2 deliveries](report2.md), [3 reconciliation and migration](report3.md), [4 SQLite](report4.md), [5 backend](report5.md), [6 verification](report6.md), [7 rollout](report7.md).

## Result

| Finding | Status | How it was shown |
| --- | --- | --- |
| A growing retry batch was posted again | **Fixed and deployed** | Reproduced at baseline (`#6:answered` posted twice). The regression test passes, along with the crash and acknowledgment variants and the overlapping-sender case |
| Intermittent `database is locked` at store open | **Fixed and deployed** | Reproduced at baseline: 294 failed opens out of 3,200 in 8-process bursts, all at the WAL pragma (codes 5 and 261). The same failure appeared once in the combined autolab suite. Afterwards: 0 of 3,200 opens failed, 20/20 contention-test runs passed, and 5/5 combined suites passed |
| Run header showed "backend unknown" | **Fixed and deployed** | The live `improve-game/run-002` header now shows `claude_code / anthropic/claude-sonnet-5 (latest attempt)` |

The installed system is agdevworld `78caec2` and agautolab `ca8e807` (pj-agdev `2d650a4`). Migration preserved every existing record. The notifier resumed without replaying sent events.

## Implemented

- **Frozen deliveries** (editor `server/notify.ts`, notify protocol 2). A message is one delivery. Its events, their order, its text and marker, the documents it called unpublished, and its destination are frozen when it is first claimed.
  - An event belongs to one delivery, and a run has at most one delivery in progress; the database enforces both.
  - Content is rendered outside any transaction and committed in a short transaction that revalidates the selection.
  - `begin` is recorded before every POST. `sent` and `failed` name the delivery, its claim and its events; stale or unrelated results are refused, and a repeated acknowledgment is harmless.
  - One sender at a time, fenced by the owning process's liveness rather than lease expiry.
- **Reconciliation** (autolab `wfnotify.py`). A delivery that may have been posted is searched for among the sender's own messages: in every place it was sent to, in its current topic, and in recorded relocations, page by page back to before its first send.
  - Found: it is acknowledged.
  - Absent everywhere: the exact payload is sent again, with that evidence recorded.
  - Search error, lost access, or a history too long to cover: it stays uncertain and is shown as a problem, with `wfe notify resolve` as the person's path.
  - `inflight.json` is retired into the store.
- **Migration.** Notify schema versions 1–2 and exec version 1, each applied once in a transaction. Pre-p5/ex1 uncertainty becomes a legacy reconciliation, resolved by the evidence in the posts' markers. Inconsistent legacy states are diagnosed. A newer schema is refused.
- **SQLite** (`server/sqlite.ts`). Busy timeout before any lock. WAL and migrations run as short steps retried only on lock contention, within a deadline. Failures close the connection and name the store, the operation and the SQLite code. There is no generic retry around agent execution, publication or POSTs.
- **Backend header.** Shows the attempt in progress, or else the latest attempt; "unknown" when that attempt recorded none; notes when attempts used several backends; keeps the configured backend separate.

## Verification summary

All failure-path acceptance came from isolated, deterministic tests: temporary registries, the fake Zulip transport, injected faults, real multi-process lock holders, and the released p5 code taken from Git history. There were no live test posts and no new RTS run. Final suites:

- editor `npm run check`: 135/135;
- autolab notifier and executor tests: 24/24;
- browser checks `exec`, `docsteps`, `backend`: all passed.

Rollback to the released p5 code on a migrated store, and forward again, was verified before rollout. Details: [report6](report6.md), [report7](report7.md).

## Remaining limitations

- **Exactly-once is not guaranteed.** A post deleted by hand, or Zulip losing history, defeats the search. A pid reused after a sender crash makes a dead sender look alive; that blocks sending, and `wfe notify sender --release` is the way out. Legacy `published` follow-ups carry no checkpoint number and are matched loosely.
- **Finite repetition.** The counts above show that the measured failure rate is gone. They do not prove the absence of every race.
- **Live normal delivery on the new path was not observed.** No coordination event occurred during the rollout window. It will be visible with the next run's first message.
- **Rollback is safe only when no delivery is uncertain.** The old code would reintroduce the original defect for one that is.

## Deferred, unchanged

These were out of scope and remain as they were:

- **Result-acceptance UX.** Human-check answers, execution completion and explicit result decisions still mean different things. `improve-game/run-002` has no recorded decision; nothing was inferred or changed.
- **Migrating `build-game`** (still v1, reported unsupported).
- **Older browser checks** with failures recorded in [p5 report3](../report3.md). They were not touched or rerun.
- **Game browser verification** of the RTS feature.
- **Archiving the rehearsal destination** `wf-p5smoke` (#28, #271, #272).

Deus ex machina note: none. This was system code, written by the Omni Agent at the plan's request.
