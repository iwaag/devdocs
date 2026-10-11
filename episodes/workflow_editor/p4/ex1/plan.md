# Workflow editor p4/ex1 — integrate the editor as agdevworld system code

## Goal and authority

Implement the corrected [migration braindump](../../design/migration/braindump.md) and [this correction's braindump](braindump.md): incorporate the workflow editor into agdevworld, then continue development of the RTS project through it.

The editor is an agdev system component. It is not a development project managed by its own ordinary project registry. The current operating catalog should contain the RTS project and its workspace, plus its shared `study-rts` repository. Future ordinary development projects remain supported.

This correction supersedes p4's requirement to register the editor as a project, execute an `improve-editor` run, and deploy from that project's repository. It also removes the need for the two editor workspaces. It does not introduce a privileged category of self-managed system projects or a new role-management system.

The earlier plan interpreted “continue editor development through agdevworld” as self-development through an ordinary project. The revised input explicitly calls for integrating the editor into agdevworld and continuing the RTS project there. Use that distinction as the implementation boundary.

This document is an implementation plan. Preparing it does not perform the cutover, unregister projects, remove workspaces, or deploy software.

## Starting point

The p4 reports describe implemented storage, Gitea, dashboard, and execution foundations. Their test results are historical evidence; rerun the relevant checks against the integrated code.

At planning time:

- The editor source was split into Gitea `developer/pj-workflow-editor` (registered as `gitea:132`).
- Two workspaces, `workflow-editor` and `workflow-editor-2`, refer to this same project. The second was used for setup commits.
- The running-service configuration points to `pj-agdev/.local/agdev/service/current`, currently a release of commit `9c29117` from that repository. The area launcher also runs that release's CLI.
- `agdevworld/nginx.conf` forwards `/wfe/` on the loopback-only port 8093 to the host service on port 8098. Agdevworld does not yet own the editor source.
- `pj-agdev/experiments/workflow_editor` remains a frozen source copy.
- Autolab invokes `wfe` through `agautolab/.local/wfexec.toml`. Its executor tests still default to the experiment source path.
- The operating registry also contains the RTS project `rts-vs-bot` (`gitea:131`) and the shared repository `study-rts` (`gitea:130`).

Reinspect working trees, branches, service configuration, queued jobs, and attempts before implementation. Do not assume the above commits or process states are still current.

## 1. Target ownership and layout

Keep the editor's source, tests, CLI, and service together as an internal package in the agdevworld repository:

```text
pj-agdev/
  agdevworld/                     # existing agdevworld Git repository
    src/                         # agdevworld views and navigation
    workflow_editor/             # system-owned editor package
      src/
      shared/
      server/
      cli/
      test/
      checks/
      docs/
      package.json
      package-lock.json
    scripts/                     # system build/deployment entry points
  agautolab/                     # execution adapter
  .local/agdev/                  # operating data and managed workspaces
    registry.json
    pj-rts-vs-bot/
    .local/exec/
```

`agdevworld/workflow_editor` is tracked directly in agdevworld's existing Git repository. Do not make it a submodule pointing to the former managed editor project. Do not give this package an operational `project.yaml`, an `improve-editor` workflow, or a workspace registration. Test fixtures may still contain project definitions in isolated test directories.

Retain the internal package's own dependency lockfile and build checks initially. This avoids coupling its Node service and CLI to the Phaser frontend while establishing one authoritative source repository. Update agdevworld's documented build/check entry points to include this package; a clean checkout must supply everything required to build the editor.

The host service may remain a separate process because it operates on host workspaces. Integration means that agdevworld owns its code, navigation, build, and deployment lifecycle; it does not require rewriting the service in Python or merging it into the static frontend process.

System code and deployed artifacts must be outside the ordinary workspace catalog and must never be selected by a project publication operation. Autolab uses the installed system CLI; an ordinary project does not supply the CLI implementation or deployment script.

## 2. Preserve the useful p4 behavior

Keep the following contracts while changing ownership and packaging:

- Gitea for operating project and shared repositories.
- `devdocs: directory | submodule`, with directory as the default.
- Run layout `devdocs/runs/<workflow-id>/<run-id>/` and devdocs as the work-record authority.
- Fixed execution definitions, replay-validated run state, cross-process locking, receipts, and stale-attempt rejection.
- A single autolab execution slot, study/do/talk, human waits, explicit interruption recovery, and publication checkpoints.
- Delegate definition/display only, with execution refused.
- Distinct global dashboard and per-project resource views, including adding shared repositories to a project.
- Zulip as a future coordination adapter; no new Zulip, front, forge, or archsage dependency.

Use one shared implementation for CLI and API operations. Preserve the RTS data and identifiers so existing links and runs remain usable where already on the p4 contract. No general old-format migration is required.

## 3. Source integration and system deployment

1. Compare the deployed release, both editor workspaces, and the frozen experiment. Identify any changes that exist in only one location, including uncommitted work, before selecting the import source.
2. Import the required editor code into `agdevworld/workflow_editor`. Record the source commits and file differences in the report. Bring across functionality and tests, excluding operating data, `node_modules`, generated builds, self-development definitions, and machine-specific settings.
3. Update assumptions about the package location in default paths, setup templates, fixtures, browser checks, and documentation. Keep operating area selection explicit so moving code does not accidentally create a new empty registry.
4. Add agdevworld-owned build, validation, and deployment commands. A release must be built from a chosen agdevworld commit and its editor package. It must not fetch or deploy from `developer/pj-workflow-editor`.
5. Build and validate the candidate before activating it. Retain a known working system release for rollback. If activation or the health check fails, restore the previous service and CLI target together.
6. Update the installed service definition, launchd template, and area CLI launcher to the system release. Keep the service and CLI on the same tested code version. Rewrite the old editor-specific deployment instructions accordingly.
7. Update autolab's executor configuration as needed and point executor integration tests at the new source location. Replace the experiment-specific test setting with a name appropriate to an installed or source editor package.

Existing release directories may serve as temporary cutover or rollback artifacts. They must not remain a reason to retain the former editor project in the operational registry. After successful cutover, no active launcher, build, test default, or deployment command may depend on either editor workspace or the frozen experiment.

## 4. Agdevworld UI and operating boundary

Keep `/wfe/` as the integrated editor route and retain the existing loopback-only exposure during this correction. Add or retain clear navigation to the agdev dashboard within agdevworld, with navigation back. Do not move the write API onto the LAN-facing listener merely to make the URL shorter.

The dashboard lists development projects and shared repositories. The editor is a system feature available through navigation, not a project card, workspace row, shared repository, or run target. System source and release details belong in system operation documentation rather than the ordinary project catalog.

Enforce this boundary in shared service/CLI operations, not just display filters:

- Unregister both editor workspaces, the editor project, and its root-repository catalog entry.
- Reject direct requests for the removed workspace/project identifiers; do not resolve them through a hidden or cached registration.
- Reject registration or execution on configured system code, service release, and control-data paths. Resolve real paths and check overlaps so a symlink or an ancestor directory cannot bypass the boundary. Check registration, request/start, and queue claim; a registry edit alone must not make a protected system path executable.
- Treat the retired editor repository as excluded from this operating catalog. Registration, obtaining a workspace, or importing it as a shared repository must not silently restore the old self-development route. Identify it by configured repository identity, not only a display name.
- Keep this exclusion in system configuration outside managed projects. Workflow access declarations cannot override it.

These checks protect supported project-management operations. They are not an OS sandbox: an agent with unrestricted shell access under the same account can still access other files. Do not claim that relocation alone prevents every possible agent mistake. OS-level isolation and general user-role authorization remain outside this correction's scope.

## 5. Catalog cleanup and cutover

1. Inventory current registrations, repository identities, runs, pending jobs, attempts, and publication checkpoints. Save a consistent recovery copy of the registry and execution metadata.
2. Prevent new editor jobs from being claimed. If an editor attempt is active, stop it through the execution controls and confirm process termination before removing its registration. Record any pending work; do not leave orphaned jobs that can restart it later.
3. Coordinate service/CLI changes with the executor. Do not replace the CLI under a live attempt. Let unrelated RTS work reach a safe boundary or stop it explicitly with a recorded reason before cutover.
4. Remove only the `workflow-editor` project, `workflow-editor` and `workflow-editor-2` workspaces, and their editor repository entry from the operating catalog. Preserve `rts-vs-bot`, `study-rts`, their paths, run records, and repository revisions.
5. Start the integrated system service and executor with the existing operating area. Confirm that the browser, API, and CLI use the same catalog and code version.
6. Verify that stale editor URLs, queued jobs, and direct API requests cannot start an editor run. Restart both services and repeat the catalog/read check to rule out stale in-memory state.
7. Retire the old workspaces and frozen experiment as active code sources after confirming their changes have been accounted for. They may be kept temporarily as unregistered archives. Deleting the Gitea repository or its Git history is not required to remove the self-management dependency.

The target operating catalog has one Project and one Workspace for RTS, plus its root Repository and the shared `study-rts` Repository. This is the cleanup result for the current environment, not a hardcoded restriction to RTS or to one project.

## 6. Validation

Use isolated fixtures for failure and exclusion checks. Run relevant tests against the integrated source and then exercise the actual operating configuration.

### Ownership, paths, and installation

- A clean agdevworld checkout builds the frontend and editor package without either old editor workspace, the frozen experiment, or the former editor Gitea repository.
- The system service and `wfe` launcher execute the installed system code. Trace their resolved paths and record the owning agdevworld commit.
- Editing a fixture project's files or running its workflow does not alter installed system source, service artifacts, or launch configuration.
- Removed editor IDs are absent from dashboard/API/CLI listings, and direct operations against them are refused.
- System-path registration is refused through supported entrances, including a symlink and an ancestor-path case. Queue claim refuses a manually injected registration targeting those paths.
- Registering or importing the retired editor repository cannot restore it as an ordinary run target.

### Existing product behavior

- Run editor contract tests, type checks, and build from the new package location, plus agdevworld's frontend build.
- Run autolab executor tests against the new CLI location and the relevant browser checks for dashboard, project editing, and execution.
- Exercise create/read/history behavior in both devdocs modes using fixtures.
- Confirm Gitea publication and retry, shared-repository listing and addition, delegate refusal, question/answer/resumption, and stop/restart behavior remain correct.
- Through agdevworld, verify the RTS project and its study repository, run history, answers, and publication state. Confirm the old editor rows remain absent after restart.
- Preserve the existing distinction between unavailable information and an empty catalog.

### Real RTS acceptance

Carry out one improvement specified or agreed upon by the person through the `improve-game` workflow in the RTS workspace. Record the person's scope answer, inspect the game and test results, record result acceptance or rejection, and verify the published files from a fresh Gitea checkout.

If a real RTS run was already completed before this correction, inspect its evidence and verify continued access after cutover. Reuse that evidence where sufficient; repeat real work only when needed to establish an unverified part of the corrected route. Do not invent a human request or acceptance to complete the plan.

No editor improvement run is a completion requirement.

## 7. Implementation sequence and deliverables

1. **Inventory and ownership decision:** establish the source to import and current runtime/data state; record the exact source/release/registry map.
2. **System integration:** import into agdevworld, update build and deployment, and connect the service and CLI without changing the operating catalog yet.
3. **Operation boundary:** implement protected-target checks and test direct API/CLI and queue paths.
4. **Cutover and cleanup:** safely retire editor jobs and registrations, activate the system release, and verify RTS preservation and restart behavior.
5. **Acceptance and documentation:** perform the remaining RTS verification, update operating instructions, and record results.

Write implementation and test evidence in `report*.md` here, ending with `report.md`. Update agdevworld development/operation documentation, launchd comments, autolab test/setup instructions, and any frozen-copy notices that still direct development to the former editor project.

Keep p4's earlier reports as historical records of what was implemented. Add a correction reference where their “next steps” would otherwise instruct the person to run `improve-editor` or deploy from its repository. The parent p4 plan must reflect the corrected goal and completion criteria.

## Completion criteria

- Agdevworld owns and supplies the editor implementation as a system component.
- The installed service and CLI can be built, deployed, and recovered without a wfe-managed editor project.
- The operating project/workspace catalog contains RTS only; `study-rts` remains available and correctly linked.
- Both editor workspace entries and the editor project/repository registration are removed from active use, with no pending job able to restore their execution.
- Supported management and execution operations reject protected system targets.
- P4's storage, Gitea, dashboard, and single-autolab behavior still work, with actual RTS acceptance established or explicitly reported as outstanding.
- No real `improve-editor` run or self-deployment is required to complete p4.
