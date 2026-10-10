# Workflow editor p2/pre1 — prepare the human/agent authoring trial

## Goal and scope

Make the p1 editor ready for a person to open VS Code, ask an IDE agent to
establish a project and create one workflow, adjust that workflow in the browser,
and ask the agent to continue from those adjustments.

Inputs are `../braindump.md`, `../../p1/report.md`, the experiment's `README.md`
and `docs/contract.md`, and the accepted p2 execution proposal. Implementation
stays in `pj-agdev/experiments/workflow_editor/`.

Pre1 delivers the missing setup, tools, documentation, and update reliability,
with a reproducible readiness check. The actual human-led usability trial belongs
to p2; passing automated checks is not evidence that the person completed it.

The MVP assumption remains: **writers take turns**. Keep external-change
reflection and the existing unsaved-draft guard. Do not introduce revision-based
writes, merge engines, conflict resolution, collaborative editing, or worktree
integration for concurrent writers.

## Starting point

P1 already provides YAML definitions, project metadata editing, submodule addition,
workflow creation and graph editing, validation, approvals, and layout controls.
The local service polls definition files every second and sends SSE events while
a browser is connected. Polling is shared by clients of the same workspace.

Gaps to close:

- The seed script creates disposable fixtures; it is not a normal project creator.
- Workspace registration requires editing an ignored JSON registry by hand.
- The README and file contract do not form a complete IDE-agent entry point.
- Existing HTTP operations are not exposed as a documented, convenient agent tool.
- Git-state-only changes are not watched. Reconnection, event coalescing, and
  creation/deletion/rename need explicit update-reliability evidence.

## Deliverable 1: ordinary project creation and registration

Provide shared service operations, browser actions, and CLI commands for creating
a new project and registering an existing workspace. `wfe` is the proposed CLI
name, not a tool already available at the start of this phase.

Project creation accepts a destination, stable ID, display name, intent, and goals.
It creates a root Git repository, `project.yaml`, `.gitignore` including `.local/`,
and a devdocs repository mounted as a submodule, then registers the workspace.
Initialize and commit the minimum repository content needed to make the submodule
usable. Describe these initial commits in the operation's help and result; later
definition saves continue to mean file saves, not implicit commits or pushes.

Use local repositories, with no Gitea dependency. Support a newly created local
devdocs source or an explicitly supplied existing source. Keep source repositories
outside the disposable fixture tree and use relocatable relative submodule URLs
where the chosen layout permits. Enable local Git transport per operation when
needed, without changing global Git configuration. Read the user's Git identity;
report a missing identity instead of silently borrowing the fixture's identity.

Registration accepts an existing workspace, observes it, and records its actual
root. Report incomplete project structure with actionable diagnostics. Repeating
registration must not create duplicate rows. Do not overwrite an occupied project
destination or unrelated registry entries. If creation fails partway, identify
what was created and support an explicit continuation or repair without deleting
the user's work.

The browser has visible Create project and Register workspace actions even when
there are no usable registrations. Missing registry files produce an empty setup
state; malformed registries produce a readable error, not an empty list that hides
the problem. A successful action opens the project without restarting the service.

## Deliverable 2: one documented tool surface for agents and people

Add a thin CLI around existing domain/service operations. Do not implement a
second validator, approval algorithm, or layout algorithm. Select and document
which commands require the running local service; explain unavailable-service
errors and the actual startup command.

At minimum expose project creation, workspace registration/listing, workspace
status, workflow validation, approval, and automatic layout. Reuse the existing
submodule-add operation through the same tool surface. File editing remains a
first-class way to author project metadata and workflows; do not force every edit
through the CLI. Provide useful human-readable results, exit codes, and structured
output where an agent needs to inspect results.

Each command's help says what it reads or changes, its inputs, and what its result
means. The top-level help is a concise index. Examples name commands that really
exist. Verify invocation from the intended authoring directory, not only from the
experiment's checkout; arrange a working executable or documented launcher there.

