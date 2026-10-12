# Workflow editor p8 — step 2: a readable definition screen

Changed in agdevworld `workflow_editor/`: `src/workflowView.ts` (rewritten around the new layout), `src/inspector.ts`, `src/icons.ts` (interface glyphs), `src/styles.css` and `src/canvas.ts`. In `canvas.ts`, `events` is now public, and `focusCard`, `frame` and `dispose` were added. Step 3 fills in `frame` and `dispose`.

## What the screen is now

- **Header**: a slim dark bar of 52 px. It holds the breadcrumb (Projects / project / Workflow), Refresh, the save state and Save. ⌘S still saves.
- **Title band** (light):
  - the editable workflow name;
  - a one-line purpose preview, which is the intent's first line (the whole intent is in its title);
  - the file and workspace;
  - the approval pills, always visible: "Intent: approved / changed since approval / draft", the same for Definition, and "Author approved" when both are approved;
  - the validation chip;
  - the two explicit entry points, **Intent & approvals** and **Workflow settings**.
- **Intent & approvals** opens a panel under the title band. It holds the authoritative intent textarea, each approval's record (who, when, or "Last approved" when stale), the declared approver, the two separate approval buttons and the reason they are disabled: unsaved draft, read-only file, or validation errors for the definition. The intent textarea and the approver input are built once, so closing and reopening the panel keeps what was typed. The approver input is no longer rebuilt on every approval-state refresh; before, typing in it during a recalculation could be lost.
- **Workflow settings** opens the inspector column with the documentation policy, the repository bindings (path, access, remove, add) and the full validation list. Each part has an addressable heading. The validation chip opens settings with the focus on *Validation*, and the policy chip in the toolbar opens settings at *Documentation policy*.
- **Validation**: a summary chip ("Valid" / "n errors, m warnings") instead of a status line. Clicking an issue selects its node or connection on the canvas; an issue about a binding opens settings at the bindings, which are marked. An issue therefore never leads to a control that is not on screen.
- **Toolbar**:
  - **Add node ▾**, a menu with "Basic types" (each with a one-line purpose) and "Your templates". The templates keep their color, icon, provenance tooltip and `data-template`. The menu also shows a catalog error and links to User nodes.
  - Compact / Mini and Auto-arrange.
  - On the right, the policy chip ("Default documentation: plan talk, 5 reports, final report").

  The menu supports the arrow keys and Escape. It closes on an outside click, and it is disabled while the file is read-only.
- **Inspector**: closed when nothing is selected. Selecting a node, a connection or a generated step opens it, and so do the two entry points. Every panel has a close button (×), and Escape closes it as well. On close, the focus returns to the opener: the settings button, the validation chip, or the node's card. Clicking the canvas background still deselects. A selected card that the opening inspector would cover is brought into view.
- **Node inspector**: Name and Description first, then Type, and the delegate target where relevant. The rest are expandable groups, each with a one-line summary:
  - **Appearance**: "icon present · color teal";
  - **Repositories**: "app · editable", open by default;
  - **Connections**: "Verify → this → Human check";
  - **Made from your template**: "present · revision 1".

  Node issues and Delete node come last. Group open state is remembered across inspector rebuilds, which happen after every structural edit. The inspector keeps its scroll position when it is rebuilt for the same selection.
- **Generated steps** stay inspectable and read-only. Their panel no longer has a "Workflow settings" button: the title band has one.

## What did not change

- **Draft and file safety**. There is one draft. Every panel edits it in place, and opening, closing or switching panels neither saves nor discards. The external-change banner, Reload from disk, save errors, the read-only state of a missing or unusable file, the raw view, and the unsaved-navigation guard (`isDirty`) are unchanged.
- **Approval semantics**. Both approvals still need a saved draft, intent needs text, and the definition needs no validation errors. The digests are the same.
- **Event handling** for definitions and the inspector's "do not rebuild while typing" rule.

## Checked in the browser

On the private state fixture (`checks/states.ts --serve`) with an ad-hoc Playwright probe:

- **Draft kept**. A description typed into Verify's inspector survives closing the inspector (× and Escape) and selecting the node again. The save state says "Unsaved changes". Closing returns the focus to the Verify card.
- **Validation chip**. It opens the settings with the focus on the Validation heading.
- **`broken.yaml`**. The error banner names the YAML problem, the raw text is shown, and Add node is disabled.
- **Screens**. The default screen, node selection, the Add node menu, the Intent & approvals panel, and settings at 1280 × 800 render as intended. There were no console errors.

The existing browser checks address parts of the old layout: the intent textarea was always visible, the template buttons were on the toolbar, and the status line had `.v-summary.error` text. They are updated in step 6, where the full validation runs.

## Left for step 3

On a small graph the canvas still opens at the old fit scale, up to 1.2: `tidy.yaml` at 1280 × 800 shows huge cards. When the inspector opens, the canvas narrows and the view is not adjusted. The cards themselves are unchanged.
