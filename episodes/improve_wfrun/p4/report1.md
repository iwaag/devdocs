# improve_wfrun p4 step 1: warden's coordination contract and execution path

Date: 2026-11-03. Plan: [plan.md](plan.md). Host-specific facts are in ignored local records
(`pj-agdev/.local/improve-wfrun-p4/notes.md`).

## Starting point, confirmed

| Item | Value |
| --- | --- |
| WFE deployed | agdevworld `8e35240` on the coordinator and worker `gpupc` |
| Executor | agautolab `5b62114` on both |
| Open attempts | 0; 21 queued `warden/evidence-fixture` jobs waiting for profile `hand` (pre-existing, kept) |
| Single-execution assumptions | `exec.ts` claim: one open attempt per worker lane (capacity 1, `lanes` capacity in `workers.json`); `wfexec.py`: one `Running` per executor; `cli/remote.ts`: `claim --lane` refused, `state.json` rewritten without a lock; warden's guide: "The execution slot is single: the run you continued starts only after your run's attempt ends" |

## What changed

**Contract and guides (warden project, Gitea `developer/pj-warden`, commit `8619d68`).**

- `README.md`: warden's purpose is system stability support with two jobs, recovery and execution
  coordination; what each does, who detects and who decides, limits (no collision prediction, moves
  between attempts only, nothing while the executor is down).
- `agent/warden.md` (new): the facts every warden role works under — the system, the nodes, that a
  node executes several attempts at once, records and their actors, the own node to record, what is
  read afterwards.
- `agent/recover.md` (moved from `agent/guide.md`, which is removed): the recovery role; the single-slot
  sentence is replaced by the fact that the continued run is admitted by its standing decision and does
  not wait for the recovery attempt, and that interference from a shared node is reported and sent to
  coordination (`wfe coordinate reconsider`).
- `agent/coordinate.md` (new): the coordination role — what it sees (`wfe coordinate view|events`,
  deeper reads), what it can do (start, defer, assign, note, stop, instruct, cagent requests), and the
  facts: nothing starts by itself, unnecessary waiting is a bad outcome too, no category rule, decide
  from the actual work and the measured facts, deferrals name their reconsideration, constraints, the
  basis token, the own node and the short outcome.
- `devdocs/workflows/coordinate.yaml` (new): agent `coordination-agent`, profile `sonnet`, node
  `studio`, lane `coordination`, no Zulip intake, one `coordinate` node. `recover.yaml` names the new
  guides; `project.yaml` states both jobs.

**The coordination path (agdevworld `96404c2`, corrected by `38adb4b` and `1053a73`, `workflow_editor/`).**

- `docs/coordination.md` (new) is the contract: configuration, admission, events and turns, decisions,
  the view, where it shows, limits. `docs/workers.md`, `docs/runs.md`, `docs/observation.md` are
  updated where they stated the one-slot rule.
- `workers.json` gains `coordination: {worker, lane, workspace, workflow}`; lanes are names, not
  capacities (`server/workers.ts`; the legacy object form is read as names).
- Coordination runs are ordinary runs of the configured workflow, created by WFE itself
  (`ExecStore.ensureCoordination`, requester `WFE`, receipt `coordinate:turn-<n>`), claimed by the
  coordination lane of the configured worker without any decision, one at a time
  (`ExecStore.claim`). Their own start needs no decision about themselves; Zulip is not on this path.
- Events (`coordination_events`: `queued`, `attempt-ended`, `worker-available`, `worker-unavailable`,
  `reconsider`, `wake`) are durable; a **turn** (`coordination_batches`) is opened only when something
  awaits a decision (a job with no decision, a deferral due by time or by its named event, a
  reconsideration request) and none is open, from the claim, the end of an attempt and the executor's
  reconcile — a burst makes one turn. Events are taken by the coordination attempt
  (`wfe coordinate events --take`; events older than the attempt are taken at its end). A deferral's
  wake time becomes a `wake` event in the reconcile, once.
- One authoritative decision stream: one turn at a time, and the agent dispatches several runs in one
  turn (`wfe coordinate start` per job, or `apply --file`).
- Judgment unavailable: a turn whose attempt ends without a decision closes; the next opens for what
  still awaits; after three in a row automatic turns pause for ten minutes, visibly (`wfe coordinate
  view` STALLED, the observation's `coordination.stalled`, Observer tells the owners), `wfe coordinate
  wake` opens one at once; a turn whose run disappeared is closed as `lost`. Nothing falls back to a
  hidden policy and nothing queued is dropped. The coordination workflow is exempt from recovery
  (`agobserver` `[recovery] exempt`), so a failed turn never waits for a recovery run that would itself
  wait for a decision. An executor outage is still told to the owners and no turn runs without an
  executor.
- Store version 5 (`exec.sqlite`): `jobs.decision|decision_id|defer_until`, `attempts.decision`,
  `heartbeat.seen_state`, tables `decisions`, `coordination_events`, `coordination_batches`.

## Checks run

- `npm test` (workflow_editor) at `96404c2`: 740 of 740, in the release build. Later figures are in [report.md](report.md).
- `test/coordination.test.ts` (new, 4 tests at this step, 6 after the corrections below) and `test/lanes.test.ts` (rewritten, 6 tests) pass: a job
  awaits a decision and a turn is opened for it; the coordination lane claims the coordination run
  without a decision and holds one at a time; a turn closes with its decisions counted; three turns
  without a decision stall creation visibly, a forced wake overrides, a lost turn is closed.
- `wfe validate --workspace warden`: every workflow valid, `coordinate` included.

## Corrected after the first live trials

Three defects in this step's mechanism were found after deployment and fixed in `1053a73`; the tests above
did not catch them.

- A deferral's `--on` never fired: the check compared an event's id with a decision's id, two separate
  counters. No test or trial had used `--on`. Found by reading the code.
- A turn claimed before `ensureCoordination` linked it was closed as `lost` with 0 decisions, although it had
  applied 2 (live turn 14). That also counts towards the stall. A turn is now linked by its job's receipt.
- Turn creation was not atomic across processes. It is now reserved in one transaction.

## Not done / open

- The guides are written from the contract; whether the agent decides well is step 5's question.
- The 21 pre-existing `evidence-fixture` jobs will appear as awaiting a decision after deployment; the
  guide says to defer an unservable job with the reason.
