# Workflow editor p20 — workflows without workspace management

## Goal

Make the workflow graph agree with what the next ordinary run will execute.
When an agent's change reaches the project's default branch, the user sees it
without synchronizing a workspace. A person can edit, save a draft, try it and
apply it from the GUI, without performing Git operations elsewhere.

The Developer should be able to verify this outcome: ask for a workflow change,
see the resulting graph, and start a run using that graph. Editing it directly
must make it clear when the changes become effective.

Basis: [explanation.md](explanation.md) and the subsequent discussion with the
Developer. The explanation records the pre-p20 behavior, including the placeholder
change reaching all 21 published projects while their editor graphs stayed old.
Keep it as a historical baseline rather than rewriting it as the new contract.

This document plans the work; creating it does not execute the implementation.

## Conventions and scope

Follow `localrule.md`, `devpolicy/styles.md`, `devpolicy/terms.md`,
`devdocs/README_DEV.md` and `pj-agdev/.local/devenv.md`. Files are English;
conversation with the Developer is Japanese. Backward compatibility is not
required. Preserve existing edits and run evidence when replacing old behavior.
Machine-specific deployment facts belong in ignored local notes.

Implement the complete path through deployment and verification when this plan
is executed. Do not add human approval milestones. Applying a definition is a
user operation that changes what future runs use, not a new permission gate.

Workspace checkouts and per-run working trees may remain internal mechanisms.
This phase removes their maintenance from ordinary workflow viewing and editing;
it does not redesign all project storage, repository editing or run retention.
Do not introduce a general branch-management interface as a prerequisite.

## User-facing contract

| Context | Definition shown | Meaning |
| --- | --- | --- |
| Project workflow, default | Latest successfully fetched published definition | What a new ordinary run uses, subject to a later published update |
| Editing | A durable draft initially based on a published revision | Changes not yet applied to ordinary runs |
| Run definition | The definition retained by that run | What that run started with; later run revisions remain separately identifiable |

Use one graph renderer for these contexts, with explicit source and editability.
Published and run views are read-only; Edit creates or opens an identified draft.
The run's evolving execution graph must remain distinguishable from its original
definition. A link from a run must never silently open today's editable file.

Ordinary labels describe effects: Applied, Editing, Changes not yet applied,
Latest version changed, and Definition used by this run. Git revisions, paths and
workspace diagnostics belong in details. Source identity and stale/error notices
must remain visible in simple mode.

Actions:

- **Edit** starts from the current published version, or explicitly resumes an
  existing draft and shows if its base is older.
- **Save draft** durably stores the editing state without changing ordinary runs.
- **Try draft** runs an exact saved revision of the draft and records its source.
- **Apply** publishes the saved draft and makes it available to subsequent runs.
- **Save and apply** combines saving and applying for a short edit.
- **Discard draft** removes that draft through an explicit action.

Applying does not mutate already-created runs. Trial execution retains the
existing contract of using published project code with the selected definition;
it is not a trial of arbitrary uncommitted project code. Make this limit clear
where a person chooses a trial.

## 1. Trace current paths and establish source identity

Inspect the deployed baseline, not only the explanation's release. Record source
revisions and deployment facts in ignored p20 notes. Trace project workflow
discovery, published caches, editor reads/saves, live updates, request intake,
trial creation, run-definition links and default-branch integration.

Start with `workflow_editor/src/projectView.ts`, `workflowView.ts`, `runView.ts`,
`api.ts`, the corresponding server handlers, published-source and workspace-sync
services, and the storage, access, contract and run documentation.

Define a shared source descriptor for published, draft and run reads. It must
identify the project, workflow, actual definition repository/path, revision and
content digest. Preserve the distinction between a project root revision and
the definition repository revision where definitions live in a submodule.

Resolve graph validation and context reads against the selected source too.
A published graph must not acquire stale metadata or missing-area errors merely
because the coordinator checkout is old. Identify intentional host-environment
checks separately from definition validity.

Document exact API/CLI contracts and error states alongside implementation.
Handle a project without a published source explicitly as not yet applied;
never label a workspace fallback as a published definition.

## 2. Make ordinary viewing follow published definitions

Replace the published YAML-only view with the graph. Project navigation and
ordinary workflow links open this view, including workflows absent from the
local checkout. Keep text inspection available as a secondary action.

Refresh published state after integrations and applies, and detect external
pushes while a view is open through bounded polling or existing notification
facilities. Re-read on reconnect. Reuse published caches and coalesce requests;
do not fetch every repository on every UI render. Define and verify a refresh
bound, initially within 10 seconds for an open connected view.

Show the last successful check time when freshness cannot be established. On
fetch failure, keep a previously fetched graph only with an explicit stale
notice; do not silently fall back to the workspace. Distinguish deleted,
renamed, invalid and unavailable definitions without offering a misleading Save.

Correct run-definition navigation to use retained run data. Test it after the
published workflow changes and after run retirement, using the existing storage
resolution rather than requiring a live working tree.

This step fixes viewing but is not completion of p20: continue through editing,
application and migration before presenting the work as complete.

## 3. Introduce durable drafts independent of the checkout

Store drafts in service-managed storage, separate from a person's checkout.
Record a draft id, author, project/workflow identity, base source, base content,
saved content, revision and timestamps. Reloading or restarting must not lose a
saved draft. Expose existing drafts rather than silently creating competing copies.
Different users/tabs must not overwrite each other: use revision-checked saves
and identify the draft being resumed.

