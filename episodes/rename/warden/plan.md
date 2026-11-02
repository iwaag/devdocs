# Rename agops to warden

Status: planned; investigation only, no runtime changes made.
Investigated: 2026-11-02 (JST).

## Objective and scope

Rename the existing agops managed project to **warden**, in lowercase, before
extending its capabilities. Keep its current recovery function and its existing
workflows, runs, artifacts and relationships usable. This episode does not add
operations features or redesign Observer's recovery policy.

The rename includes the project identity, displayed name, managed repository,
workspace, current documentation and recovery routing. Changing only the display
name would leave most operational interfaces named agops and would not fulfill
the request. No old-name command/API compatibility or URL redirects are required.
Preserving evidence and access to existing work is a separate requirement from
backward compatibility.

This plan follows `devpolicy/styles.md`, `devpolicy/terms.md`,
`devdocs/README_DEV.md`, `pj-agdev/.local/devenv.md` and `localrule.md` (paths here
are relative to the common projects directory). Machine addresses, credentials,
service identities and migration inventories belong in ignored local notes.
Execution does not require an additional approval gate or a general rollback
framework.

## Findings

### Authoritative project and current function

agops is not a source directory or submodule of pj-agdev. It is a separate
Gitea-backed WFE managed project, locally obtained under the area's
`pj-agops/`. Its own repository is authoritative; the files under
`pj-agdev/agobserver/docs/agops/` are explicitly its initial seed content.

The inspected project checkout was clean at `44df265`. Its root `project.yaml`
has both `id: agops` and `name: agops`. `agent/guide.md` identifies the project
as agops; `devdocs/README.md` does likewise. There is no root README.md.
The `recover` workflow uses `recovery-agent`, profile `sonnet`, and Zulip
request intake. The execution agent's name describes its role and is not an
agops spelling: keep it for this rename.

Eleven workflow definitions exist:

- `recover`, `freework`, `check`, `session-check`;
- `cluster-change-check`, `cluster-report-check`, `delegate-check`;
- `context-check`, `correction-check`, `release-check`, `evidence-fixture`.

The fixture workflows are part of the current project. Renaming is not a reason
to delete them, their held runs or their intentionally unintegrated branches.

### The name crosses persistence boundaries

The local registry contains a project, a workspace and a root repository entry
for agops. Repository identity is a stable Gitea numeric ID; owner/name is also
stored. Renaming the existing repository should retain that ID and its Git
history, rather than creating a replacement repository.

The local run index contains **105 copy manifests and 105 durable run records**:
104 copies are `ready`, one is `retired`. These are storage states, not proof
that the runs have finished. Some run documents also exist in the project Git
tree. Since improve_wfrun p2, the authoritative live record lives below
`<copy>/record/devdocs/runs/<workflow>/<run>/`; the run tree links to it.
Scanning only tracked files misses this data.

Read-only inspection found agops text in these live SQLite stores:

| Store | Affected tables observed |
| --- | --- |
| exec | jobs, attempts, checkpoints, integrations |
| intake | messages, requests |
| notify | targets, runs, outbox, deliveries, delivery_events, io_attempts |
| delegate | delegations |
| conversations | messages, invitations, invitation_events, activities, activity_events, annotations, workflow_refs, workflow_snapshots |
| present | instances, usages |
| Observer | suspects, incidents, assignments, outbox, events |

The challenges store had no text matches. These are search findings, not a list
of fields to replace: many matches are historical prose or hashed snapshots.
State can change after this investigation; take a fresh inventory at execution.

Observer's actual local recovery configuration names `agops/recover`,
`wf-agops.recover` and workspace `agops`. The area's `zulip.json` authorizes
Observer for `agops/recover` in `intake.agents`. Both must change together.

### Relevant source and limitations

| Source, relative to pj-agdev | Why it matters |
| --- | --- |
| `agobserver/src/agobserver/wfrecover.py` | Default recovery workflow/channel; assignment acknowledgement uses run UID |
| `agobserver/src/agobserver/watchdog.py` | Default recovery workspace |
| `agobserver/docs/workflow.md`, `docs/agops/` | Current operating guide and clearly labelled seed material |
| `agobserver/tests/test_wfrecover.py`, `test_wfmonitor.py` | Routing, acknowledgement and recovery self-exclusion fixtures |
| `agdevworld/workflow_editor/server/registry.ts` | Project, repository and workspace registration; re-registering an existing repository does not refresh its name |
| `agdevworld/workflow_editor/server/workspace.ts` | Saving project.yaml writes that file; it does not migrate the registry or stores |
| `agdevworld/workflow_editor/server/runspace.ts`, `runrecord.ts` | Copy identity, Git branches, published source, durable records and links |
| `agdevworld/workflow_editor/shared/run.ts` | Run state and history/replay contract |
| `agdevworld/workflow_editor/server/exec.ts`, `intake.ts`, `notify.ts`, `delegate.ts` | Workspace keys, receipts, run bindings and durable delivery state |
| `agdevworld/workflow_editor/server/conversationSchema.ts` | Persisted project/workspace references and content-addressed workflow snapshots |
| `agdevworld/workflow_editor/server/runBackup.ts` | Retired-run backup identities, retention paths and hashes |
| `agdevworld/workflow_editor/cli/wfe.ts` | Registration/unregistration exist; no project/workspace rename command was found |

