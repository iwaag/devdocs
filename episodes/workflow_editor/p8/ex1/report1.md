# Workflow editor p8/ex1 — step 1: the display contract

**Source**: agdevworld `640810d` (from `8cda5c9`). **Deployment**: none. The installed release is still `8cda5c9`.

## Starting point

- **Checkout**: agdevworld `8cda5c9` on `main`, clean. It is the p8 release, installed on 2026-10-13.
- **p8 checks**: they were all green at that release (p8 [report6](../report6.md)). Step 6 runs them again.
- **Baseline**: the p8 screenshots in [../image_after/](../image_after/) are the comparison for this episode. They cover the editor and the run states. Dashboard, Project Editor and User nodes had no p8 captures; step 6 captures them in both modes.
- **Fixture**: everything in this step was looked at on the private state fixture (`checks/states.ts --serve`). It has its own registry, area and service on a loopback port. No production data was read or written.

## The preference (`src/displayMode.ts`)

- **Values**: `simple | detail`. The default is Simple. The stored value is `workflow-editor.display` in local storage. Anything other than `detail`, or storage that throws, reads as Simple.
- **A separate key**: the preference is independent of the graph's Compact/Mini key (`workflow-editor.mode`).
- **What it is**: a viewer preference. No project, workflow or run file changes when it changes.
- **One DOM for both modes.** The mode is an attribute on `<html>` (`data-display`).
  - An element marked `.d-only` is shown in Detail only.
  - An element marked `.s-only` is shown in Simple only.
  - Nothing else differs.
  - No form, field or section is built twice or rebuilt on a switch. Every draft, selection and disclosure state stays with its one element.
- **What a switch does besides setting the attribute**:
  - **Composition**: a switch requested during an IME composition waits until the composition ends.
  - **Scroll**: each mode's scroll positions are remembered per screen and restored on return. That covers the window and every element marked `data-keep-scroll` (so far the run view's side pane).
  - **Focus**: if the focused control is hidden by the new mode, the focus moves to the toggle's pressed button.
  - **Listeners**: subscribers (`onDisplay`) are told about the switch. Views remove their listeners on dispose. In step 1 nothing subscribes yet. The canvas does in step 3, because its card metrics are not CSS.
- **The control**: each screen's top bar has the same labelled control, `Simple | Detail`.
  - It is a group of two buttons with `aria-pressed`.
  - It is the last item of the bar, so it stays in the same place in both modes.
  - It never navigates, saves, submits or changes execution state.
- **Contextual entries** (`showDetail`, or `setDisplay('detail')` and then the existing reveal): every explicit way into a Detail capability switches first and then reveals its target. The entries are:
  - **Dashboard**: "New project"; "Workspace options" when a project has no available workspace.
  - **Project**: "Project settings"; "All runs (n)".
  - **Workflow**: the approval state, which is the entry to "Intent & approvals"; the validation summary when it reports a problem; the inspector's "More details".
  - **Run**: "Reports & files", "All records", the selected node's "More details", and every existing link into a side section (Execution details, Presentations, Coordination, Adjust future work).
  - **User nodes**: "+ New template" and "Edit template".

## Where each capability lives

The table lists each screen's three places for information and operations: shown in Simple, shown after a selection, and shown in Detail. Nothing was removed. Every item in the "Detail" column was visible in p8 and is now marked `.d-only`.