Approval commands record the declared approver consistently with the UI. They do
not infer or invent a person's approval from successful validation. Existing p1
approval semantics and direct YAML editing remain unchanged.

Audit the following operation matrix against actual entry points:

| Operation | Person | IDE agent |
| --- | --- | --- |
| Create project / register workspace | Browser UI | CLI |
| Edit intent, goals, workflow graph and descriptions | Browser UI or VS Code | File editing or API |
| Add submodule | Browser UI or Git | CLI or Git |
| Inspect validation and workspace state | Browser UI | CLI |
| Approve / auto-arrange | Browser UI | CLI |
| Rename/delete definition files and repair malformed YAML | VS Code | File editing |
| Inspect diffs, commit, and use Git history | VS Code or Git | Git |

Both parties may also use the CLI. Equal capability does not require reproducing
all Git or filesystem operations as browser buttons. It does require a usable,
documented route for each party: no raw JSON construction as a person's only route
to an ordinary editor operation, and no browser-only operation inaccessible to the
agent. Record any remaining asymmetry explicitly and resolve it before handoff.

## Deliverable 3: a short, usable IDE entry point

Prepare an ignored authoring area, proposed as
`pj-agdev/.local/workflow-editor-p2/`, separate from the resettable p1 fixture.
Keep its project workspaces, local sources, and registry separate from browser-test
data. A setup command may create this area and its guide, but must not pre-create
the human's project or its workflow.

The person opens this area in VS Code and starts the IDE agent with this area as
its working directory. Projects are created beneath it; subsequent project work
uses the selected project root. The setup output identifies the actual directory,
tool invocation, service startup, browser URL, and initial prompt. Machine-specific
paths belong in ignored generated files, not tracked examples.

Provide a short `AGENTS.md` there, generated from a tracked template. Its head says
who the agent is, who it collaborates with, and the available tools. Its body states:

- The person and agent are authoring Git-backed projects and workflow definitions.
- The definition files are the authority and their locations are discoverable.
- File editing, shell, Git, and the editor CLI are available; tool help describes
  usage, and `docs/contract.md` describes the file format.
- The task authorizes project creation, registration, file editing, and necessary
  local Git operations in the authoring area.
- Browser and external editing take turns in this MVP.

The initial reading path is explicit: this short guide; the tool's help when an
operation is needed; the file contract when authoring definitions; and, for an
existing project, `project.yaml`, `.gitmodules`, and the relevant workflow files.
Do not require reading historical episode reports or implementation code to begin.
Generate working links or paths to the canonical contract and help rather than
maintaining another copy of their contents.

Suggested initial prompt:

> Create a project in this authoring area for <purpose>. Clarify its intent and
> goals, establish the repositories it needs, and create one workflow. Make the
> result available in the editor for me to review and adjust.

Do not add model/provider quotas, API-cost restrictions, blanket approval gates,
or speculative security rules to the guide. Keep existing local-service boundaries;
this phase is not a reason to remove them or build a new permission system. A new
guide rule arising from a trial failure retains the evidence reference.

## Deliverable 4: measured, reliable external-change reflection

Start with the existing 1000 ms polling interval. Measure before changing the
watching mechanism. Do not treat a shorter interval as the default solution.

Measure file-save completion to visible UI update, event counts, data reloads,
Git subprocess counts, file-read activity, and process CPU during idle and editing.
Use a representative project with one workflow for the intended p2 task and a
larger synthetic case to expose repeated full refreshes. Record file/repository
counts, repeat count, latency distribution, and load observations with the results.

Initial usability target: under the representative local workload, ordinary valid
saves should normally appear within about two seconds. This is a target to verify,
not a claim about p1 or a hard guarantee across machines. Pick the final interval
and refresh strategy from the measurements and document the tradeoff.

Explicitly exercise and fix demonstrated gaps in:

- Project and workflow text edits, including atomic-rename saves.
- Several related files saved in a burst: coalescing must retain every required
  refresh rather than replacing a content refresh with a context-only refresh.
