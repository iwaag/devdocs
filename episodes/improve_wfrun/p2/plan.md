# improve_wfrun p2: retire run copies without losing records or work

## Purpose and starting point

Separate a run's durable record from the lifetime of its working copies. Finished runs must remain
readable after their copies are retired, and their saved work must be recoverable when a correction
or another supported continuation needs it.

References: [original problems](../p1/problems.md), [p1 final report](../p1/report.md), and
[p2 handout](handout.md). p1 delivered published-source discovery and execution on WFE `145c852`,
including coordinator and remote-worker trials. Its reported remaining issue is increased copy size
when all pinned submodules are initialized. The original inventory counted about 480 coordinator
copies and 75 GB; refresh those measurements rather than treating them as current facts. Worker
copies were not inventoried in that investigation.

The handout also requests correction and a first execution of gimmickimg's published but invalid
`partial-animation`. Track that as a separate deliverable so its outcome is not confused with storage
retirement or with the earlier flower scene produced under freework.

## Scope

- Durable run records and file reads independent of an active Git checkout.
- Inventory, eligibility inspection, explicit retirement and restoration through supported tools.
- Coordinator and worker copies, including unpublished local data and delayed worker messages.
- Restartable migration and lifecycle operations, deployed trials and measured storage effects.
- The handout's workflow correction and first-run verification, plus focused system feedback when
  integration introduces an invalid workflow.

Run-branch deletion, Git object-pool/worktree redesign, automatic age-based deletion and broad cleanup
of caches or shared outputs are deferred. Keep existing run branches through p2. Do not reset or
synchronize editing workspaces as part of retirement. Existing legacy/IDE runs without managed copies
remain readable; report their eligibility separately rather than silently treating them as managed.

## Storage and lifecycle contract

### One durable authority

Keep the authoritative run record, frozen definition, requests, reports and run-owned evidence in
durable storage outside the disposable working tree. Preserve logical run identities, request
receipts, allocation history and links. A run's published Git record remains an explicitly labelled
snapshot, not a substitute for the latest record after retirement.

Define a single record/file resolver and a separate working-tree resolver. Reads, observation,
delegation assessment, review history and allocation must work without materializing a checkout.
Operations needing project code resolve or restore a working copy explicitly. Preserve the normal
agent experience of writing run documents under the documented run path; choose and document how
that path reaches the durable documents without introducing two writable authorities. Git publication
and worker uploads must continue to produce intentional snapshots of the same documents.

Move record locking to a stable location outside the tree before any deletion can occur. Record
relocation must exclude concurrent writers, including worker uploads, notifier/relay operations and
late review or correction events. Preserve event sequence and provenance; storage changes do not
create execution, acceptance or integration events.

### Retired is a storage state

Represent ready, retiring, retired and restoring storage explicitly, with recoverable failure details.
These states are independent of run completion/cancellation. Keep damaged or accidentally missing
copies distinguishable from intentional retirement. Record who requested the operation, its inventory,
preservation manifest, verification result and per-host progress.

An eligible initial candidate is completed or cancelled, with no active attempt, queued continuation,
pending checkpoint/integration or unresolved worker synchronization. Held, waiting and idle standing
conductor runs are not finished candidates. Inspect presentations, retained reviews, external processes
and paths that still depend on the tree. Retain a copy while those dependencies remain; do not silently
stop a presentation or cancel work to make it eligible. Historical delegation links alone need not
keep a checkout once their records and evidence can be read independently.

Recheck eligibility while excluding new claims and tree writers. A concurrent correction or resume
must either win before retirement or wait for a verified restoration; it must never start in a tree
being removed. Inspection and ordinary record reads do not restore or schedule work.

### Preserve before deleting

Inventory each repository, nested submodule, working-tree modification, staged change, untracked file
and ignored file on every host that holds a copy. Include worker-only artifacts and logs not yet
received centrally. Directory names or `.gitignore` membership do not prove that bytes are disposable.

