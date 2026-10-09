# Workflow and project editor — p1 plan

## Goal and agreed scope

Build a standalone MVP under `pj-agdev/experiments/workflow_editor/` that edits
Git-backed project and workflow definitions through both a browser UI and an
ordinary text editor or agent IDE. Prove the complete editing cycle with real
local repositories before integrating it into agdevworld or executing workflows.

Inputs:

- `braindump.md` in this directory.
- `../design/braindumps/braindump.md`, `braindump2.md`, and `braindump3.md`.
- `../design/idea1/workflow_editor_compact.jpg`, `workflow_editor_mini.jpg`,
  and `project_editor.jpg`.
- The implementation proposal accepted in this conversation.

**Developer decision: assume edits do not conflict in the MVP.** Human UI edits
and external file edits are sequential. Do not implement revision-based writes,
three-way merging, field-level patches for concurrency, conflict resolution UI,
collaborative editing, or agent worktree integration. External-change reflection
remains in scope; concurrent editing guarantees do not.

## Data ownership and repository layout

Use one YAML format for the first version. Files remain understandable and
editable without the UI. No database is the authority for definitions.

| Information | Authority |
| --- | --- |
| Project schema, stable ID, name, intent, and goals | Root `project.yaml` |
| Submodule paths and repository locations | Root `.gitmodules` |
| Recorded submodule revisions | Git gitlinks |
| Actual checkout, initialization, and uncommitted changes | Workspace Git state |
| Workflow definition, approvals, and layout | `devdocs/workflows/*.yaml` |
| Registered workspace locations and host-specific settings | Ignored local configuration |

The project root is a Git repository, conventionally named `pj-<name>`. Its
`devdocs/` is a required submodule. Other submodules may use `study/`, `wedo/`,
or arbitrary paths. The study/wedo naming conventions categorize repositories;
they do not prescribe node types or access rights. Every workspace has an
ignored `.local/` directory.

Keep `project.yaml` small; do not copy repository URLs, branches, commits, or
workspace inventories into it. Discover workflows from `devdocs/workflows/`.
Keep machine addresses, absolute local paths, and credentials out of tracked
implementation files and examples.

## Workflow definition contract

Define and document `ag.workflow.v1` with these fields:

- `schema`, stable `id`, and display `name`.
- `intent`: multiline author intent, the truth against which the rest is reviewed.
- `repositories`: keyed bindings with project-relative `path` and
  `access: readonly | editable`. Resolve sources through `.gitmodules`, with
  explicit support for the fixed project root and devdocs repositories.
- `nodes`: stable IDs mapped to `type`, `description`, repository binding IDs,
  and a target workflow ID for a `delegate` node.
- `edges`: explicit `from` and `to` node IDs.
- `approvals`: separate intent and definition approval records.
- `layout`: optional node positions; automatic layout when positions are absent.

Node types are `study`, `do`, `delegate`, and `talk`. Names and descriptions can
change without changing IDs. A delegate resolves another workflow by ID within
the same project, independent of its display name or filename.

For p1, an edge means that its target waits for its source to complete. Multiple
outgoing edges express parallel branches; a node with multiple incoming edges
waits for all predecessors. A delegate waits for its target workflow to complete.
Parallel work beside a delegate is another branch. These are definition semantics,
not an execution engine implemented in this phase.

Validate schema versions, IDs, node types, repository bindings and paths, edge
endpoints, graph cycles, workflow ID uniqueness, delegate targets, and recursive
delegation. Incomplete drafts may be saved, but invalid definitions cannot be
approved. Report missing or uninitialized repositories explicitly. Validation
checks structure and references; it does not claim that a graph fulfills its intent.

Access is workflow-specific: the project view labels access in the context of
the selected workflow. `readonly` is a declaration and an editor operation
restriction in p1, not filesystem isolation from external editors or agents.

Keep formatting and comments where practical with a round-trip YAML document
representation. Do not build a bespoke text-patching engine for the MVP. Document
the supported YAML subset and reject unsupported constructs without rewriting
the source. Preserve unrelated fields during supported edits.

## Saving and reflecting external changes

- The filesystem is authoritative; the UI holds a draft until explicit Save.
- Saving writes the definition file, not an implicit Git commit or push.
- Observe external file changes using a simple watcher or bounded polling.
  Reload automatically when the UI has no unsaved changes.
- With an unsaved UI draft, retain it and show that an external change was
  detected. Require explicit reload/discard before adopting external content;
  do not add merge or conflict-resolution machinery. This guard is not a
  concurrent-write guarantee.
- On malformed external YAML, retain the last valid rendering, show the parse
  error, and prevent that stale rendering from being saved over the invalid file.
  Recover after the file is corrected.
- Use a temporary file and atomic replacement to avoid partially written UI saves.
  This protects file integrity, not against simultaneous external writers.
- Keep save failure visible and retain the draft for retry.

## Approval semantics

Record the approved content digest, declared approver, and time separately for:

1. Intent approval, covering `intent`.
2. Definition approval, covering intent, repository bindings, node descriptions
   and types, delegate targets, and edges.

Specify canonicalization and test it. Exclude approvals themselves, layout,
comments, and formatting from semantic digests. Editing semantic content makes
the corresponding existing approval stale without erasing its record. Intent
changes make both approvals stale; a layout-only change makes neither stale.

Approvals concern the definition, not permission to execute or acceptance of
results. They do not attest to repository contents or authenticate the author:
the YAML can be edited directly. Display draft, approved, and changed-since-approval
states accurately. Approval actions operate on saved, valid content.

