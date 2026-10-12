# Workflow editor p7/ex2 — step 2: one refresh lifecycle per run view

Commit: agdevworld `15b6e8f`.

## What changed

- **`src/runRefresh.ts` (new): `RefreshCoordinator`.** Every reason to re-read is now a request to one coordinator per mounted view: first load, SSE change, reconnection, the 3 s host cadence, and the person's own operations.
  - A `full` request subsumes a `host` request.
  - One batch runs at a time. Requests made during a batch merge into the next batch, which is a single pending kind, so the queue is bounded.
  - `request()` resolves only after a batch that *began after the request* has been applied.
  - `dispose()` aborts the read in flight, drops anything pending, releases waiters, and guards `apply`, so nothing is applied after it.
- **`src/runView.ts`.**
  - The old `serial` `load()` and the independent `async` `setInterval` are gone.
  - `readBatch` reads the run, exec and notify concurrently, each wrapped as an `Outcome`. It then reads presentations according to the run just read; if that reading is malformed, it uses the last usable record.
  - `applyBatch` publishes one coherent result:
    - the run: last-usable-record behaviour kept;
    - exec: a failed read keeps the last facts and shows its error;
    - notify and presentations: a failed read keeps the last value.
  - The host cadence requests `host` only when no batch is running or waiting. SSE `flush`, Refresh, reconnection and every mutation (`submit`, presentation start/stop, adjust, checkpoint retry) request `full`, and mutations `await` it.
- **File reads** carry their own identity (`fileRequest`). A response is applied only if it is the latest read and the view is not disposed. `dispose()` also invalidates the identity.
- **`src/api.ts`.** `run`, `runFile`, `runExec`, `runNotify` and `runPresentations` accept an `AbortSignal`.
- **`src/live.ts`** is unchanged: its interface was sufficient.

## Verification

New section 6 of `checks/runInteraction.ts`. The run's own `run`, `exec` and `notify` reads are delayed by 1.2 s each, longer than the 3 s cadence allows for three sequential reads. Six progress records are made 400 ms apart, and the check waits through several cadence ticks.

| Assertion | Before (`6058c0f`) | After |
| --- | --- | --- |
| Slow reads converge on the latest sequence | passes, 5554 ms | passes, 638 ms |
| Shown sequence never goes back | passes (55 → 60 → 61) | passes (55 → 60 → 61 → 61 → 61) |
| No read overlaps another of its kind | **fails**: exec 2, notify 2 in flight at once | passes: 1 each |
| Leaving mid-read (2.5 s delay, then navigate) starts no further reads and leaves no run DOM | passes | passes |

The old renderer was never shown rolling the sequence back in this setting: its `serial` load already ordered full reads. Its failures were overlapping host reads and slow convergence. The new code also cannot apply an older batch over a newer one, because batches are sequential and each applies everything it read at once.

Other checks:

- `checks/runs.ts --repeat 10`: all 33 checks pass, including the malformed record, outage/reconnection, the fast R1 → R2 switch, and the history view.
- Sections 1–5 of the interaction check still fail exactly as in step 1. This step does not change rendering.
