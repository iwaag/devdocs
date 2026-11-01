# improve_wfrun p1: consistent discovery and execution of published workflows

## Purpose

A workflow created and integrated by a run must become discoverable and executable without anyone
pulling the registered project workspace. The editor, project readers, request intake and delegation
must agree on which published workflows exist and which definition a new run uses.

This plan follows [problems.md](problems.md), especially problems 1–4, and the Developer's agreement
with the proposed direction. The motivating case is gimmickimg's `partial-animation`: its definition
reached Gitea, but the project screen and run creation still read an older workspace. Producing the
flower scene inside freework also did not demonstrate execution of the new workflow.

## Scope and intended behavior

- The default source for published workflow discovery, validation and ordinary execution is the
  project's committed default branch on Gitea. Resolve it to an immutable commit before creating a run.
- The definition and initial project tree of an ordinary run come from that same commit. Submodules,
  including devdocs stored as a submodule, follow the pins at that revision.
- The registered workspace remains the editing area. Its age, local commits or uncommitted changes
  do not prevent discovery or execution of published workflows.
- Editing and an explicitly selected trial of editing content remain available. The UI and records
  distinguish them from published execution and state the definition and code sources actually used.
- Existing runs retain their frozen definitions and execution history.
- Agents can inspect publication, request availability and the workflow/revision actually used by a
  child run. No new approval or agreement gate is introduced.

Copy retirement, record relocation, branch deletion and clone storage optimization are outside p1.
They belong to later phases of problem 5. This phase does not bulk-pull project workspaces or discard
their local changes.

## Source contract

Use the existing committed-content access machinery (`server/access.ts`) as the foundation rather
than creating an independent Git cache or a second workflow registry. Introduce a common workflow
source/resolution interface where needed. Every consumer must be able to identify the selected
project, source, root commit, pinned definition repository/commit, definition digest and fetch state.

The following consumers must use this contract for published operation:

- CLI project discovery and the editor's published workflow list/detail;
- definition validation, repository bindings and cross-project delegate target discovery;
- ordinary request/run creation and initial copy materialization;
- intake route discovery and notification channel provisioning;
- delegate target resolution and dispatch, including cycle checks.

A run creation freezes the resolved revision; a later fetch must not silently change its code or
definition. Requests retried with the same receipt retain the same run and selected source. At
delegation dispatch, record the selected revision and carry it through intake to child creation so
that a delayed delivery cannot silently execute a different definition. Existing channels may be
reconciled asynchronously, but readiness and pending provisioning must be observable and recoverable.

Cache freshness is a separate fact from revision identity. Reads may expose cached content with its
age and fetch failure. Ordinary execution must not silently describe that content as the latest:
require a successful source refresh for default-branch resolution, or an explicit selection of a
known revision. An unavailable source is not an empty workflow list. Invalidate or refresh relevant
discovery after integration so a newly published workflow is not hidden by the cache window.

## Implementation steps

### 1. Trace and reproduce the source mismatch

Trace `workspace.ts`, `access.ts`, `runs.ts`, `runspace.ts`, intake, notification and delegation
consumers, including editor writes and workspace-based validation. Record the paths that currently
read definitions from a workspace and the paths that independently choose a run's base commit.

Build an isolated fixture with an old workspace and a newer remote commit containing a new workflow.
Reproduce both the missing editor entry and the run-start failure; the latter was only inferred in
the original investigation. Include a dirty workspace and devdocs-as-submodule variant.

### 2. Unify published resolution and run creation

Implement the source contract and move the published consumers onto it. Materialize ordinary run
copies from the selected committed source, without depending on the workspace's branch, local
commits or initialized-submodule set to decide what the published project contains. Preserve the
existing host-local resource/context behavior unless a demonstrated dependency requires adjustment.

Choose integration targets consistently with the published source rather than accidentally inheriting
an unrelated editing branch. Preserve existing per-binding integration policies. Record portable
provenance in the run; host paths and credentials remain local.

Update intake, notifier and delegation together. A workflow integrated during a running service's
lifetime must become usable without a service restart or manual channel setup. Verify recovery and
idempotence when publication, provisioning or request delivery is interrupted.

### 3. Make published and editing states clear in the editor

