# p13 ex1 Step 1 — reusable context parts

## What was traced

- **Drafts and saves today.** The project editor keeps one `draft`/`saved`
  pair for `project.yaml` (intent and goals in the Detail-only project card,
  rebuilt on every render). The workflow editor keeps one draft of the whole
  file; its intent textarea is a persistent element inside the Detail-only
  *Intent & approvals* panel. Neither editor offered `guides`
  (`project.yaml guides`, `execution.guides`): they were read-only in
  Workflow settings and not shown for projects.
- **Override drafts** lived only in the standalone `contextView.ts`, as one
  textarea shared by whichever scope was selected in the matrix.
- **Lifecycle.** `main.ts` vetoes navigation through each view's
  `isDirty()`; views dispose their live streams. Display mode is CSS on one
  DOM (`.s-only`/`.d-only`), so a persistent element keeps its draft,
  caret and disclosure through a switch; a rebuilt element does not.
  `keepingFocus` restores focus only to an element that is still connected.
- **Run input.** The browser can already record an additional instruction:
  `POST …/runs/<wf>/<run>/ops` offers `instruction.receive` (the same
  record a Zulip post or `wfe run instruct` produces; refused on cancelled
  or completed runs). The run view had no form for it.
- **API.** `GET api/context/override?scope=system` and the `PUT` already
  work without a project. Nothing in the server had to change for a
  system-only view.

## What was built (agdevworld, `workflow_editor/src/contextParts.ts`)

- **`OverrideEditor`** — one local instruction (system, project or
  workflow). Built once and kept. It owns:
  - its draft, Save / Discard changes / Remove, and the author name (stored
    per browser, shown next to Save in both modes);
  - refresh: a clean editor follows the saved text without moving the
    caret; a draft is never replaced, and a newer saved text is announced;
  - conflict recovery (*Use the saved text* / *Save mine over it*), with
    the draft kept on any failure;
  - the result message, which names the scope ("Saved for every run of
    workflow make…, nothing was started");
  - Detail-only file, revision, author, hash and revision history.
  `wider: true` marks an editor of a wider scope opened from a narrower
  editor; its hint and save message say so.
- **`ListEditor`** — goals and guide references edited in place. It
  reconciles rows rather than rebuilding them, so a refresh never takes the
  input being typed. Values go into the host's draft, and the host saves.
- **`sourceBlock`** — a repository-owned or wider-scope source, readable in
  place. Long text is collapsible; origin, revision and hash are in Detail.
- **`ContextSection`** — the collapsible Context section:
  - an always-visible summary (shared part, local state, how many inherited
    sources, unsaved changes);
  - **Shared with this project**: the host's own controls, with the host's
    save;
  - **Only in this environment**: an `OverrideEditor`;
  - **Inherited instructions**, with the scope named. A wider override has
    *Edit instructions for all projects* / *Edit project instructions*,
    which opens an embedded editor that keeps its draft when closed.
    Project defaults link to the project editor;
  - for a workflow, *What a new run is given (saved settings)*, which says
    when unsaved edits are not included.
  It follows the editor's selection (`setWorkflow`). Its project is the one
  the resolver reports, never chosen again. Open state is per viewer.
- **Overview reuse.** `contextView.ts` now uses `OverrideEditor` for its
  *Quick edit* and in the project chooser. `#/context` edits the
  instructions for all projects without choosing a project. Matrix cells
  link to the owning editor's section (`#/home/context`,
  `#/ws/<ws>/context`, `#/ws/<ws>/wf/<file>/context`). The rows are now
  called Shared / Local; storage paths are Detail-only.
- **Styles** for the section, editors, lists and inherited blocks, with
  narrow-layout rules.

`tsc` clean, `vite build` succeeds. Editors are integrated in Step 2.