The inspected WFE source revision was `33b357d`. The latest deployment entry in
the supplied local guide names `2951d02`; establish the actual deployed and
worker revisions before implementing. Do not assume the checkout and service
are identical.

WFE derives run UIDs from project/workflow/run plus creation time, branch names
from `wfe/<project>/<workflow>/<run>`, and notification keys/channels from the
project name. A file rename or unregister/register sequence alone is therefore
insufficient. Unregistration also removes workspace entries and checks for
unfinished execution; it is not a migration primitive.

## Target naming

| Surface | Target |
| --- | --- |
| project.yaml id and name | `warden` |
| registry project and primary workspace ID/label | `warden` |
| existing managed root repository | `pj-warden`, same owner and numeric repository ID |
| workspace directory | `<area>/pj-warden` |
| recovery workflow target | `warden/recover` |
| Zulip workflow folder/channels | `wf-warden`, `wf-warden.<workflow>` |
| newly created run branches and IO channels | `wfe/warden/...`, `wf-warden.<workflow>.<run>.io` |
| recovery execution agent and workflow IDs | remain `recovery-agent` and the existing workflow IDs |

Existing immutable historical identifiers may retain agops where they record
what actually happened. Each such retention must be deliberate and its current
lookup must still work. Current routing and product display use warden.

## Work plan

### 1. Specify the migration against a fresh inventory

1. Record the repository/default branch, registry entries, all workspaces,
   worker copies and deployed revisions. Check that warden, pj-warden and the
   target Zulip names do not collide. Read the published project definition as
   well as the local checkout: new WFE runs follow the published default branch.
2. Inventory all 105 existing runs again, their actual execution states,
   delegations, open reviews, presentations, recovery assignments and queued
   deliveries. Identify writers on every execution host. Do not cancel work
   merely to make the rename easier.
3. Enumerate structured references in the stores above, copy manifests, run
   histories, Git remotes/branches, shared outputs, retention repositories,
   retired archives and backup manifests. Search current definitions and
   context in other registered projects for incoming agops targets. Local
   tracked-source searches alone cannot establish that those targets are absent.
4. Produce a field-by-field migration manifest: old value, new value, owner,
   dependent keys, verification, and whether the field is operational or
   historical. Include composite keys, JSON payloads, receipt uniqueness,
   UID acknowledgement and hash dependencies, not just workspace columns.
5. Resolve existing-run identity explicitly before applying changes. Prefer
   retaining creation identity, original branches and frozen definition/source
   snapshots while rebinding their operational workspace to warden. Inspect
   replay, allocation, integration, Observer matching and retention restoration
   to determine where that is supported. If it is not, implement a recorded
   identity transition and its reader/replay support. Do not silently rewrite
   only run.json's top-level project field or overwrite historical snapshots.
   Historical identity resolution is for existing records, not a second active
   agops entrance. New runs must allocate beyond existing run numbers.

Deliverable: an executable, narrowly scoped migration with a read-only preview,
explicit preconditions and a durable progress record. It must detect already
applied operations and partial completion. A universal rename UI is not required.

### 2. Prepare source, definitions and documentation

1. Update Observer defaults and current examples to warden; adjust its tests.
   Keep self-recovery exclusion and open-assignment matching correct for both
   migrated existing recovery runs and newly created ones.
2. Prepare changes to the managed project's project.yaml, agent guide and
   devdocs entry. Add a short root README explaining warden's current recovery
   role, its workflows and its relationship to Observer. Do not describe future
   functionality as already implemented.
3. Update current WFE intake documentation and applicable test fixture names.
   Historical examples in `server/exec.ts`, `shared/run.ts` and `docs/runs.md`
   describe named trials; preserve their provenance rather than globally
   replacing every agops string.
4. Keep `agobserver/docs/agops/` clearly historical, with a note pointing to the
   renamed authoritative project. It must not become a competing current guide.
   Update the current Observer section of `devdocs/README_DEV.md`; append the
   actual deployment/configuration facts to ignored `.local/devenv.md` when done.