| Screen | Simple | After a selection (Simple) | Detail only |
| --- | --- | --- | --- |
| Dashboard | **Projects** (heading), New project, one tile per project with name, purpose (the intent's first line), status icons (runs waiting, in progress, unreadable; definition or workspace problems), the waiting runs with who holds the next move, Open project; a short Gitea problem when creation is impossible | — | registry path and read time, Gitea and execution-host chips, project ids, devdocs mode, root repository links, "read from … at …", workspace lines, Obtain a workspace, shared repositories, Create / Register / Shared repository / Register by path forms |
| Project | large name, purpose, "Project settings"; Request a run (workflow, your name, words); Recent runs (5) with a state icon, the workflow's name, the run's name and how long ago it changed, Open; Workflows with an icon, the name, one state (Approved / Draft / Changed since approval / errors / cannot be read), Open | — | the project form (id, name, devdocs, intent, goals, save), structure diagnostics, sub-repositories with branches, commits, remotes, dirty and publication chips, access per workflow, Add existing repository, Add by location, Workspaces, the run folder path and executor/input line, count chips, ready/failed lines, runs beyond the five most recent, workflow files and ids, approval chips, New workflow, the workspace switcher, Refresh |
| Workflow | name, the purpose line, one approval state (also the way in to Intent & approvals), validation only when it reports a problem, Save and its state, Add node, the graph | the node's name and description fields, its issues, "More details"; a connection's ends and "More details"; a generated step's purpose and its document | file and workspace line, the two approval pills, Intent & approvals panel (whole intent, records, approver, the two approvals), Workflow settings (policy, bindings, full validation), Compact/Mini, Auto-arrange, the policy chip, the canvas hint, node type, delegate target, appearance, repository bindings, connections, template provenance, Delete node / connection, a step's waits and instructions, ids |
| Run | the run's title, its execution and decision states, the live connection, the focus card (situation, the current human task, its forms and actions), "Read final report", "Original request", "Reports & files", the graph, "All records", Recording as | the node's state, description, recorded outcome, wait, failure, its report and a link to read it, the latest observation of a node at work, "More details"; an opened file | the run reference, status line with counts, workflow/run path and revision, executor situation chip, Open current definition, Run details (times, backend, attempts, provenance, the full position), graph chips, counts and legend, the full recorded-outcome list, exact times, question metadata, presentation revisions, the executor's attempt/process line, every side section (resolved questions, instructions, presentations, definition and Adjust, catalog, related runs, files, execution and publication, coordination, result and decision, history), node ids, type, template, times, notes and its questions, Refresh |
| User nodes | the heading and a one-line purpose, your name, each template's card (icon, name, purpose), selection with its preview, "+ New template", "Edit template" | the selected template's preview | ids, revisions, who changed it and when, expectations, the editing fields, Save, Discard, Delete, examples, the catalog file and generation, Refresh |

**Simple leaves out** the following: commit ids, remotes, branches, internal paths and file names, backend or model identifiers, exact timestamps, run references, routine technical explanations, and the per-step outcome list. Status is shown once on each screen. On the run, it is the title band's chips and the focus card; the graph head no longer repeats the counts. The breadcrumb names the project. On a run it ends at "Run", and the run's own title is the heading.

## The visible reduction on first opening

These observations come from the fixture at 1440 × 1000, before anyone opens or closes a section:

- **Project.** p8 opened with the project form, the sub-repositories (branch, commit, uncommitted and unpublished chips), the workspaces with branch @ commit, and every run with its counts. Simple opens with the name, purpose, request form, five recent runs and the workflows. Two of p8's three columns of technical facts are gone from the first view.
- **Completed run.**
  - **Gone from the first view**: the reference path, the counts line, the revision line, the executor chip, the six-line outcome list, the exact acceptance time, the graph's three chips with their counts, the legend and nine side-section labels.
  - **What remains**: the title, Completed and Accepted, "Accepted from Sample Person's answer", the stopped review, the three actions and the graph.

The visual hierarchy (sizes, icons, tiles, cards) is step 2's work. In this step the Simple screens are plain, with some unstyled icons.

## Verification so far

- `tsc --noEmit` and `vite build` pass.
- Both modes of Dashboard, Project, Workflow, completed Run and User nodes were opened on the fixture. There were no page errors.
- Not yet run: the browser checks. Several p8 checks assume Detail content (the dashboard forms, run counts, side sections). Step 6 updates them: they open in Detail where they test Detail capabilities, and Simple gets its own check.
