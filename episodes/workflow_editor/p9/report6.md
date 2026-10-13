# p9 step 6 — Migrating existing runs without changing their meaning

Step 6 of [plan.md](plan.md). Committed in agdevworld `116cc58`. The contract
is in `workflow_editor/docs/storage.md` *Migration*. Nothing is deployed and
nothing live has been migrated: that happens in step 7, with the writers
stopped.

**The real inventory has not been taken yet.** It must come from the new
release with the writers stopped. Opening the live `exec.sqlite` with this
code upgrades its schema to version 2, and the release now running would
then refuse the store ("newer than this release understands").

## What changed

### `wfe runs migrate --dry-run`: the inventory

It writes nothing, and it lists:

- **Runs.** Each one's execution state, hold, open attempt, queued jobs,
  unpublished checkpoints and open presentations, with the action:
  - `migrate` (or "an interrupted migration: continues");
  - `migrated`;
  - `keep-terminal`: completed or cancelled, and read as before through the
    resolver;
  - `keep-ide`: a run performed by an IDE agent, which stays in the project
    workspace;
  - `blocked`: an open attempt, or a record that cannot be read.
- **Repositories.** Each one's head and branch.
- **Unattributed files.** Uncommitted files of the project workspace
  outside every run folder.

### `wfe runs migrate --by <name>`

**Preconditions:**

- It is refused while the executor reports (heartbeat younger than 45 s).
  Intake and claims must stay paused.
- It first takes the host-local recovery copies, in
  `<area>/.local/backups/p9-migrate-<time>/`:
  - `exec`, `notify`, `intake` and `present` (consistent, `VACUUM INTO`);
  - the area's settings files;
  - each workspace's refs, heads and uncommitted files;
  - the inventory.

**Per non-terminal run the execution host performs:**

| Step | What it does |
| --- | --- |
| `reserved` | Reserves a copy under the run's own id (`reserve` with `adopt`; receipt `migrate:<workspace>/<run>`). |
| `made` | Makes the copy from the committed project. |
| `placed` | Places the run's folder there as it is in the project workspace, including a record that was never published. |
| `recorded` | Appends `run.relocate`, a new reducer operation that records only the storage. |

`run.relocate` is refused while an attempt is open or when the run already
has storage, and it is not a coordination event.

**Unchanged:**

- nodes, questions, answers, holds, decisions, receipts and jobs;
- nothing is queued, resumed or announced;
- the commits the run published before stay on the shared branches.

**Checkpoints keep the mode they were made with.** exec.sqlite v2 adds
`checkpoints.mode`: `run-copy` for new checkpoints, NULL for old ones. An old
failed checkpoint retried after the move still publishes the project
workspace's branch, and is never reinterpreted as a run-branch checkpoint.

**Unattributed changes** are reported with their paths and left alone.

**Restartable.** The journal is `<area>/.local/migrations/p9.json`.

- Running the migration again continues each unfinished run from its last
  step.
- A copy reserved before a crash is finished, not duplicated. This works
  even without the journal: the migration's own receipt marks the copy as
  "continue".
- A finished run is reported as `migrated` and not touched again.

**The handover's held run.** jevlab `experiment/run-001` is plan-only and
held before agreement. The migration keeps its shared commits, its plan and
its hold, and detaches its future execution from the project workspace. It
does not cancel or revert anything to unblock the project; step 3 already
removed the reservation.

## Verification

`test/migrate.test.ts` (new, 3 tests). The fixture builds the pre-p9 state
the way the old executor did: records in the project tree, published on
`main`.

- run-001 is the incident: planned and asked, answered "保留", taken up,
  `run.stop` `by iwaag`, attempt ended "stopped".
- run-002 waits for an answer. Its last note was never published, and its
  old checkpoint row failed.
- run-003 is cancelled.
- run-004 is an IDE run by "Omni Agent".
- `scratch.txt` is an uncommitted file of unknown origin.

| Case | Evidence |
| --- | --- |
| Dry run | <ul><li>Actions are `migrate, migrate, keep-terminal, keep-ide`. run-001's reason says "held (stopped)", and run-002 shows its failed checkpoint.</li><li>`scratch.txt` is listed as unattributed, and no copy is made.</li></ul> |
| Migration | <ul><li>`migrated: [run-001, run-002]`; `kept: [run-003, run-004]`. `scratch.txt` is reported and untouched.</li><li>The recovery copies exist: `exec.sqlite`, `registry.json`, `zulip.json`, `refs-lab.txt`, `inventory.json`.</li><li>Both runs resolve to their copies, and storage names `wfe/lab/experiment/<run>`. Their last entry is `run.relocate`, with seq + 1.</li><li>Nodes, questions, hold, attempts, decisions and execution are deep-equal to before. run-002's unpublished note moved with it.</li><li>No job is queued and `claim` is idle. `main` is unchanged.</li><li>The notify outbox after a scan is the same set as before, so there are no duplicate notifications.</li><li>The terminal run resolves to the project tree, and the IDE run is still Omni's.</li></ul> |
| Old checkpoint | Mode NULL. The retry publishes "experiment/run-002: after attempt aold-000003" on `main`, and no run branch appears. |
| Twice | `migrated: []`, `already: [run-001, run-002]`, and the seqs are unchanged. |
| The held run afterwards | <ul><li>It is still held, its answer text is kept, and `plan.md` is readable.</li><li>A history view at an old commit still resolves, inside the run's copy.</li><li>A new request in the same project becomes `run-005` (numbering continues) and is claimed and executed while run-001 stays held.</li><li>The person's resume is claimed in run-001's copy with its plan.</li></ul> |
| Interrupted | A copy reserved before a crash (no journal) is inventoried as "an interrupted migration: continues". The migration finishes it: storage is recorded and the copy has its repository. |
| Refusal | With a fresh executor heartbeat: "the executor reported within the last 45 s; stop it". |

| Check | Result |
| --- | --- |
| Editor suite | **284/284** |
| `tsc` | clean |

## For step 7

Deployment order:

1. Stop the executor (`com.agdev.agautolab-wfexec`) and the editor service,
   so that claims and intake pause.
2. Activate the new release.
3. Take the real dry-run inventory.
4. Migrate with `--by`.
5. Start the services again.