- Workflow creation, deletion, and filename changes, including when the affected
  workflow is currently open. Do not silently recreate a deleted file.
- SSE disconnect/reconnect, service restart, and edits made while disconnected:
  re-read current state on reconnection without depending on a missing old event.
- Registry changes and switching workspaces. Requests for a previous view must
  not update a disposed view or replace newer state.
- Malformed intermediate YAML followed by a valid save; preserve the existing
  visible-error and recovery behavior.
- Git HEAD, index, dirty state, and submodule initialization changes that do not
  modify the watched definition files.

Prefer event coalescing and targeted refreshes before replacing the watcher.
Separate relatively cheap definition observation from heavier Git inspection;
evaluate a slower Git refresh while the relevant view is open and an explicit
Refresh action. Keep an SSE outage visible rather than presenting an indefinitely
stale screen as current. No connected viewer should mean no periodic watch work.

If polling remains adequate, retain it. Consider filesystem notifications only
if measurements show a material shortcoming; include atomic saves and reconnect
reconciliation in that decision. Do not add simultaneous-writer guarantees.

## Implementation sequence

1. **Baseline and capability audit.** Reproduce the current startup and a sequential
   file/UI edit on isolated data. Map the operation matrix to existing functions
   and missing entry points. Capture baseline latency/load and the event-loss
   cases above. Finalize the minimal CLI and local project/source layout.
2. **Create/register and CLI.** Extract shared operations as needed, add the browser
   setup actions and CLI, and prove project creation from an empty authoring area.
   Check retry/partial failure and existing-project registration. Fill the operation
   matrix with real commands or UI paths, not planned names.
3. **IDE handoff.** Add the reusable guide template and non-destructive authoring
   setup. Verify help, contract links, executable availability, initial working
   directory, and prompt using a fresh setup with no fixture assumptions.
4. **Update reliability and tuning.** Fix the reproduced notification gaps, measure
   again, and select the observation intervals. Keep the p1 turn-taking assumption
   and current save semantics. Record before/after evidence.
5. **Readiness rehearsal and report.** Run the scenario below on separate rehearsal
   data. Produce `report.md` here with check results, actual startup instructions,
   capability matrix, update measurements, and any limitations. Leave the person's
   p2 authoring area ready without doing their project-creation trial for them.

## Validation and completion criteria

Run focused tests of new shared operations, CLI/UI parity, setup without a registry,
and notification recovery, plus the existing experiment type checks, tests, and
production build. Browser checks must use their own registry and repository tree;
do not run resettable p1 checks against the human's authoring area.

The readiness rehearsal must demonstrate:

1. Starting from the documented working directory, discover tools and the contract
   without reading implementation code or receiving undocumented assistance.
2. Create and register a new local project with devdocs as a real submodule; write
   one workflow through the documented agent-accessible tools and file edits.
3. Open it in the browser and adjust descriptions, edges, and layout; save.
4. Read those changes through the agent-accessible path, make a further sequential
   file edit, and observe it in the browser without manual page reload.
5. Exercise both parties' validation, approval, and auto-layout routes against the
   same underlying logic. Check Git operations through VS Code/Git documentation.
6. Observe correct recovery after notification interruption, malformed content,
   and definition creation/deletion/rename. Check Git-state refresh separately.
7. Confirm that setup and tests do not overwrite an existing project, registration,
   or the person's prepared authoring area.

Pre1 is complete when these checks pass, the operation matrix has no unexplained
asymmetry, and the person has concrete instructions for starting the p2 trial.
Record the actual limits of the measured update behavior. Do not mark p2's
human-led trial complete based on this rehearsal.

## Out of scope

Workflow execution, agent chat embedded in the editor, Gitea integration,
agdevworld integration, existing-project migration, remote deployment/discovery,
new authentication or permission systems, comprehensive browser Git management,
and concurrent editing remain outside pre1.
