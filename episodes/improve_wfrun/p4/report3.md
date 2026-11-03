# improve_wfrun p4 step 3: concurrent execution and consistent decision application

Date: 2026-11-03. Plan: [plan.md](plan.md). agdevworld `96404c2`, corrected by `38adb4b` and `1053a73`; agautolab `f1b41a7`, corrected by `d63a91f`, `246e6f5`, `96b5a32`.

## WFE (`server/exec.ts`)

- **Admission replaces capacity.** `claim` takes, for the claiming worker, the oldest queued job that
  is admitted (`jobs.decision = start`, by a decision on the job or the run's standing decision,
  applied at enqueue and at claim); a job awaiting a decision or deferred waits with that in its note.
  No lane has a capacity. The conversation lane keeps its label (recorded on the attempt) and is
  admitted like any other, so no accidental ceiling remains. The coordination lane is the one
  mechanism with a rule of its own (one coordination run at a time, no decision). An area without
  `coordination` in `workers.json` keeps one attempt per lane as before (fixtures, isolated areas);
  it is a documented setting, not a fallback.
- **One live attempt per run** stays the record's rule (`attempt.begin`); the coordinator's claim
  transaction makes one attempt per job; receipts and the retransmitted-claim answer are unchanged.
- **Decisions against identified facts.** A decision names its job and run; `--basis` is the view's
  token over the record's sequence, the job and its state, the node and the ordinary attempts open on
  the worker (coordination attempts and load samples excluded). A changed basis records `stale`,
  applies nothing and answers the changed facts. A decision while the run's attempt is open is
  refused (`attempt-open`); two decisions on one job leave the later in force and the earlier
  `superseded`; `apply --file` applies each item on its own and reports it, so a retry covers the
  failed members only.
- **Events, ends and turns** as in step 1; the end of an ordinary attempt is an `attempt-ended` event
  with its outcome and situation ("physical execution failures are reported with their actual cause").

## autolab (`agautolab/src/agautolab/wfexec.py`)

- An `Executor` holds `running: dict[attempt, Running]`; each tick tends every attempt (stop,
  timeout, session attach, end) and then claims once more; `current` remains the oldest attempt for
  the views and tests. Stops, exit files, logs, end reports and checkpoints are per attempt; a
  failure in one attempt's tick is logged and the others go on.
- A **keeper thread** (`serve`) sends the heartbeat, with the host's measured load, and runs the
  reconcile (`--holding` every lane's attempts) every minute, so a launch, an end or a slow
  publication never delays them; `--once` and the tests tick without a keeper. Notify and intake
  threads are unchanged.
- `adopt` takes back every live attempt of every lane; an ordinary lane this process serves as no
  loop of its own is adopted by the default loop. The `lanes` setting names the extra loops (the
  coordination lane on the coordinator's host).

## Remote worker (`cli/remote.ts`)

- `state.json` is changed under a lock (`mutateState`: read, change, write), never from a copy read
  earlier: the executor's keeper (heartbeat, log shipping), its claims and the ends of several
  attempts run as separate `wfe` processes at once. Log offsets are recorded after shipping, outside
  the lock. Forwarded commands, checkpoints, restart adoption and reconcile are per attempt as before.
- The coordination lane stays on the coordinator's host; a remote `claim --lane` is refused with
  that reason.

## Checks run

- `test/coordination.test.ts`: two ordinary attempts open on one worker beside the coordination
  attempt; racing — two decisions, two concurrent claims of one job (one attempt), a decision on an
  executing run refused, concurrent enqueues of two runs (one job each, each awaiting).
- agautolab `tests/test_wfexec.py`: new
  `test_two_attempts_on_one_worker_are_tended_apart_and_both_adopted_after_a_restart` — with
  coordination configured, the coordination lane launches the coordination run (a stub that admits
  everything through `wfe coordinate`), the default loop launches two attempts one tick after the
  other, the heartbeat carries the measured load, stopping one leaves the other, the stopped run's
  resume is admitted by its standing decision and launched beside the other, a restarted executor
  adopts both. The stub proves that decisions are carried; it says nothing about judgment.
- Existing executor tests (sessions, launch ownership, lanes without coordination, store lock) pass
  unchanged except for the recorded `placed`.
- `test/workers.test.ts` (remote worker protocol with a real HTTP coordinator and the worker's own
  `wfe`) passes with the locked state.

## Not done / open

- No remote-worker test of two simultaneous attempts with forwarding and publication at once; the
  live trial on `gpupc` (step 5/6) is the evidence for that.
- What I read for races: `cli/remote.ts` (materialization, upload, publication, state) and the
  coordinator's checkpoint and alignment paths. Per-run copies and per-attempt logs are disjoint, the one
  shared file on a worker was `state.json`, now locked, and publication locks are per run. I did not audit
  every shared cache, and harness-level state such as Claude Code's own store was not examined.
- A launch in one lane's tick still delays that lane's other attempts' stops and claims until it returns, for
  example during a slow remote materialization. The keeper keeps heartbeat and reconcile going meanwhile.

## Corrected after the first live trials

- An adopted process gone without its exit file was held for ever (`246e6f5`). The defect predates p4 and was
  found live on `gpupc`.
- The keeper's reconcile could call a `launching` attempt lost during a slow claim, because the keeper now runs
  beside the ticks. It is held back while a claim is prepared, within a bound (`96b5a32`). Found by reading.
- A decision that moved a run computed its basis over the target node, so it was always stale (`38adb4b`).
  Found live in trials 3 and 4; the agent retried without the basis.
- macOS memory was not measured under launchd because of `PATH` (`d63a91f`).