5. Implement only the WFE migration/identity support established as necessary
   in step 1. Keep changes in their owning repositories and update pj-agdev's
   submodule pins where applicable.

### 3. Rehearse the data transition

Use an isolated copy of the relevant configuration, stores and representative
run data. Disable outgoing transport and job execution in the rehearsal.

- Cover a completed run, held run, open recovery assignment, delegation with
  another project, presentation/review, pending notification, and the retired
  run. Include references from outside agops to agops.
- Verify that run history replay agrees with stored state and all referenced
  attempts, records, copies and artifacts remain reachable.
- Verify that notification/intake receipts still deduplicate old requests;
  migration must not repost old outcomes or create a second recovery run.
- Verify stable Zulip numeric bindings and Observer acknowledgement matching.
- Exercise interrupted migration and rerun; it must converge without duplicate
  projects, runs, channels or deliveries.
- Verify retired-run restoration and branch ownership. Keep existing hashed
  backups unchanged; a new backup must describe the post-migration state.
  Document how historical backup identities are resolved during restoration.

Run the relevant Observer tests and WFE migration/replay tests. If WFE changes,
run its `npm run check` (type checking, tests and both builds). Record actual
results, not an assumed pass from a textual search.

### 4. Apply the coordinated rename

1. Establish a short maintenance window by pausing the actual writers:
   coordinator/dispatch, intake/notification processing, affected workers,
   Observer and its watchdog, plus scheduled backup activity where relevant.
   Let active attempts settle or record a recoverable pause; never move a
   working directory underneath a live process. Prevent UI writes during the
   multi-store transition.
2. Capture a consistent migration input for the affected live stores, registry,
   configuration and run manifests/records. This supports diagnosis and retry;
   it is not a promise of general rollback. Retired-run backup alone does not
   cover live trees, the workspace or Gitea.
3. Rename the existing Gitea repository, publish the prepared definition on
   its default branch, move the workspace and update remotes and stored paths
   (including worker/copy references). Preserve repository ID, commits and
   unintegrated work. Update the registry's project/workspace entries and the
   repository's stored name together using the migration mechanism.
4. Apply the rehearsed structured data transition, retaining the agreed
   historical values and updating every dependent operational reference.
5. Rename the existing Zulip folder and applicable workflow/IO channels while
   retaining their numeric IDs and message history. Update notification target
   keys/names/bindings consistently; provisioning new empty channels would
   strand the existing request and result threads.
6. Switch Observer's workflow/channel/workspace settings and the area's
   `intake.agents` workflow authorization to warden. Apply discovered incoming
   current definition/context references in other projects.
7. Activate any required WFE and Observer changes on the coordinator and
   workers, perform read-only consistency checks, then resume processing.
   Keep an exact completed/pending operation list if any external operation
   fails; do not resume with partially changed routing.

### 5. Verify the deployed result and report

- The dashboard, project view, workflow discovery and new requests show
  lowercase warden. There is one active project/workspace and the repository
  retains its identity and history.
- All pre-existing runs are accounted for. History/replay checks pass;
  attempts, copies, delegation links, presentations and artifacts remain
  accessible. A representative held run can continue under warden without
  reviving completed work. Restore the retired-run fixture in isolation.
- A new warden freework/check request loads the published renamed definition,
  allocates a noncolliding run and delivers its result once.
- In a controlled disposable run, exercise Observer's incident →
  `wf-warden.recover` → `warden/recover` → acknowledgement → target continuation
  → restored verification path. Confirm a recovery run's own failure does not
  recursively create another recovery run. Existing assignments must neither
  disappear nor consume a new recovery budget because of the rename.
- Check pending callbacks/delegations and receipts after restart. Confirm no
  duplicate recovery requests, notifications or newly provisioned old channels.
- Create and verify a post-migration retired-run backup when affected. Audit
  remaining agops matches by purpose: historical evidence is expected; current
  defaults, active routing and accidental stale paths are not.
- Write `devdocs/episodes/rename/warden/report.md` with changes, commits,
  deployed revisions, migration counts, actual checks, retained historical
  names and remaining limitations. Put machine-specific evidence and a directly
  usable local browser link in ignored local notes so the person can inspect
  the deployed result with minimal steps.

## Investigation boundary

This plan is based on local source, the managed checkout, local configuration,
copy manifests/durable records and read-only SQLite inspection. No rename,
deployment, message send, service pause or remote mutation was performed.
Remote repository/channel state, worker files, current run activity, incoming
references across all managed projects and exact field migration semantics
remain execution-time checks. Their absence has not been assumed.
