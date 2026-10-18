# p13 ex1 Step 2 — Home, project and workflow editing

## What changed (agdevworld `workflow_editor/src/`)

- **Home** (`homeView.ts`): a Context section, *Instructions for all
  projects*, above the projects in both modes. It sits outside the part
  that re-renders every 20 s, so its draft and caret are never replaced, and
  it re-reads the saved text on every dashboard refresh. No project needs
  choosing. The header's *Context* button became the Detail-only
  *Context overview*.
- **Project editor** (`projectView.ts`): a Context section between the hero
  and Boast, in both modes.
  - **Shared with this project** holds the only editors of project purpose,
    goals and guides (new: `project.yaml guides`). They are persistent
    controls fed from the view's one `project.yaml` draft, with *Save
    project.yaml*, validation for those fields, and each guide's saved
    content readable in place.
  - **Your instructions for this project** is the local override.
  - **Inherited instructions** shows the instructions for all projects,
    with an in-place editor of that scope.
  - The Detail project card keeps id, name and devdocs, and points to
    Context for the rest.
  - The hero has *Purpose & instructions*, which opens the section without
    switching mode. Its unsaved note says which kind is unsaved
    (`project.yaml`, local instructions or both).
- **Workflow editor** (`workflowView.ts`): a Context band under the title,
  in both modes, with a bounded height so the graph stays on the page.
  - **Shared with this project** holds the workflow intent (the same
    `#intent` control, moved here) and the workflow's own guides
    (`execution.guides`, new in the editor), with *Save workflow*: the
    same draft and save as the header's Save.
  - Also: the local workflow instruction; inherited project purpose, goals
    and guides (*Edit in the project* opens the project's Context); the
    project and all-projects instructions, each editable in place; and
    *What a new run is given (saved settings)*, which says when unsaved
    edits are not included.
  - *Intent & approvals* shows the saved intent it approves and links back
    to Context.
  - A project change on disk refreshes what is shown as inherited.
- **Saves stay separate.** Definition values go through the editor's
  existing save (`project.yaml` or the workflow file). Local instructions go
  through the p13 override API, each with its own Save and its own result.
  The summary and the navigation guard count both: `isDirty()` is the
  definition draft or any open context draft.
- **Routes** (`main.ts`): `#/home/context`, `#/ws/<ws>/context` and
  `#/ws/<ws>/wf/<file>/context` open the editor with its Context section
  shown, without changing the display mode. Matrix cells link there.
- **Narrow layouts**: the section's grids use `minmax(0, 1fr)`, its button
  rows wrap, and the project hero stacks below 640 px.

## Checks

New `checks/contextEditors.ts` (private fixture over the LAN name, plain
HTTP), which starts in Simple. Journeys 1–3, as far as Step 2 reaches:

- Home: instructions for all projects saved without a project. The message
  names the scope and says nothing started; the store holds the text.
- Project: the purpose has exactly one control. The system instruction is
  shown as inherited with its scope. A project instruction is saved "for
  every workflow of project p".
- Workflow: that instruction appears as inherited, with the project
  purpose and guide.
  - Intent and a local draft both survive a Detail/Simple round trip and a
    live refresh (`project.yaml` changed on disk, and the new purpose shows
    as inherited).
  - Leaving the page asks first.
  - The local save leaves the definition draft unsaved, and the preview
    says so. The definition is then saved through *Save workflow*.
  - A conflicting local save is refused, shows the saved text and keeps the
    draft; *Save mine over it* works.
  - *Edit instructions for all projects* names the wider scope, saves it,
    and creates no narrower override. Everything stays in Simple.
- A matrix cell link opens the project editor's section.

Results: all pass in Chromium desktop, WebKit desktop and Chromium at
390 px. In WebKit, fetches cut short by `page.reload()` are logged as
"access control checks"; the check ignores only those.

Regressions:

- `checks/context.ts` (p13): updated for the new wording; all pass.
- `checks/ui.ts` `openIntent` now opens Context for the intent and *Intent
  & approvals* for the approvals.
- `checks/displayMode.ts` uses the new location and the new hero wording.
  Its remaining failures — "accepted while cleaning up", "completed without
  a decision", then a timeout at the decision evidence field — happen on
  the Step 1 baseline too, in run views this step did not touch.
- `checks/simplerUi.ts` has the same three failures as the baseline
  (answer recorded, accepted while cleaning up, completed without a
  decision).