For each item, record one of: retained in durable records; retained as verified published output;
preserved in an archive; reconstructible from verified retained inputs/revisions; or explicitly
discardable under a documented project/run policy. Unclassified data keeps that copy in place.
Do not turn a cache/environment classification into an approval gate: provide evidence and supported
operations, and leave ordinary disposition decisions to the responsible agent.

Retain the exact commits needed for every repository through a durable archive/bundle or equivalent
managed retention. A disposable access cache or the current remote branch tip is not a recovery
guarantee. Preserve noncommitted bytes and the index state, executable modes and symlinks needed for
restoration; do not follow symlinks outside the copy when collecting data. Host-local settings remain
in local/private storage, not Git or publicly served artifacts. Published shared outputs keep their
existing identity and retention rules and are never deleted by this operation.

Write archives to staging, verify their manifests and checksums, then commit the lifecycle transition
before releasing the source tree. Make each step retryable after a crash. Confirm that the destination
is outside the deletion tree and has capacity. Report unavailable archives/shares as unavailable,
never as empty. Moving bytes to another disk is distinguished from actual aggregate space savings.

### Restore the saved run

Restore into staging from the saved revisions and preserved files, not current main. Verify the
result, attach the latest durable record and atomically make the copy ready. Restoration does not
itself queue a job, integrate, replay notifications or change run identity. A supported correction or
continuation that needs a tree invokes this path before execution, retaining its own normal semantics.

Keep per-host copy state and synchronization receipts. An unreachable worker is unknown and its
copy cannot be declared deleted or fully inventoried. Report partial retirement accurately. Reject or
reconcile stale worker uploads using the existing attempt identity plus lifecycle versioning where
needed, so a delayed upload cannot overwrite a newer record or resurrect a retired tree. Session
reuse after restoration follows the existing harness contract; record replacement when reuse is
unavailable rather than claiming the old session was recovered.

## Implementation steps

### 1. Inventory and specify the migration boundary

Refresh coordinator and worker inventories with logical run identity, status, Git and non-Git sizes,
dependencies, publication/sync state and expected reclaimable bytes. Investigate large copies first.
Inventory tools must be read-only and bounded, with inaccessible hosts and incomplete scans reported.

Trace `runspace.ts`, record locks, access readers, execution claims/checkpoints, worker forwarding,
presentations/reviews, corrections, allocation and backup/rebuild paths. Inspect the actual artifacts
behind the p1 report's claims relevant to storage. Write the detailed storage contract, migration map
and recovery procedure before changing the resolver. Prefer existing journals/manifests over a new
independent index that can drift.

### 2. Separate records and migrate safely

Implement the durable authority and resolvers, then update all record readers/writers and publication
paths. Preserve named-file and raw-file links, including evidence outside the run folder: retain it
with a stable mapping or keep the copy until the dependency is resolved. Retired reads must not depend
on the project workspace containing an integrated snapshot.

Add dry-run migration and a restartable journal. Take consistent backups of affected stores and
manifests. Quiesce the relevant writers for cutover, with a documented lock order. Preserve existing
history and source provenance, including p1 delegation `target.source`. Keep migration and deletion
separate: first demonstrate correct reads/writes after relocation, then permit copy retirement.

### 3. Implement inventory, retirement and restoration

Expose candidate inspection, per-run retirement, restoration and operation status through CLI/API,
with dry-run and structured results. Integrate storage status into the run view and offer the supported
actions there. Simple mode states whether work is retained and what action is available; Detail shows
per-host progress, dependencies, manifests, measured sizes and recovery failures.

Implement the worker lifecycle alongside the coordinator path. Start with explicit selected runs;
bulk execution consumes an inspectable selection and revalidates each run. Do not add an unattended
retention scheduler in p2. Archive verification and a failed eligibility check report technical facts,
not a request for human permission.

### 4. Resolve the workflow-definition handout

