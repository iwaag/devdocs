# Workflow editor p9 — isolated runs and non-blocking holds

## Objective and agreed direction

Implement [braindump.md](braindump.md), following the proposal accepted by the
Developer. The incident and earlier behavior are described in
[the project_migration p2 handover](../../project_migration/p2/handover.md).

A person can stop a run, retain its plan and unfinished work, execute another
workflow in the same project, and later resume the original run. Work must
neither disappear nor leak into another run.

The isolation unit is the **run**, across all its attempts. Each run gets its
own branch and working directory, including writable submodule repositories.
Checkpoint publication saves to run branches; integration into shared branches
is a separate operation grounded in the existing result acceptance.

Keep the agent responsible for interpreting requests and deciding what work to
retain, revise or undo. Deterministic code owns storage, process lifecycle,
identity, concurrency and recoverable Git operations. Do not introduce a
taxonomy of plan-only, partially implemented and other interruption scenarios.

This plan covers implementation, migration, verification and deployment. Its
creation does not itself change any running service or managed run.

## Current implementation and scope

- `pj-agdev/agdevworld/workflow_editor/server/exec.ts`: `holder()` reserves a
  workspace for a begun run until completion or cancellation. A held run still
  prevents new requests. `claim()` provides the registered workspace as cwd.
- `server/publish.ts`: attempt checkpoints commit and push the current branch;
  a detached submodule falls back to its remote default branch. Saving work
  therefore also puts unfinished work into the shared project.
- `server/runs.ts`, CLI/API, notification and presentation code resolve records
  under the registered workspace. Moving only the executor's cwd is insufficient.
- `pj-agdev/agautolab/src/agautolab/wfexec.py` launches and stops attempt process
  groups. `agautolab/agent/guides/wfrun/guide.md` describes the agent's contract.
- `agautolab/src/agautolab/missionspace.py` provides relevant experience with
  isolated working copies and integration journals. Reuse the design lessons,
  without importing the older mission/task approval protocol into workflow runs.

The handover's stopped experiment is the migration acceptance case. Before
implementation and again before migration, re-read actual state through the
installed CLI and Git. Consult Nautobot or `pj-clusterintent/nctl` for service
state; host addresses, paths, backup locations and live identifiers belong in
ignored local notes, not this plan.

## Behavioral contract

| Operation | Effect |
| --- | --- |
| Hold (`stop`) | End the active attempt, retain the plan, records and work, and prevent automatic resumption. Other runs can proceed. |
| Resume | Queue work in the same run's working directory, with the person's instruction. |
| Cancel | End this run permanently and stop its presentations through the existing cancellation path; retain its records and saved changes. |
| Integrate | Apply the accepted result to the recorded shared target branches, with a durable outcome per repository. |

- Retain the host's existing single-attempt execution slot. Independent run
  storage removes workspace ownership across attempts, not the execution limit.
- Waiting for an answer also does not reserve the project. Answers to a held
  run remain recorded without implicitly resuming it.
- A stop request is not proof of process termination. Fence further reports,
  confirm that writers are gone, and save the checkpoint before declaring the
  stop finalized. An unfinished save remains visible and recoverable for that run.
- Preserve working directories on hold and, initially, cancellation. Ignored
  files, environments and generated data are not preserved by Git commits alone.
- A failed checkpoint push affects that run's publication state. It must not
  block unrelated runs once their separate working directories are available.
- Execution completion, result acceptance, checkpoint publication and shared
  integration remain distinct facts. Never turn plan agreement into result
  acceptance or label accepted-but-unintegrated work as integrated.

## 1. Establish the run storage contract

Inventory all paths that assume one project working tree. Define and document
one run-location resolver before changing execution:

- Stable identity combines the project, workflow and run; branch names must
  also avoid collisions in repositories shared by multiple projects.
- Keep a host-local index mapping identity to the dedicated working directory,
  repository identities, starting commits, run branches and integration targets.
  Record portable provenance with the run; never serialize machine-specific
  absolute paths into tracked records.
- Reserve run identifiers under a project-level lock, considering both existing
  records and isolated runs. Two requests must not allocate the same run number.
- Create the working copy idempotently from committed project state. Do not
  absorb a person's uncommitted edits. Record what was selected as the baseline.
- Isolate the root and writable submodules, including shared study/wedo and
  devdocs repositories. Verify nested submodule behavior, branch ownership,
  Git configuration scope and gitlink handling in fixtures before selecting
  the concrete worktree layout.
- Make local configuration and required ignored resources available explicitly.
  Do not share mutable output directories between runs or sweep credentials
  into commits. A missing local dependency must identify the affected run.
- Keep one authoritative writable `run.json` and one lock for it. A historical
  copy included in a shared commit is a snapshot, not another live record.
- Define index reconstruction from retained manifests and branch provenance.
  Missing or damaged working copies are reported; reading must not silently
  recreate a competing run or restart its work.

