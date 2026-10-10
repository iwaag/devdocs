# workflow_editor p3/pre1 — step 1: contract and baseline

## Result

The run contract is written:
`pj-agdev/experiments/workflow_editor/docs/runs.md` (`ag.workflow-run.v1`).
It defines the run layout, input kinds, definition snapshot and closure,
identifiers and collisions, one history-replay reducer, node states and
readiness, waiting and questions, delegated runs, the execution summary,
acceptance, artifacts, the operation matrix, observation, and the future
integration boundary.

The baseline was recorded before any code change: the checks pass, and the
update measurement was made on isolated data.

## Existing run-related capabilities

| Where | What exists | Relation to runs |
| --- | --- | --- |
| Editor (`shared/`, `server/`, `cli/`, `src/`) | Definitions only. Graph semantics (edge = wait, join = all predecessors, delegate = waits for another workflow) are documented in `contract.md` but nothing executes them. No `run` concept in code. | Runs are new. The graph semantics become checked readiness rules. |
| Definition approvals | `approvals.intent/definition` with digest, declared approver and time | Recorded in the snapshot as states; not a gate (contract.md already says approval ≠ permission to execute). |
| Watcher / SSE | 1 s stat-based definition poll, 3 s Git poll, hello = full re-read, merged refresh needs | Extended with run files; no scheduler or database. |
| `devpolicy/agent_records.md` | One record per **agentic run** (request id, backend, outcome, cost/time, failure report) | A different unit. A workflow run spans many agent turns and human waits. `ag.workflow-run.v1` is separate. It keeps executor identity and backend apart, and leaves unknown backend facts `null`. |

## Contract decisions

The plan's proposals were adopted. These points were settled in the contract:

- **History is replayed, not trusted.** `run.json` stores the current state
  and the history. The state must equal `replay(history)` from the
  `shared/run.ts` reducer. Each entry also stores the `change` it made, and
  that change is checked too. A disagreement is an `inconsistent` record. It
  is visible and refused for further operations, and it is never repaired
  automatically. CLI, service and browser all use this one module.
- **Readiness** = pending and every predecessor `completed`. **Blocked** is
  derived from a pending node behind a failed, cancelled or blocked
  predecessor. Failure and cancellation never satisfy a dependency.
- **A node waits on one thing**: a question, a child run or an external
  reference, with a holder. The view derives the current holder. An answer
  that has not been taken up moves the next move to the executor.
- **Answer, take-up and completion are three records.** Take-up and withdraw
  resume a node waiting on that question. An answer alone does not.
- **Resuming a failed node** is allowed with a required reason. It is a
  recorded manual decision, not a retry mechanism. `cancelled` is final.
  Run cancellation cancels every pending, running or waiting node in the same
  entry. Failed nodes keep their failure.
- **Delegation** needs a running delegate node. The child is written first,
  then the parent links it. Parent completion reads the child and records its
  execution state and sequence as evidence. A missing, failed or cancelled
  child is refused.
- **Execution summary**: `cancelled`, `completed`, `not-started`,
  `in-progress` or `stopped`. It also carries counts, ready, blocked, active,
  waiting with holder, and failed, so mixed facts stay visible.
- **Acceptance** is `decide --decision accepted|rejected` with actor and
  evidence. `accepted` is only recorded on a completed execution.
- **Run ids** are `run-NNN` (next number) or `run-<name>`. The directory is
  created exclusively and existing names are refused. Question ids are `q1`,
  `q2`, …. Timestamps are ISO 8601 UTC with milliseconds and are display-only.
- **Input**: `braindump.md` records `author` and `recordedBy`. `request.md`
  records `requester`, `entrustedBy` (run, file or note reference) and an
  optional `onBehalfOf`. `--original` keeps a derived request's source input
  as `original-input.md`.
- **Snapshot**: the bundle is a byte copy of the root workflow and its
  transitive delegates. Each file gets a `sha256` (checked on read), a
  semantic `definitionDigest` and approval states. A child's bundle comes from
  the parent's bundle. Repository HEAD, branch and dirty count are recorded as
  context only.
- **Stale-view guard**: `--expect-seq`, which the browser always sends. It is
  not a lock. Writers still take turns.

### Operation matrix (planned)

The full matrix is in `docs/runs.md` under "Operations". The browser offers
the person's operations: answer a question, decide on the result, and cancel
the run. Execution records (start, wait, complete, fail, ask, delegate,
attach) are the executor's reports of its own work. The browser shows them
but does not make them. The person can record any of them with `wfe run …
--by <name>`, so this is a choice of medium, not a missing capability. A run
is created through the IDE, where the person gives the braindump.

## Baseline

The checks pass on an unmodified tree (`npm run check`, Node 26.6.0):

- type check passes;
- 51 tests pass, 0 fail;
- the production build passes.

Update behavior was measured on isolated data with `node checks/measure.ts
--repeat 20 --skip-large`. The script uses its own temporary repositories and
service on port 8195. The raw output is in
`pj-agdev/.local/workflow-editor-measure/p3pre1-baseline.{json,txt}`.

| Case | n | p50 | p90 | max | over 2 s |
| --- | --- | --- | --- | --- | --- |
| external edit → workflow view | 20 | 607 ms | 864 ms | 878 ms | 0 |
| external edit → project view | 10 | 630 ms | 1112 ms | 1112 ms | 0 |

- Idle with a viewer (15 s): 5 `git status` and 25 file reads; 49 ms CPU
  (workflow view).
- No viewer: no work.
- All 21 update probes pass: bursts, delete, rename, malformed and recovery,
  outage, Git state, workspace switch, registry.

These are the comparison points for run observation in step 3.

## Running services

These are local editor processes, not part of the Nautobot-managed inventory,
so they were identified from the process list. They are left untouched:

- `:8095` + Vite `:5175`: the p1 fixture service;
- `:8097`: the person's p2 area `pj-agdev/.local/workflow-editor-p2/`. It
  holds project `rts-vs-bot` with workflow `build-game`, which is the likely
  subject of the human p3 trial.

All pre1 checks use separate ports and temporary or `.local` rehearsal data.