Ask gimmickimg through its ordinary project workflow to repair `partial-animation`'s invalid result
confirmation, then inspect the actual integrated definition and `wfe delegate targets` readiness.
Commission one bounded run of `partial-animation`, using a small task consistent with its intent.
Check the actual child record, revision, output and completion evidence. Record who found, requested,
implemented and verified the correction separately. Do not count the earlier freework scene as this
test or implicitly commission a larger scene rebuild.

On the WFE side, report added/changed workflow validation failures to the producing run at integration
and in its result evidence. Validate against the intended integrated revision and published peers,
not the stale editing workspace. Prefer actionable diagnostics and accurate readiness over a new
approval gate. Include a focused regression test for the `confirms-type` case and unavailable peer
information; distinguish invalid content from incomplete validation. Keep this change separate from
the storage migration so failures can be diagnosed independently.

### 5. Verify, deploy and retire a measured live batch

Run relevant tests, the required WFE package checks and executor tests when worker behavior changes.
Run replay/read checks over all registered live workspaces before release. Include a registry-scale
timing check for discovery, intake and notification, reflecting the p1 performance regression.

Use the existing `wfrun-trial` project for coordinator and remote-worker lifecycle trials, attributed
to a check actor. Test retirement and restoration there before selecting a small batch of real
finished runs with complete inventories. Restore at least one retired live trial and verify its files;
exercise a supported correction without replacing its frozen definition with today's workflow.

Deploy the necessary coordinator, executor and worker revisions through the existing mechanisms.
Verify installed revisions, UI reads, worker synchronization and normal new-run execution. Record
before/after allocated disk usage on each storage location, archive cost and retained dependencies.
Do not extrapolate savings to uninspected copies. Update storage/access/worker/review contracts,
tool help, backup instructions and development entry points to match delivered behavior.

## Acceptance evidence

| Case | Required result |
| --- | --- |
| Durable records | Retired runs retain latest events, definitions, reports, delegation evidence, review history and working file links. |
| Authority | Concurrent record writes and worker forwarding cannot create two writable records or lose an event during migration. |
| Eligibility | Active, held, idle, queued, unsynchronized and tree-dependent runs are retained with concrete reasons. |
| Local data | Dirty/staged/untracked/ignored files and nested submodule commits survive verified restoration; unknown data is not discarded. |
| Worker data | Worker-only files are preserved; lost contact and partial retirement remain visible; stale messages do not corrupt restored state. |
| Interruption | Crashes during relocation, archive creation, deletion and restoration are recoverable and retries remain idempotent. |
| Corruption/unavailability | Missing or corrupt archives prevent destructive progress and produce actionable errors without false success. |
| Restoration | Saved code and local bytes return despite main advancing; the latest record remains authoritative and no job starts merely from restore. |
| Identity | Retired run numbers and request receipts remain reserved; replaying an old request cannot create a replacement run. |
| Shared outputs | Published outputs and adoption references still resolve and verify after retirement. |
| Continuation | Correction/restoration races are serialized, frozen definitions remain intact, and session replacement is recorded when necessary. |
| Handout | The corrected `partial-animation` is ready and an actual run of it produces verifiable output. |
| Capacity | Measured reclaimed bytes and preservation costs are reported separately for coordinator, workers and archive storage. |

## Deliverables and reporting

- Implemented and deployed record separation, lifecycle tools and worker support, with updated contracts.
- A refreshed inventory, migration/recovery evidence, lifecycle trials and a measured live retirement batch.
- Independently verified disposition of the `partial-animation` handout and integration diagnostics.
- `report.md` here with exact revisions, checks actually run, evidence locations, data migrations,
  recovery limitations, per-deliverable outcomes and remaining work. Machine addresses, archive paths,
  deployment logs and private inventories stay in ignored local records.

An older release that cannot read relocated records or lifecycle events is not a usable code-only
rollback. Document coordinated data recovery and preserve any newer events before rollback. Branch
cleanup and further deduplication remain follow-up work justified by the measured remaining costs.
