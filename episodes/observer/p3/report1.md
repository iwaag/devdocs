# observer p3 step 1 — observation and recovery contracts (2026-10-14)

## What was inspected

- WFE (agdevworld `c100f56`, the installed release): `shared/run.ts` (reducer, `situation`, holds, attempt fencing, receipts), `server/exec.ts` (queue, slot, attempts, checkpoints, integration, `reconcile`, `runView`, `executorState`), `server/activity.ts` (turns and last write from the attempt log), `server/runspace.ts` (copies, `runSpace`, index), `server/runs.ts` (`runRefs`, `readRun`, `listRuns`), `server/intake.ts` and `docs/intake.md`, `cli/run.ts` (who records), `cli/wfe.ts` (`wfe exec …`).
- Executor: agautolab `wfexec.py` (`fa26c16`): one slot, 2 s ticks, heartbeat every tick, `reconcile` every 60 s, process group, 4 h attempt budget, `finished` on exit; after an executor restart a live agent keeps the slot and its end is recorded as `unknown` by reconcile.
- Observer (`pj-agdev/agobserver`): `monitor.py` (2227 lines, conversation-based), `triage.py`, `health.py`, `review.py`, `hold.py`, `disposition.py`, `withdraw.py`, the watch path (`listener`, `intake`, `worker`, `observe`, `notify`, `anchor`, `destination`, `store`), and the agentroom relay's `watchdog.py`.
- Live state (2026-10-14 ~11:40 JST): nctl `status` ok (Nautobot, worker, dumps); `com.agdev.wfe-service`, `com.agdev.agautolab-wfexec` and `com.agdev.agobserver-zulip` loaded; executor heartbeat current and idle; the only non-terminal run is jevlab `experiment/run-001`, held `stopped` by the Developer's answer (seq 11). Observer's local state: 0 tracked requests, 7 old incident files (4 dismissed, 3 reported, none open), 16 watches all terminal (14 met, 2 cancelled).

## Ownership map

| Subject | Authority | Owner |
| --- | --- | --- |
| Request, plan, questions, answers, instructions, holds, completion | `run.json` in the run's own copy and the documents it references | WFE operations |
| Work products, checkpoints, integrated changes | the run copy and run branches, publication/integration journals | WFE / executor |
| Attempts, PIDs, queue, slot, heartbeat, activity | `exec.sqlite`, attempt logs (host-local) | executor through `wfe exec` |
| Incidents, assignments, verified outcomes | Observer's store (`agobserver/.local/workflow/observer.sqlite`), linked by run `uid` | Observer |
| Agent-to-agent request and reply | Zulip (`wf-agops.recover` request topics) | transport |

Observer detects and coordinates; the recovery agent investigates and decides; WFE validates and performs transitions (`run.recover`); the executor owns processes. Plan agreement and result acceptance stay separate from permission to investigate or continue.

## Contracts written

- **[`workflow_editor/docs/observation.md`](../../../../pj-agdev/agdevworld/workflow_editor/docs/observation.md)** (agdevworld): run identity (`uid` = `<project>/<workflow>/<run>#<creation>`, creation = digest of the `run.create` entry; same display ids across registrations are `ambiguous`, never merged), the versioned observation `ag.wfe-observation.v1` (observation time, per-run source state and read consistency, seq, situation, attempts, process/group/log/open-tool facts, queue position and work ahead, hold with `human`, inputs, waiting reason and responsible party, last agent report, recoveries, publication and integration problems, explicit `unavailable`), the evidence table that keeps heartbeat, liveness, CPU, harness activity, open tool calls and workflow progress apart, and `run.recover` (only `interrupted`/`unknown` holds; attempt and sequence guards; no live process; budget 2 per person-originated control; receipt per incident and attempt; actor = the recovery run's agent verified by its own open attempt; grants no replay, acceptance, plan change or answer).
- **[`agobserver/docs/workflow.md`](../../../../pj-agdev/agobserver/docs/workflow.md)** (pj-agdev): responsibilities, the classification table (respected holds and waits, `interrupted`, `lost-attempt`, `working`, `uncertain`, `unowned`, `queued`, `queue-stalled`, `executor-down`, `publication`/`integration`, `unobservable`, `terminal`), incident identity `(run uid, family, episode)` with the episode table, the record set (incident, evidence, judgment, assignment, event), the recovery path (journal before send, read-back before re-send, acknowledgement = recovery run created from the post, performance = `run.recover` naming the incident, restoration = later agent reports), bounds and outcomes, and the session/attempt distinction.

### Decisions worth stating

- **The recovery recipient is a workflow.** `agops/recover`, a manual-policy workflow in a small system project, executed by the same executor and slot. Observer requests it by posting a `request-<incident>` topic in its Zulip channel; WFE's intake gains *agent requesters scoped to named workflows* (bots are otherwise ignored), and the post becomes a run with input kind `request` (requester Observer). This uses the existing durable intake, queue and wakeup path and no legacy listener. The recovery run exits after deciding, so it never holds the slot while the continuation it queued waits behind it.
- **Executor and service failures have no automatic recipient in p3.** A recovery run needs the executor it would be repairing. Observer and a separately launched watchdog report them to the owners by direct message, with the affected runs; this limitation is recorded rather than hidden.
- **A live but silent attempt is never given a second writer.** Mechanical evidence decides most cases (an open tool call within its declared timeout is a wait); the local model is asked only for silence without an open call and without CPU progress, and its `waiting` only postpones the next look.
- **`run.recover` instead of `run.resume`.** `run.resume` stays the person's (or an agent naming a person's words). The recovery records the incident, the ended attempt and the recovering agent's own run and attempt, so "who continued this" is a record, not a convention.