Version the changed storage/protocol contracts as needed. Use an explicit
migration instead of maintaining two indefinite execution modes. Preserve
existing evidence and fixed workflow definitions; do not regenerate history.

**Completion evidence:** fixture runs have separate roots, indexes and branches;
repeated allocation, interrupted creation and restart find the same identity.

## 2. Route execution and records through isolated storage

Use the resolver consistently in `server/exec.ts`, `server/runs.ts`,
`server/workspace.ts`, CLI/API, file access, watching, notifications and
presentations, and in the claim consumed by `wfexec.py`.

- Keep logical workspace identity separate from the attempt's physical cwd.
  Agent guides, file bindings and commands resolve against that run's copy.
- Lists and dashboards include held, cancelled and unintegrated runs even when
  none of their files exists on the shared branch. Deduplicate migrated snapshots.
- Answers, instructions, adjustments, decisions and process reports update the
  same record from every entry point. Preserve sequence guards, receipts and
  attempt fencing.
- File links, commit links, historical views, SSE updates, presentation sources
  and notify/intake lookups resolve the intended run and revision.
- Current project workflow editing remains on the project workspace. The run's
  frozen definition and its revisions remain owned by that run.

Change checkpoint publication to explicitly target each run branch. Remove the
default-branch fallback from isolated execution. Publish submodule commits
before root gitlinks and retain the existing retry journal concept. Record the
target with each step so a retry cannot publish to a subsequently checked-out
branch. Serialize checkpointing with other mutations of the same run.

**Completion evidence:** two runs modify the same tracked path independently;
both are visible through CLI/browser and notifications, with no shared-branch
change and no cross-run file contamination.

## 3. Make holds non-blocking and record their actual actors

Replace lifetime workspace ownership with run-level exclusion and the existing
host execution slot. Audit requests, claims, answer-driven resumes, explicit
resumes, cancellation, restart reconciliation and checkpoint retries together.

- A held or waiting run cannot prevent another request from being queued.
- Stopping cancels queued work for that run and terminates its active attempt;
  it cannot terminate or drop another run's work.
- Preserve dirty, untracked and ignored files through interruption and restart.
  Do not use shared-tree resets or stash switching as the isolation mechanism.
- Resuming uses the retained run baseline. Do not silently merge the latest
  shared branch or reset plan agreement. The agent can inspect intervening
  changes and seek new agreement when the plan or reviewed result changes.
- Attribute agent-initiated stop/resume/cancel to the agent, with a structured
  reference to the person's answer or instruction when that is the source.
  Browser actions retain their actual human actor. Fix CLI defaults that can
  misrepresent the recorder.
- Update Simple and Detail to state the effect: work is retained, the run can
  resume, and other requests can proceed. Keep draft/focus preservation intact.

**Completion evidence:** hold A, execute B in the same logical workspace, and
resume A with its original plan and files. Repeat with an answer wait, a running
process, an interrupted save, and an executor/service restart.

## 4. Integrate accepted results independently of saving

Add an integration operation and durable per-repository journal, separate from
checkpoint publication. Reuse existing result acceptance; do not require a
second approval of an unchanged result just to merge it.

- Bind acceptance to an exact deliverable revision and the repository commit
  set saved after all attempt writers stop. Acceptance during a final attempt
  must not accidentally approve later edits. Changes to administrative run
  records alone are distinguished from changes to reviewed deliverables.
- Existing acceptance can queue integration once the required final checkpoint
  is available. A run completed without acceptance remains saved but unintegrated.
- Record target branch, observed target commit, candidate commit and resulting
  commit for each repository. Serialize updates by actual repository/target,
  including repositories shared across projects.
- Fast-forward when possible. If targets moved, prepare the combined result in
  an isolated integration working copy, inspect and validate it. Route conflicts
  or necessary repairs to agent work; a terminal run can be followed by a linked
  repair run rather than silently reopening completed nodes.
- If the reviewed result changes, present that change for agreement. Never
  force-push or overwrite another run's work to satisfy an earlier acceptance.
- Publish child repositories before the parent records their gitlinks. A
  multi-repository integration is not atomic: show partial progress and resume
  from its journal after failure. Verify ancestry and remote state on retry.
- Keep authoritative run records accessible after integration. Do not turn a
  snapshot merged into the project into a second source of live state.

Provide inspect/dry-run, execute and retry paths through the normal CLI/API and
run view. The agent guide explains the facts and available tools; each tool's
help owns its operational details.

**Completion evidence:** unchanged-target, advanced-target, conflict, shared
submodule, remote rejection and crash-after-push cases converge without losing
work or publishing unaccepted changes. UI distinguishes saved, accepted,
partially integrated and integrated outcomes.

## 5. Define abandonment and reversal without erasing evidence

Cancellation retains work by default. A request to discard changes is separate
from stopping execution:

- Unintegrated work has not changed the shared branch, so no shared rollback
  is needed. Retain the run record and identify its saved branch/artifacts.