## Screens

### Project editor

- Edit project name, intent, and goals.
- List submodules grouped by study/wedo/other, with path, source, recorded commit,
  actual HEAD, initialization, and dirty state. Show detached HEAD accurately.
- Add a local repository as a submodule at a chosen project-relative path. Show
  errors and partial state if Git cannot complete the operation; do not reset
  unrelated changes. Destructive submodule removal is outside p1.
- List workflows, create a workflow, and open its editor.
- List locally registered workspaces and switch the selected workspace. Distinguish
  local registration from observed availability; do not invent remote host state.

### Workflow editor

- Top area: name, intent, validation results, save state, and approval state/actions.
- Canvas: distinct icons and colors for all four node types, directional edges,
  and repository badges under nodes.
- Inspector: node type, description, repository bindings, and delegate target.
- Add/delete nodes, add/delete connections, move nodes, pan/zoom, and save.
  Deleting a node also removes its incident edges from the draft.
- Compact and mini modes render the same graph with different detail density.
  Full descriptions remain accessible through selection.
- Create and edit repository bindings using the project's actual repository list;
  direct-file references to missing repositories are displayed as unresolved.

Follow the supplied concept images for composition and visual vocabulary. Do not
show working Run/Share actions, execution history, or fabricated activity states.

## Implementation boundaries

Use TypeScript and Vite for the frontend, DOM cards and SVG connections for the
graph, and a small local service for file access, observation, and Git operations.
Choose the service implementation and YAML parser during step 1; keep the HTTP
boundary independent of the UI. A browser alone cannot supply these filesystem
and Git operations.

Separate definition parsing/validation, approval digests, Git workspace inspection,
file persistence, and UI rendering. Register workspace roots explicitly, bound
file operations to those roots, and invoke Git with argument arrays. Bind the
service locally and limit browser write access to the editor's configured origin.

Existing agdevworld `src/sessionGraph.ts` provides visual precedents for compact
cards and SVG edges; its conversation tree is not the editable DAG model.
Existing autolab initialization handles `main/direction/devlog` and pattern-managed
projects. Do not migrate those projects or route the MVP through their initializer.

Create fixture repositories with `git init` under
`pj-agdev/.local/workflow-editor/`. Use local sources and real submodules, with
at least two workspaces cloned from one project. Use portable relative submodule
URLs within the fixture layout. If local transport needs enabling, enable it only
for the fixture command, not through a global Git setting.

Commit fixture submodules before committing the root gitlinks when preparing a
reproducible snapshot. A seed script and synthetic examples belong in the tracked
experiment; generated repositories and machine-specific registrations stay ignored.

## Implementation steps and evidence

1. **Contract and fixtures.** Finalize project/workflow schemas, canonicalization,
   YAML subset, and save semantics. Build a repeatable fixture seed with all four
   node types, parallel branches and an all-predecessor join, another workflow for
   delegation, and readonly/editable bindings. Include invalid-reference examples.
2. **Project read/write surface.** Implement workspace registration and inspection,
   project metadata editing, local submodule addition, and workflow discovery and
   creation. Verify results against actual Git state, including an uninitialized
   submodule, dirty checkout, and detached HEAD.
3. **Workflow editing.** Implement the canvas, inspector, connections, repository
   bindings, layouts, display modes, validation, and explicit save. Reopen saved
   files and verify that the same graph and text return.
4. **External editing.** Verify sequential IDE/file edits and UI edits in both
   directions. Exercise malformed external content, corrected content, failed
   saves, and the unsaved-draft guard. Do not implement concurrent editing tests
   or claim concurrent-write safety.
5. **Approval and end-to-end trial.** Add approval actions and stale-state display.
   Run the complete scenario below, capture screenshots in ignored local storage,
   and document the outcomes and remaining limitations in `report.md`.

Run focused tests for graph/reference validation, semantic digest behavior,
filesystem save/reload behavior, and actual Git fixture inspection. Use browser
checks for graph operations, inspector editing, modes, and readable error states.
Run the frontend build and the experiment's applicable checks before completion;
no external service deployment or existing-agent runs are required.

## Acceptance scenario

1. Seed local source repositories, a project with devdocs/study/wedo/other submodules,
   and two cloned workspaces without Gitea.
2. Open workspace A and inspect project intent, goals, repositories, and workflows.
3. Create/edit a workflow in the UI, including all node types, branching, joining,
   a delegate reference, and repository bindings; save and reopen it.
4. With the UI draft saved, edit the YAML through an external editor and observe
   the corresponding UI update. Then edit in the UI and inspect the saved YAML.
5. Demonstrate validation and recovery for a missing reference and malformed YAML.
6. Approve saved intent and definition. Change a node description and verify stale
   definition approval; change intent and verify both stale. Reapprove, move a
   node, and verify that layout changes preserve approval.
7. Commit the relevant fixture submodules and root references, publish only to
   their local fixture origins, then update workspace B explicitly. Confirm that
   project metadata, workflow content, approvals, and layout are reproduced.
8. Record what passed, any deviations, how to start the editor, and the explicit
   assumption that writers take turns.

## Deferred beyond p1

Workflow execution, agent chat inside the editor, Gitea integration, agdevworld
integration, migration of existing projects, remote workspace discovery/deployment,
conditional branches, loops, retries, failure transitions, author authentication,
OS-level readonly enforcement, and robust concurrent editing are outside this MVP.
