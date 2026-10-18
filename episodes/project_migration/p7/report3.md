# p7 Step 3 — worker lifecycle and local materialization

Source: agdevworld `58e5c29`, `7745db5` (WFE: `cli/remote.ts`, `server/workerApi.ts`); agautolab
`84ae825` (executor). Contract: `workflow_editor/docs/workers.md`. These are fixtures and code;
deployment onto the two real environments is Step 5, where real-environment verification is recorded.

## The worker

The retained executor (`agautolab.wfexec`) is the worker on both hosts.

- **Identity.** A `worker` setting names its node; every `wfe exec …` carries it. The existing worker
  keeps using the local CLI. On the PC the same executor's `wfe` is the coordinator's client:
  `WFE_WORKER_AREA` is set, and the launcher has no registry.
- **One central intake and notification.** Intake and notify stay off in the PC's settings; they run
  only on the coordinator's host, as now. No Zulip entrance is provisioned for the PC.

On the PC, `wfe exec`:
- `claim` is a claim with a persisted receipt, followed by preparation;
- `started`, `session`, `context` and `stops` are the coordinator's calls;
- `heartbeat` reports the offer and host facts, then ships attempt logs by offset;
- `finished` ships the log, uploads the run folder, reports the end, and publishes the worker checkpoint;
- `reconcile` judges this worker's own processes and retries owed checkpoints;
- `adopt` lists live processes the executor can take back.

## Materialization

At claim the coordinator publishes the run's branches (`dispatch to <worker>`) and answers with every
repository, run branch and published commit. The worker then:
1. clones the root with `--no-checkout`, and checks out the run branch;
2. initializes the submodules the copy has, parents first, nested ones included, each on its run
   branch;
3. fetches each run branch and its integration target, and verifies the published commit is
   reachable;
4. moves each branch only forward. A branch that went another way, or an unreachable commit, is
   `preparation on <worker> failed: …`: the attempt ends interrupted and the run is held with that
   reason. No other revision is used.

After that it excludes `.local/`, brings the run folder from the record copy, and stages copied input
sets from its **own** share. Each file is checked against the recorded sha256.

It copies only `<worker area>/overlays/<project>/` into the copy's `.local/`, never the existing
workspace's `.local/`.

Git credentials are the worker's own (`gitea.json` beside its token file). They are passed as an
HTTP header through `GIT_CONFIG_*`, as on the coordinator; no URL or config file holds them.

Godot imports and other tool output happen inside the worker's copy. A forwarded `wfe run publish`
uploads the named folder into the record copy, and the coordinator publishes it to the share with
the existing manifests. Published input/output folders on the share are never a tool's working
directory.

## Processes

The agent is launched by the worker that owns it, in its own process group, behind the launch gate.
The gate now runs the harness as its child and writes `exit <code>` / `signal <name>` to `<log>.exit`.
It ignores the group's SIGTERM so it outlives the harness, and SIGKILL ends both.

- A stop is carried out by the owning worker and is pending until that worker reports the end.
- A restarted executor adopts its live attempt, so a later stop or end is handled again.
- If the process is gone, `reconcile` records the real exit from the file, or `lost` → `unknown`
  (held). The local reconcile on the coordinator reads the exit file too.
- A launch that never registered is lost after two minutes. Nothing is relaunched or moved.
- A worker that cannot reach the coordinator claims nothing. Its running attempt continues, its end
  report is retried by the next ticks, and its agent's forwarded commands fail with the reason.
- Heartbeat silence is shown as lost contact, not as an ended process.

Same-host sessions: `session_stored` checks the worker's own harness store. A missing conversation
goes through the existing `--missing` replacement. A reassigned run starts a new session (docs).

## Tests

- agautolab: 43 passed, 1 skipped.
  - New: a restarted executor adopts its live attempt and carries out a later stop, and the end comes
    from the exit file.
  - Gate checked separately: normal exit 3, SIGTERM, and no `go`.
  - The 13 inherited `test_wfnotify`/`test_wfconversation` failures were stale history numbers
    (`attempt.context` shifts later events by one per attempt) and are corrected. No delivery behaviour
    changed.
  - `executor.node` added to two identity expectations.
  - The session-policy refusal now waits instead of dropping.
- WFE `test/workers.test.ts` (Step 2) covers duplicate claims and reports, the lost claim, worker
  restart (exit file and `lost`), and stop through the owner.
  - Disconnect/reconnect is covered as unavailability: claims fail and reports are retried.
  - Damaging timing cases use fixtures only; no production run was disrupted.

## Defects found and fixed here

- macOS refuses `killpg` (EPERM) on a group that holds only an unreaped process. The stop loop now
  logs and waits instead of failing the tick.
- A lost claim left a pending worker checkpoint (Step 2).

## Left for later steps

- Run view display (Step 4).
- PC deployment: supervisor, runtime versions, token, share mount, Godot, harnesses (Step 5).
- Real-environment trials (Step 6).