## Fixtures

`agobserver/tests/fixtures/workflow/` — observation documents with the expected classification:

| File | Case | Expected |
| --- | --- | --- |
| `interrupted-lost-retry.json` | jevlab `study/run-001` attempt `amv6sl63n-accd7c`: exited with `survey` running after the lost background retry; held `interrupted` | `interrupted`; incident (`attempt`, `amv6sl63n-accd7c`); recovery request |
| `intentional-hold.json` | jevlab `experiment/run-001` held `stopped` by the person's answer while another project's run works in the slot | `respected-hold` and `working`; nothing opened |
| `normal-wait.json` | an open question; a paced foreground loop inside its declared Bash timeout; an answer queued behind that attempt | `awaiting-person`, `working`, `queued`; nothing opened |
| `unavailable-source.json` | a missing copy; a record that kept changing; executor heartbeat 612 s old | `unobservable` twice (no conclusion); one `executor-down` incident naming the affected run |

## Inventory: keep, adapt, remove

| Component | Decision | Why |
| --- | --- | --- |
| WFE `ExecStore.runView`, `executorState`, `activityOf`, `runSpace`, `runRefs`, `readRun` | **keep**, reuse in `wfe observe` | already the facts the monitor needs; one source of rules |
| WFE `activity.ts` | **adapt**: also the session id and the open tool call (with declared timeout) | the required distinction between a long foreground command and a hung one is not visible otherwise |
| WFE `ExecStore.resume` | **keep** for people; **add** `recover` beside it | recovery must not be recorded as a person's resume |
| WFE intake (`requesters`) | **adapt**: `agents` scoped to workflows, request input | the recovery request must reach a run through the durable intake |
| Observer `monitor.py` (front discovery, trace, `stall_candidates`, Front asks, conversation rescue rules, tracked.json, held/retired/disposition reading) | **remove** from the active path and the package | conversation-derived lifecycle and legacy serving IDs; no compatibility kept |
| Observer `triage.py` | **adapt** into a workflow judgment (`wftriage` role, same `run_role` mechanics, prompt/input kept beside the transcript) | bounded asynchronous local-model judgment is reusable; its prompt is conversation-shaped |
| Observer `health.py` (owner probes via `agag.health`) | **remove** | legacy serving probes; process facts now come from WFE |
| Observer `review.py`, `review_status.py` | **remove** (occurrence review in Zulip) | tied to monitor incidents and Front; a recovered workflow incident keeps its own record and evidence; a developer review may be rebuilt on it later |
| Observer `hold.py`, `disposition.py`, `withdraw.py`, `trial.py` | **remove** | operate on `#front` request records (holds and dispositions are WFE records now: `run.stop`, `run.cancel`) |
| Observer watch path (`intake`, `worker`, `observe`, `notify`, `anchor`, `destination`, `store`) | **keep as library, not started** | general condition observation and exactly-once delivery with read-back remain useful; the old Zulip watch entrance is not required |
| `notify.py` read-back pattern, `destination.py` three-valued resolution | **reuse the pattern** for recovery requests | durable delivery with read-back before re-send |
| agentroom `watchdog.py` (health record → states → DM owners) | **relocate/adapt** into `agobserver.watchdog`, its own launchd job, reading Observer's workflow health and the executor heartbeat | it lived in a relay tied to retiring agents; the evaluation logic is reusable |
| `monitor-health.json` (`agobserver.monitor-health.v1`) | **replace** with `workflow-health.json` (`agobserver.workflow-health.v1`): loop cycle, observation freshness, judgments, recovery backlog | the old fields describe requests and mirrors |
| Old incidents (7) and watches (16) | **archive** with a retirement note, not replayed | none open; no work claimed complete |

## Next

Step 2 implements `wfe observe` and the activity/process extensions in WFE.