Keep unsaved browser edits distinct from saved draft state. External updates
must not replace either silently. New drafts start from a freshly resolved
published definition; a resumed older draft displays that its base changed.

Trial creation pins the saved draft's exact bytes, digest and published code
source. Unsaved changes are saved as part of the action or the action explicitly
uses the saved revision; the UI must not imply it tried changes it omitted.
Ordinary requests and delegation continue to use published definitions.

## 4. Apply drafts through the service

Apply uses an isolated service-owned Git working area and the existing identity,
access and repository publication mechanisms. It must not commit unrelated
checkout files or require the registered workspace to be clean or up to date.

Implement the following publication behavior:

1. Read the saved draft revision and refresh its actual publication target.
2. Compare base, latest published content and draft. Preserve unrelated upstream
   changes. Merge compatible edits and validate the resulting definition.
3. When edits conflict, retain the draft and show the conflicting changes with
   a way to resolve them in the GUI. Do not force-push, overwrite newer work or
   require a person to repair a workspace in a terminal.
4. Commit only the intended definition changes and necessary repository pins.
   Publish with an expected-head check; if another writer wins, reconcile again
   or return a recoverable conflict with the draft intact.
5. Record the resulting source revision and read it back through the same
   published-source resolver used for ordinary requests. Mark Applied only
   when that resolver exposes the intended definition.

Resolve publication through submodules according to the actual project contract:
publishing definition bytes alone is insufficient if the project must advance a
gitlink to use them. Record intermediate outcomes and recover partial publication.

Make retries after timeout or a lost response recognize an already successful
apply. A failure must leave the draft available and distinguish failed,
in-progress and uncertain outcomes from success. Handle simultaneous applies,
workflow deletion/rename and invalid merged definitions explicitly.

After success, show the applied graph and clear the applied draft state without
discarding newer edits saved concurrently. A later external publication can
advance the displayed version; preserve attribution to the apply result.

## 5. Remove workspace maintenance from the ordinary flow

Remove the Published / Workspace (editing) split and Update workspace dependency
from ordinary workflow navigation. Keep checkout inspection and manual sync in
developer details or CLI for people who intentionally use them.

Inventory existing workflow-file edits before switching editing paths. Preserve
them and offer explicit import as drafts with their actual provenance. A stale
checkout that differs only because it is behind is not itself a user draft.
Local commits and uncommitted workflow changes need comparison with their own
base; do not automatically publish them or reset the checkout. Unrelated dirty
files must not block new viewing, drafts, trials or application.

Update normal editor routes, command help, project/run links and documentation
to agree on the new contract. Adapt any retained direct-checkout CLI operations
so they state their effect honestly; they must not masquerade as GUI draft saves
or application. Remove obsolete implementation paths where no longer needed.

## 6. Verify the product contract

Use meaningful service/integration tests for source selection, revision checks
and publication, plus browser checks for what the user actually sees.

| Scenario | Required evidence |
| --- | --- |
| Agent integrates or an external writer pushes a workflow change | Open project graph updates within the refresh bound without workspace sync; a new run records the updated source |
| Checkout is behind, dirty or missing the workflow | Published viewing, editing and applying work; unrelated local files are unchanged |
| Draft save and browser/service restart | Saved edit survives; ordinary execution still uses published content |
| Draft trial | Run records exactly the selected draft revision and its code source |
| Apply / Save and apply | New graph and new ordinary run use the applied content; previous runs retain their definitions |
| Concurrent upstream and draft changes | Compatible changes survive; conflicting edits remain recoverable and resolvable in the GUI |
| Two tabs, repeated apply, lost response or restart | No silent lost edits, false success or duplicated publication |
| Definition in a submodule | Both publication and ordinary execution resolve the intended version, including any required pin update |
| Published source unavailable or malformed | Graph communicates unavailable/stale/invalid state without substituting checkout content |
| Historical or retired run | Definition link reads that run's retained definition, not the latest project workflow |
| Existing local edits | Inventory/import preserves real edits without manufacturing drafts from ordinary checkout lag |

Run the repository's required checks and relevant regression suites. Measure
published refresh load and responsiveness with the registered project set so
the change does not repeat the earlier intake/heartbeat delays.

## 7. Deploy and leave a direct verification path

Commit and publish changes, deploy the coordinator and any worker/client changes
required by the final contract, and verify the running revisions. Inspect current
deployment procedures and state before operating services; keep machine details
and commands in ignored notes. Backward-compatible operation and rollback are
not goals; any migration must account for existing drafts, edits and run evidence.

Verify the original placeholder case on the deployed GUI: graphs reflect the
published definitions for the affected projects without synchronizing their
checkouts. Record the actual project inventory at execution time rather than
assuming the baseline count remains unchanged.

Use an identifiable test project for an end-to-end edit, trial, apply and ordinary
run. Also verify an agent integration updates the open graph. Check access from
another LAN PC. Leave direct links and a short Developer check requiring only
opening the graph and, if desired, performing a small edit.

Write `p20/report.md` with implemented behavior, source/deployed revisions,
executed checks, migration results, evidence locations and remaining limitations.
Update maintained WFE documentation and `devdocs/README_DEV.md` in the same work.
Completion requires the deployed user flow, not merely new source-selection
labels or a working workspace-sync button.
