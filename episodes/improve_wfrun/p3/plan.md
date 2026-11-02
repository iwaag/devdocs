# improve_wfrun p3: verified recovery and measured branch cleanup

## Purpose and starting point

Make the durable storage introduced in p2 recoverable after loss, then remove obsolete run branches
where recovery no longer depends on them. Measure the resulting space and operational benefits
before expanding cleanup or introducing another storage design.

References: [p2 final report](../p2/report.md), [p2 deployed evidence](../p2/report5.md),
[original problems](../p1/problems.md). WFE `aec2430` is the reported deployed baseline. p2 relocated
490 copies, retired seven real runs and two trial runs, and demonstrated a correction on a restored
run. Its measured batch freed approximately 4.15 GiB on the coordinator and 0.86 GiB on the worker.
Retention and archives consumed part of the apparent saving; per-copy sizes counted shared Git
objects more than once.

The p2 retention repositories and archives are not backed up. One retention repository serves
multiple retired runs, so its loss affects all of them. Recovery instructions were not rehearsed,
and restoration followed by worker re-dispatch was tested only in a fixture. These are the first
tasks of p3, before destructive branch cleanup.

## Scope and invariants

- Consistent, verifiable backup of the state required to read and restore retired runs.
- Recovery into an isolated area, with a documented recovery path for the deployed release.
- A live worker retirement/restoration/re-dispatch trial.
- Inventory, dry-run, explicit selection, deletion and recovery of obsolete WFE run branches.
- A bounded live cleanup batch and measurement of ref counts and allocated storage.

No automatic age-based deletion, broad retention-ref pruning, global aggressive GC or object-store
redesign is included. Existing editing branches and workspaces remain untouched. Shared outputs,
adopted deliveries and unique unpublished data are not deleted by branch cleanup. Further retirement
and data disposition are evaluated separately using p2's tools.

Retired run identities, request receipts, number allocation, latest records, frozen definitions and
evidence links survive cleanup. Branch absence is not run absence. Reads do not restore a checkout
or schedule execution. A storage operation is not an acceptance or permission gate.

## Step 1: inventory recovery dependencies and define the backup set

Refresh the deployed revisions, lifecycle inventory, per-host storage usage and run-branch inventory
on Gitea. Include nested/shared repositories, coordinator caches, retention repositories and worker
archives. Use bounded reads and state what could not be inspected. Inspect the relevant p2 evidence
and actual storage, rather than relying on its summary alone.

Map each retained run to everything recovery needs:

- durable record folders, copy manifests, archive contents/index/exclude files and worker archives;
- retention refs and all reachable objects, including staged-only blobs and unpublished commits;
- registry/configuration, allocation/receipt information, lifecycle journals and required control stores;
- published shared outputs whose matching bytes were deliberately omitted from archives;
- the software revisions, host configuration and private credentials needed to interpret or access them.

Reuse the existing backup/deployment mechanisms where suitable. Define the consistency boundary and
lock/quiescence strategy across records, manifests, Git refs, archives and database snapshots. A
filesystem copy taken while these are independently changing is not automatically a consistent backup.
Active runs outside the guaranteed scope must be named explicitly; do not claim full-service disaster
recovery from a retired-run backup alone.

Store the backup on a different failure domain from the source. Record identity, schema/version,
source revisions, capture times, file hashes, Git refs and restore instructions in a manifest. Keep
machine locations and private material in ignored records and appropriately private backup storage.
If shared outputs have their own backup, bind this backup to a verified compatible snapshot or retain
the required bytes here. A reference to the same live share is not a backup of its contents.

## Step 2: implement and prove backup/recovery

Provide supported creation, inspection, verification and recovery operations, with dry-run where
useful and structured failure results. Publish a backup only after its staged contents have been
verified; a partial or corrupt backup cannot replace the last usable one. Establish a repeatable
backup invocation and report the last verified snapshot, its age, coverage and failures. Select and
document an operational cadence; backup expiry policy must preserve a verified recovery generation.

Restore a selected backup into an isolated area that cannot send notifications, claim live jobs,
integrate or alter production refs. Remap host paths through configuration while retaining logical
identities. With the original retention/archive paths unavailable to the recovery process:

1. Read retired run records, reports and evidence links.
2. Restore a coordinator copy with local/staged data and nested repository commits.
3. Restore a worker-backed run and verify its worker archive and unpublished commit bundles.
4. Verify shared-output bytes and any dependent adoption references selected for the trial.
5. Check event replay, run allocation and receipt identity without dispatching a duplicate request.

Include missing/corrupt archive and retention-repository cases in fixtures, and demonstrate that a
failure identifies affected runs rather than reporting success. A rehearsal on disposable copies is
sufficient; do not damage production to simulate loss. Record recovery time, bytes restored, checksum
comparisons and any external dependencies still required.

Distinguish data recovery on the current release from rollback to pre-p2 code. Document a tested
current-release recovery procedure. If retaining the older-code rollback procedure as supported,
rehearse it in isolation, preserving newer events; otherwise label it unverified and do not present
it as the fallback for p3. A backup predating newer events cannot overwrite them without accounting
for that loss.

## Step 3: complete the live worker lifecycle evidence

Use `wfrun-trial` and a named check actor. Create a small worker-only file and an unpublished commit
on a side branch, retire the finished run, verify receipt of its archive and removal of the worker
copy, then restore and dispatch a supported correction to the same worker.

