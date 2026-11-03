# improve_wfrun p4 step 2: evidence and actionable choices for warden

Date: 2026-11-03. Plan: [plan.md](plan.md). agdevworld `96404c2`, corrected by `38adb4b` and `1053a73`, `workflow_editor/`.

## The read view

`wfe coordinate view [--json]` (`ExecStore.coordinationView`, schema `ag.wfe-coordination.v1`):

- every worker: contact state and age, **measured load** (`load`: cpus, load averages, memory total and
  free, GPUs with memory and utilization, and the time measured — reported by the executor with every
  heartbeat, `wfe exec heartbeat --facts-json`, from `os.getloadavg`, `vm_stat`/`/proc/meminfo`,
  `nvidia-smi`), offered profiles, host facts; its open attempts with the run's request (first 280
  characters), active nodes, the last recorded progress and the log's last write time, lane, the
  decision that admitted it, a pending stop;
- every queued ordinary job: admission (`awaiting`, `deferred`, `admitted`), its decision with the
  reconsideration, why it was queued, the job's note (a worker that cannot serve it), the request,
  what is ready, the run's situation, where its node came from and any placement constraint, and a
  **basis token**;
- recent decisions, pending events, the open turn, a stall, recent turns, and "needs" (why a turn is
  due).

Missing facts read as missing (`(not readable)`, `load: not measured`, `never reported`). Deeper
reads stay on demand: `wfe run show --history`, `wfe run state`, `wfe exec status`, the attempt's
log, `wfe observe`.

## The operations

`wfe coordinate …` (`cli/coordinate.ts`; `ExecStore.decide`, rows in `decisions`):

| Operation | Effect |
| --- | --- |
| `start <run> [--worker <id>] [--scope run\|job] --reason … [--observation …]… [--assumption …]… [--basis <token>]` | the job is admitted; scope `run` (default) stands for the run's later jobs on that worker; `--worker` moves first |
| `defer <run> --reason … [--until +10m\|<time>] [--on <events>] …` | the job waits; reconsidered at the time, at the named events, or at the next turn |
| `assign <run> --worker <id> --reason …` | `run.assign` between attempts, as warden (`via: coordination`) |
| `note <run> --text …` | delivered to the run's agent in its next brief (*Execution coordination*: the admitting decision, who shares the node, the notes) and readable with `wfe coordinate notes` |
| `reconsider --reason … [--run …]` | a `reconsider` event from a working attempt (its run named automatically), from recovery or from anyone; opens a turn |
| `events [--take] [--all]`, `decisions [<run>]`, `notes <run>`, `wake`, `apply --file` | reading, taking, forcing a turn, a batch applied one by one and reported per item |

The decider is the coordination run's agent inside its attempt (recorded as `agent {workspace,
workflow, run, attempt}`), else `--by`.

## Placement provenance and constraints

- `executor.placed` (`request` | `workflow` | `project` | `default`) is recorded at creation
  (`shared/run.ts`, `server/runs.ts`); the run view shows it ("named by the request (a placement
  constraint)" / "the workflow's choice") and every move (`assignments`: from → to, by, reason).
- A node the request named, or a node anyone outside coordination assigned last, is a constraint:
  warden's move is refused (`placement-constraint`) and recorded as `refused`; the view says so per
  job. Workflow, project and default choices are preferences warden may change; warden's own moves
  are not constraints.
- Relocation facts: a move applies at the next attempt only (an open attempt refuses it); a retained
  session is replaced on the new node by the existing record mechanism (the decision's note says so);
  worker-held files follow the worker's publication; a profile or tool the other node lacks is visible
  in the view (profiles, host facts) and not checked by the move.

## Decision output

A decision row keeps: who, when, action, run, job, worker, scope, reason, observations, assumptions,
reconsideration, basis, state (`applied`, `stale`, `refused`, `superseded`), detail, the deciding
run and attempt, the admitted attempt (filled at claim), the turn. No taxonomy, confidence score or
approval step.

## Checks run

- `test/coordination.test.ts`: start admits; a standing start admits a later resume at once; a defer
  waits, its wake event opens a turn, the status reads `Deferred · By warden, until …`; a stale basis
  applies nothing and answers the changed facts; the request's node is kept and the refusal recorded;
  warden's move is recorded with provenance and is no constraint; a person's assignment afterwards is;
  a note reaches the next brief; the brief says who shares the node and offers `reconsider`.
- `wfe coordinate --help` and `wfe exec --help` text updated; `tsc` clean.

## Not done / open

- No "request additional information" operation beyond the existing run operations (`wfe run ask`,
  `wfe run instruct`) and the notes; the guide names them.