- For integrated work, expose the exact integration commits and affected
  repositories so an agent can prepare a reversal as new work. Use new commits,
  preserving later changes and the original evidence; do not rewrite history.
- Treat submodule target changes and parent gitlinks explicitly. Moving a
  gitlink back is not equivalent to reverting a shared submodule repository.
- External effects such as a deployed service or data written outside Git need
  their own inspection and reversal. Never claim that Git reversal undoes them.

Automatic branch deletion, working-directory garbage collection, exhaustive
rollback classifications and a dedicated restart-from-plan UI are outside this
phase. Same-run resumption is required; plan reuse can use the retained files
and an explicit reference in a new request.

## 6. Migrate existing runs without changing their meaning

Implement an explicit, restartable migration with a dry-run inventory. Classify
storage facts needed for migration, not the human intent of stopped work.

- Enumerate run records, open attempts, jobs, pending checkpoint retries,
  presentations, repository heads and uncommitted files before writing.
- With all affected writers stopped, preserve the records, databases, refs and
  local files needed for recovery. Store machine-specific recovery details in
  ignored local notes.
- Move nonterminal runs to their dedicated locations and rebind all consumers.
  Retain completed/cancelled historical records through the common resolver.
  Migration must not enqueue a held run, change its questions, or send duplicate
  notifications.
- Preserve the old checkpoint targets and receipts. Resolve or explicitly
  migrate pending retries; never reinterpret an old shared-branch checkpoint as
  a new run-branch checkpoint.
- Previously published work stays published. For the handover's plan-only run,
  leave its existing shared commits intact, preserve its plan and hold, and
  detach future execution from the shared workspace. No cancellation or revert
  is needed to unblock the project.
- If older runs have mixed or unattributable changes, report the concrete files
  and commits for agent resolution. Do not guess which changes to remove.

**Completion evidence:** migration twice has the same result, a crash can be
resumed, historical links still work, and the held incident run is available
without reserving its project.

## 7. Verify, deploy and report

Use isolated fixtures and test repositories for destructive and concurrency
scenarios. The main acceptance sequence is **A held with its plan and unfinished
work → B runs in the same project → A resumes**.

| Area | Required evidence |
| --- | --- |
| Isolation | Same-path edits in A/B, root and writable submodules, dirty/untracked/ignored files, unrelated human edits left intact |
| Lifecycle | Plan-only hold, mid-attempt stop, waiting answers, cancellation, late reports, duplicate requests, restart and failed publication |
| Discovery | Unique numbering, lists after restart, record repair/reconstruction, CLI/API/UI agreement, historical reads |
| Integration | Exact accepted revision, target advancement, conflicts, partial multi-repository progress, retry after push, shared submodule targets |
| UI and coordination | Simple hold/resume, accurate saved/accepted/integrated state, drafts/focus retained, run links, presentations and no duplicate notifications |
| Migration | Dry-run, interruption/retry, no silent execution, retained history and no loss of local files |

Run `npm run check` in `agdevworld/workflow_editor`, the affected executor tests
in agautolab, and the relevant existing execution, run interaction, presentation
and browser checks. Add tests for the new storage and failure boundaries rather
than mechanically testing every helper. Record any pre-existing failures with
evidence and assess their relevance.

Deployment follows these steps:

1. Finish fixture verification, documentation and agent-guide updates; commit
   and push the changed component repositories and superproject pointers.
2. Inspect current service and run state through nctl/Nautobot and the installed
   workflow CLI. Quiesce affected writers for migration and activation, keeping
   new intake/claims paused until the new storage is ready.
3. Build the committed release, take the required local recovery copies, migrate
   storage and activate matching editor/executor code without an old writer
   touching the new layout. Verify service and CLI release identities before
   resuming intake and claims.
4. Verify the migrated held run remains held and its plan is readable. Exercise
   the full A/B/resume flow with disposable runs, and verify the incident
   project's request admission without starting the person's stopped experiment.
5. Leave the normal browser entry ready for immediate human inspection. Record
   source commits, deployed versions, migration results, validation evidence and
   any remaining limitations in this episode's `report.md`; update developer
   documentation and ignored environment notes where appropriate. Commit/push
   the final documentation and pointer changes.

Do not reintroduce the removed release rollback command, previous-release
management or automatic fallback. Failed deployment or migration reports its
actual state and continues through an explicit repair path; recovery copies
are not permission to restore stale coordination databases and replay posts.

## Definition of done

- A stopped or waiting run no longer reserves the entire project.
- Its plan, records, tracked changes and retained local files remain available.
- Another run can work independently and the first can resume in its own copy.
- Checkpoints cannot accidentally publish unfinished work to shared branches.
- Accepted integration is identifiable, recoverable and separate from saving.
- Existing runs are migrated without cancelling, resuming or erasing them.
- The deployed CLI, UI, executor and notifications agree on run identity,
  location, lifecycle and publication state.
