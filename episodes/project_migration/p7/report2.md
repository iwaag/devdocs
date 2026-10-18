# p7 Step 2 — authoritative state and assignment

Source: agdevworld `58e5c29` (WFE). Contract: `workflow_editor/docs/workers.md`. Not deployed yet (Step 5).

## What changed

**Assignment is a record.** `run.create` carries `executor.node`. A run created by an execution
host gets it from, in order: the request's `--node`, the workflow's `execution.node`, the
project's `execution.node`, the area's `default` in `<area>/workers.json`. An unknown node is
refused before a copy is made. `run.assign` (`wfe run assign … --node --reason --by`) changes it
between attempts, with who and why (`assignments[]`); nothing changes it by itself.
`execution.node` is validated, written back and digested only when declared, so existing
definitions keep their approval digests.

**One coordinator, one store, one slot per worker.** The coordinator's `exec.sqlite` (schema v3)
records each attempt's and job's worker. Everything recorded before is migrated to the local
worker when the store is opened; the old `autolab` heartbeat row becomes the local worker's.
`claim` takes, for the claiming worker only:
- the oldest queued job of a run assigned to it, when that worker's slot is free;
- jobs of other workers are skipped, never taken or dropped;
- a job its worker cannot serve now (profile not offered, session policy its harness cannot
  keep, dispatch not published) **waits** with the reason in its note.

Previously such a job was dropped. The five existing fixtures that encoded the drop now assert
the wait. Allocation, receipts and copies are unchanged (still under the workspace lock on the
coordinator), so run numbers stay unique per workflow whichever node performs a run.

**Reports are bound to (worker, attempt).** `started`, `session`, `context`, `finished` and the
worker's checkpoint are refused from another worker (`attempt-foreign`). A repeated `started`
with the same pid, and a repeated `finished`, answer what was recorded. A claim may carry the
worker's receipt; the same receipt gets the same answer, so a lost answer cannot make a second
attempt. Stops, `adopt` and `reconcile` are per worker. The coordinator looks at process ids only
for the local worker.

**Snapshot authority.** For a remote worker:
- the coordinator's copy is the **record copy**. Its `run.json` is the only writable record.
- the worker's `run.json` is a snapshot, refreshed after each forwarded command and before its
  own commit. Upload refuses it.
- the worker commits code + run folder + the current snapshot and pushes the run branches.
- the coordinator then moves each repository of the record copy forward to the pushed commit,
  never backwards or forced, keeping `run.json` (`alignRecordCopy`).
- until that worker checkpoint is published, the coordinator publishes nothing of the run and
  integration waits (`workerCheckpointOwed`).
- record-only commits made later (a decision, the integration record) are the coordinator's and
  are fast-forwarded by the worker at its next preparation, so history stays linear. The final
  integration result stays visible because it is a coordinator checkpoint after integration.

**Retained semantics.** These still go through the coordinator unchanged:
- questions, answers, holds, resume, stop, cancel, adjustments and delegation;
- context resolution (at claim, on the coordinator);
- immutable attempt snapshots, detail reads and other-project reads (`wfe projects`/`context`
  are forwarded).

What changed for a remote attempt is what it is *told*: the brief names the worker's copy and
the host facts the worker offered. It never names the coordinator's paths or the person's
workspace. `sinceBase` gives a `git log … origin/<target>` for the worker's copy.

Single-slot wording was removed from the delegation brief, `wfe delegate` help, runs.md and
sessions.md. The waiting rule for delegations ("waiting frees the slot") holds per worker.

**Observation.** `wfe observe` adds:
- `workers[]`: last contact, slot with its pending stop, queue, waiting notes, profiles;
- `executor.node` per run;
- per-worker queue order;
- a remote attempt's process as the worker's to observe, with the worker's last contact. Its
  shipped log is read here.

`executor` stays the local worker's, as Observer and the watchdog read it.

## Migration prepared

- The `exec.sqlite` v3 migration runs on first open (backup before deployment, Step 5).
- `wfe runs assign-nodes [--dry-run] --by …` records `run.assign` (via `migration`) for every
  unfinished run with no node.
- Completed and cancelled records are not rewritten; they read as the default's.
- On the coordinator today this applies to the two held runs (jevlab `experiment/run-001`,
  playprey `onboard/run-002`); they are not resumed.

## Tests

- WFE 403/403 (401 before plus `test/workers.test.ts`).
- The new fixture runs a real coordinator service and a remote worker area, driven through the
  worker's own `wfe` over HTTP. It covers:
  - node precedence and an unknown node;
  - two slots at once, and the local worker leaving the remote worker's older jobs alone;
  - the copy materialized from the dispatch, and a brief with only the worker's paths;
  - fenced and repeated reports;
  - forwarded agent commands writing the coordinator's record, and an edited snapshot ignored;
  - the worker checkpoint and record-copy alignment with the authoritative `run.json`;
  - a job waiting for an incompatible worker and not taken by the other, then an explicit
    reassignment;
  - a wrong token, and concurrent claims producing one attempt;
  - claim retransmission;
  - a lost claim ending `unknown` (held, not rerun);
  - an exit file read after a restart.
- The fixture found one defect, fixed in the same commit: a lost claim left a worker checkpoint
  pending forever and blocked the next dispatch. Without a copy on the worker, the checkpoint is
  now recorded as `nothing`.