Verify the actual worker files, hashes, saved code revision, frozen definition and latest central
record after dispatch. Worker-only commits currently return as bundle files: prove they can be read
and recovered, and state that they are not automatically reinstated as refs. Inspect session reuse
or its recorded replacement. Confirm completion and normal synchronization afterwards. Check delayed
messages by fixture rather than injecting ambiguous writes into production.

Fix defects found in this path before using worker-backed runs in branch cleanup. Attribute trial
actions and result decisions to the check actor, never to the Developer.

## Step 4: implement recoverable run-branch cleanup

Define one common eligibility/resolution operation for CLI/API consumers. An initial candidate must
be a managed finished run, successfully retired, with complete host synchronization, a verified
recovery manifest and no active attempt, queued continuation, pending publication/integration or
dependent operation requiring the branch. Held/idle standing runs and unknown ownership are excluded.

Inventory exact repository/ref/object-id tuples and their owning run identities. Enumerate branches
created or used in shared and nested repositories, not just the root. Never infer eligibility solely
from age, a branch-name prefix, or whether a commit is reachable from main; cancelled and unintegrated
work can also be valuable. Verify retained commit/index reachability and backup coverage independently.

Expose a read-only preview with reasons, expected ref-count change and known storage limits. Execution
consumes an explicit selection and rechecks eligibility and each ref's expected object ID under the
appropriate lifecycle/publication coordination. Delete only the observed ref version; a concurrently
advanced branch is left intact and reported. Partial multi-repository deletion is journaled and
retryable. Retention refs, tags, main and non-WFE branches are outside the selection.

Preserve deleted ref mappings in a durable cleanup record and its backup. Recovery recreates a branch
at its recorded object ID from retained storage; an existing conflicting branch is reported rather
than overwritten. Restoration and subsequent correction must be able to recreate/publish the run
branch normally. Ensure allocation, receipts, history reads and index reconstruction do not rely on
the deleted branches still existing in Gitea or a stale access cache.

Branch deletion and Git object collection are separate operations. Keep protected retention refs
intact. Measure remote/cached/local storage separately, then run ordinary repository maintenance only
on a selected, backed-up repository when its concurrency and reachability requirements are satisfied.
Do not use immediate aggressive pruning on shared active repositories. Record whether reflogs,
other refs or hard links still retain objects; zero byte savings is a valid result.

## Step 5: verify, deploy and measure a small live batch

Add tests covering cleanup eligibility, ref races, partial failure/retry, ref recreation, restoration
after deletion, receipt replay and run-number preservation. Include shared/nested repositories and
a cancelled unintegrated run. Run the required WFE checks and affected worker tests, plus record replay
on all live workspaces. Repeat the registry-scale discovery/intake/notification timing checks so
backup or inspection work does not recreate p1's performance regression.

Deploy through the existing coordinator and worker mechanisms; verify installed revisions. First
exercise branch deletion and recovery on the trial project, then a small selection of fully inventoried
real retired runs. Verify record/evidence reads after deletion, restore a selected run, and show that
a correction can publish its branch again. Use fixtures for destructive failure injection.

Measure before/after ref counts and aggregate allocated bytes by storage location, with backup,
retention and archive costs stated separately. Capture timings and unrelated activity that limits
attribution. Do not sum per-copy `du` values as a reclaim estimate. Report branch deletion and any
subsequent GC results separately; do not extrapolate the batch to the whole registry.

Use the remaining inventory to recommend whether to retire more copies, discard demonstrated
reproducible data through the responsible agent, or investigate deduplication later. This evaluation
does not authorize blanket deletion of `.godot`, environments or generated files by name.

## Acceptance evidence

| Case | Required result |
| --- | --- |
| Backup coverage | Every selected run's restore dependencies are present in verified backup generations; omitted scopes are explicit. |
| Independent recovery | Records and saved work recover in isolation without reading the original retention/archive paths or emitting live effects. |
| Consistency and failure | Concurrent capture, interruption and corruption cannot yield a backup labelled usable with inconsistent contents. |
| Worker continuation | A live restored run receives worker-only bytes and recoverable commits, then completes with its frozen definition. |
| Cleanup eligibility | Active, ambiguous, incompletely synchronized and unprotected runs are retained with actionable reasons. |
| Ref race and retry | Advanced refs are not deleted; partial deletion resumes idempotently without affecting unrelated branches. |
| Recovery after deletion | Exact refs can be recreated; run restoration and subsequent publication work without changing identity or history. |
| Shared storage | Other runs' retained objects, adopted outputs and shared/nested repository refs remain valid. |
| Identity and reads | Retired records, links, receipts, numbering and replay still work after branch removal and cache refresh. |
| Capacity | Actual ref reduction, allocated byte changes, preservation costs and measurement limits are separately reported. |

## Deliverables and follow-up

- Deployed backup/recovery and branch-inspection/cleanup operations with maintained contracts and help.
- A verified independent recovery generation and rehearsal, plus the live worker continuation evidence.
- An explicitly selected live cleanup batch with recovery checks and measured results.
- `report.md` here naming exact revisions, checks actually run, evidence locations, per-deliverable
  outcomes, recovery procedure and limitations. Machine-specific paths and operational logs stay in
  ignored local records. Update storage/worker/access documentation and the development entry points.

The unexplained agent exits without node records remain a separate investigation. Record occurrences
encountered during p3 and distinguish agent behavior from storage defects; do not expand this phase
into a general execution-guide rewrite. Automatic retention scheduling, broad retention-ref pruning
and a new Git storage architecture require evidence from this phase before a later plan adopts them.