Show published workflows by default, with an explicit way to inspect and edit workspace content.
Make ordinary execution and a trial using editing content unambiguous. Preserve drafts and selection
when switching views. For a trial, capture the exact definition bytes and identify the code baseline;
never imply that uncommitted project code was included when only the definition was captured.

Expose workspace relation to the published revision as same, behind, ahead, diverged or unknown,
with dirty state separately. `atCommitted: false` alone does not establish that a workspace is behind.
Simple mode shows the useful situation and next action; Detail exposes commits, counts, fetch age,
source selection and evidence.

Provide a workspace synchronization operation through both UI and CLI. It fast-forwards only when
the selected branch and affected submodules can be updated without losing local work. Otherwise it
returns the concrete reason and relevant differences. Recheck state before mutation; do not stash,
reset, merge or rebase implicitly. Define partial-failure reporting and recovery for multi-repository
updates. Synchronization remains optional for published execution.

### 4. Make creation and use of a workflow independently verifiable

Use the existing delegation mechanism for a newly published workflow. Expose enough information
for a conductor to discover it, inspect its request readiness and identify the child run's actual
workflow and revision. Include these facts in the delegated result evidence where they are missing.

Add only the guidance supported by this incident: creating a definition and running that definition
are distinct results. When the request requires both, the conductor can commission creation, verify
publication/readiness, then delegate to the new workflow. Freework output is not evidence that the
new workflow ran. Keep operational usage in tool help and avoid imposing this sequence on unrelated
requests or expanding the always-loaded guide with a procedural checklist.

### 5. Verify, release and document

Run the relevant automated tests and required package checks. Exercise the deployed UI and ordinary
request/delegation paths after release, following the existing WFE and worker deployment mechanisms.
Update `docs/storage.md`, `docs/access.md`, delegation/intake contracts, CLI help and the maintained
development entry points to describe the final behavior. Update executor guidance only where needed.

Inventory existing records before a schema/reducer change and run replay checks against them.
Backward compatibility is not a requirement, but new provenance and receipts must remain readable
through deployment and recovery. Record the release revisions, affected components and recovery
procedure; code rollback alone must not be claimed sufficient if new records require newer code.

## Verification and acceptance evidence

| Case | Required evidence |
| --- | --- |
| Stale workspace | A new remote workflow appears consistently in CLI and UI and starts without pulling the workspace. |
| Dirty/ahead/diverged workspace | Published execution uses the selected remote revision; editing files, refs and uncommitted changes remain unchanged. |
| Revision race | Advancing the remote between discovery, run creation and copy materialization cannot mix definition and code revisions. |
| Delayed delegation | Dispatch, delayed intake and child creation preserve the selected source; repeated delivery creates no duplicate child. |
| Submodules | Definition and code use the selected root's pins, including nested modules and a module absent from the editing checkout. |
| Live discovery | Integration makes a new workflow routable and its notification target available without restart; pending/failure states are visible. |
| Source failure | Stale cache, fetch failure, missing workflow and invalid definition are distinguishable; no silent fallback to workspace content. |
| Editing trial | Published execution and editing trials are visibly distinct; trial records identify exact definition bytes and code baseline. |
| Workspace sync | Safe fast-forward works; conflicting local state is reported without loss; a partial failure is diagnosable and recoverable. |
| Existing run | Resume continues with its frozen definition; source changes do not rewrite its history. |

The end-to-end trial reproduces the gimmickimg situation in a dedicated test project: freework creates
and integrates a new workflow while the registered workspace remains old, then a conductor discovers
it and delegates a small verifiable task to it. Check the actual child record, definition provenance,
result and integration evidence rather than accepting the conductor's summary alone. Attribute trial
actions to the test actor, not the Developer.

Repeat the essential discovery and execution checks on the deployed service, including a remote worker
when the source/materialization changes affect it. Do not use the real flower-generation request as
an implicit new commission. Any live limitations or untested paths remain explicit in the report.

## Deliverables

- The implemented and deployed source contract, editor behavior and optional workspace sync operation.
- Tests and trial evidence covering the source mismatch and the new-workflow delegation path.
- Updated contracts and tool help matching the delivered behavior.
- `report.md` in this directory, naming revisions, checks actually run, evidence locations, remaining
  limitations and follow-up work for copy retirement. Machine-specific logs and addresses stay in
  ignored local records.
