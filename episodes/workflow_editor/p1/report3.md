# workflow_editor p1 — step 3 report: workflow editing

## Outcome

The workflow editor works end to end on the real fixture. In the UI I created a
workflow with all four node types, a branch, a join, a delegate and repository
bindings. I moved, connected, deleted, panned and zoomed, used both display
modes, and saved. Reopening returned the same graph and the same file text.
Results: 42 browser checks pass (`checks/step3.ts`), and 26/26 service tests
pass including 6 new persistence/HTTP tests. The type check and the production
build are clean.

## What was built

- **Top area** (follows `workflow_editor_compact.jpg`): editable name, the
  intent box "Workflow Intent (Ground Truth)", validation summary, save state
  with explicit Save (also ⌘/Ctrl-S), and the Compact/Mini switch. It also shows
  approval pills and actions, which are implemented now and exercised in
  step 5. There is no Run, Share or activity state.
- **Canvas** (`src/canvas.ts`): DOM cards on a transformed surface with SVG
  edges and arrowheads. Each type has its own color and line icon: study
  (book + magnifier, blue), do (gear + terminal, amber), talk (speech bubble,
  mint), delegate (branch, solid purple, showing the target workflow's
  *name*), plus a dashed card for an unknown type. Repository badges sit under
  each card: dark = readonly, amber = editable, red dashed = unresolved
  (undeclared binding or repository not in the project). A card with
  validation errors carries a red "!".
- **Interactions:** click/Enter selects. Drag moves. Dragging the right-hand port
  onto another card connects. Clicking an edge selects it, and Delete removes
  the selected node or edge. Dragging the background pans; Ctrl/⌘-wheel or pinch zooms;
  plain wheel pans; +/−/Fit buttons. Deleting a node removes its incident edges
  and its position from the draft.
- **Inspector** (`src/inspector.ts`): node type, name, description, delegate
  target (the project's workflows by id; unknown targets shown as missing),
  repository bindings as checkboxes with access badges, "Waits for" / "Then"
  connection lists with delete, "Connect to" picker, and Delete node.
  With nothing selected it shows the workflow's bindings: path picked from the
  project's actual repository list (unresolved paths stay visible as such),
  access, add/remove (removing also clears node references), and the full
  validation list (clicking an issue selects its node or edge).
- **Layout** (`src/layout.ts`): stored positions are in compact units. Mini mode
  draws the same positions scaled by 0.66 with smaller cards, so both modes show
  the same arrangement. Nodes without positions are placed by rank (longest
  path), in file order, below occupied slots. The first move or add stores all
  current positions so nothing jumps. "Auto-arrange" re-lays the graph by rank,
  which is a layout-only change. New nodes go to a free slot right of the
  selected node (or near the view centre) and are panned into view.
- **Validation** runs live on the draft with the service-supplied project
  context (repositories, workflow IDs and their delegate targets).

## Evidence

`checks/step3.ts` on a fresh seed (screenshots `step3-*.png` in
`pj-agdev/.local/workflow-editor/screenshots/`):

- Created `release-check` from the project screen; set a multiline intent;
  added bindings tools (study/tools, readonly), runtime (wedo/runtime,
  editable), docs (devdocs, editable).
- Added six nodes of all four types via the toolbar, edited names/descriptions
  and bindings, and set the delegate target `repo-setup`.
- Connected five edges via the inspector and two by port drag. A deliberate
  cycle was reported. Deleting the temporary node removed it and its two incident
  edges; an edge selected and deleted with the Delete key was re-added.
- Auto-arrange put the two parallel branches in one column on different rows.
  A drag moved a node by exactly the dragged distance. Zoom enlarged the cards
  and a background drag panned by the dragged distance.
- Mini mode showed the same 5 nodes and 5 edges without descriptions; the full
  description was available by selecting a card.
- Saved file: 5 nodes, 5 edges, all four types, study → two successors
  (branch), verify ← two predecessors (join), delegate target by id, binding
  access, node bindings, positions for every node, multiline intent, and the
  creation comment kept.
- Reload: same node ids and positions, same edges, Save disabled (clean), and
  the file is byte-identical.
- `onboarding` (no stored layout) renders 6 nodes / 6 edges by automatic layout.
  It shows "→ Repository Setup", readonly and editable badges, and opening it
  does not write.
- `draft-gaps` shows two unresolved badges, a missing delegate target and "4 errors".

Service tests (`test/persistence.test.ts`): a UI-style edit round-trips to
an equal model, keeps the header comment, and continues flow style. A no-op save
does not write (mtime unchanged). Deleting a node removes exactly its entries.
An unwritable directory makes the save fail with `EACCES`, leaving the old file
and no temp file. Malformed or anchored content on disk is never overwritten. HTTP
checks cover foreign origin 403, configured origin 200, no-origin 200, wrong
content type 415, foreign Host 403, path traversal 400, wrongly shaped model 400,
and unavailable workspace 409.

## Deviations and notes

- **Readonly as an editor restriction:** p1's editor has no operation on
  repository *contents*. It writes only definition files and runs
  `git submodule add`. So `readonly` is shown (badges, project access column)
  but there is nothing for it to block yet. It is a declaration, as the contract
  says, and is not filesystem isolation.
- Node placement first stacked new nodes at +40px offsets, so the cards
  overlapped and a drag-connect landed on the wrong card. It was replaced
  by free-slot placement. The canvas also turns any programmatic scroll
  (focus, `scrollIntoView`) into a pan, so cards, edges and controls stay
  aligned.
- The browser checks fit the view before clicking cards. Playwright can no
  longer scroll an `overflow: hidden` canvas, which is now intended behaviour.
- Ctrl/⌘ + wheel zooms, and plain wheel pans (trackpad-friendly). Mouse users
  zoom with the buttons or Ctrl-wheel.
