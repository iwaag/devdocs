# Workflow editor p8 — step 3: a rebalanced canvas

Changed in agdevworld `workflow_editor/`:

- `src/canvas.ts`: cards, framing, resize, disposal and the focus hint.
- `shared/layout.ts`: card metrics only. Compact is 240 × 128 (was 232 × 132) and mini is 156 × 64. The arrangement spacing (`STEP_X` 300, `STEP_Y` 168) and stored positions are unchanged.
- `src/styles.css`.
- `src/runView.ts`: the run's current position is given as the focus, and the canvas is disposed with the view.
- `checks/docsteps.ts`: the report-icon geometry, the Fit button's title, the policy chip, and opening settings before the policy switch.

## Cards

- **Content**: icon and type label on top, the name (16.5 px, semi-bold), then the description (13 px, at most two lines) and a foot row.
  - In the editor the foot holds the repository chips. Editable bindings are amber-tinted, read-only ones neutral, unresolved ones red and dashed; the meaning of the old dark/amber badges is unchanged.
  - In the run view the foot holds the state pill (glyph + words). Repository chips are omitted there because they are definition detail.
- **Colors** are restrained. Each type has a light fill with a quiet 1 px border, and a node's own appearance color replaces the fill and border only. The heavy colored header band of p7 is gone, and the delegate card is light violet instead of a dark fill. The type is still readable without color: icon plus the uppercase label. A state is readable without color too: glyph plus words in the pill, the ring, and the accessible label.
- **Report icon**: a 26 px rounded square with the document icon, placed *inside* the card's lower-right corner (10 px from the right, 9 px from the bottom). A card that has a report reserves that corner in its foot. The icon remains a separate, selectable step element with its state glyph and "Writing report" label, and it follows its card while the card is dragged and when the card's height changes (the existing card observer).
- **Final report**: a readable documentation endpoint. It is dashed with no type color, has a "Document" label, "Final report", `report.md`, and its state in a run. It is still a generated step, not an authored node type. The plan talk keeps the normal card size, with a dashed border.

## Framing rule

The initial view, set once the canvas has a real size:

1. **The whole graph fits at a readable scale** (≥ 0.72 in Compact, ≥ 0.95 in Mini). The view shows it all, centred, at the largest scale up to **1.0**. The old fit allowed 1.2, which made a three-card workflow huge.
2. **It does not.** The view opens at the readable scale. In the editor it shows the graph's start: the left edge, with the leftmost column centred vertically when the height does not fit. In the run view it centres on the run's **current position**:
   - what runs or waits;
   - otherwise what failed;
   - otherwise the final report of a completed run;
   - otherwise what is ready.

   The view is clamped to the graph's edges.
3. **A note at the top** of the canvas then says "*n* of *m* cards fully in view" and offers **Fit to see all**. The note follows every pan and zoom, and it disappears once every card is in view.

**Fit** (the button) shows the whole graph at up to 1.0, with a minimum of 0.25. The zoom controls now show the scale ("72%"). Auto-arrange is explicit, as before, and changes layout only. The run view never writes positions: missing positions are computed in memory (`positions()`), so showing a run cannot persist an arrangement.

The threshold is a judgement. At 0.72 a card name is about 12 px, and the two-line description about 9.5 px. Names are what the person scans, while descriptions are read at 1.0 or in the inspector.

Measured on the state fixture at 1440 × 1000:

| Graph | Initial view |
| --- | --- |
| `tidy` (3 cards) | Everything, 100 % (was 120 %) |
| `improve` definition (plan + 5 + final) | 72 %, from the start; "4 of 7 cards fully in view" |
| `improve` completed run (8 cards, r2) | 72 %, centred on the final report; "4 of 8" |
| `improve` review run | 72 %, centred on Human check (waiting); "4 of 7" |
| `pipeline` (16 + plan + final) | 72 %, from the start, three lanes visible; "15 of 18" |

The concept's two-row layout is not reproduced: positions stay as stored, for any DAG.

## Container size changes

The canvas observes its own size. **The first real size sets the initial view.** After that, a change of size never resets the zoom or pan: opening or closing the inspector, or resizing the window, leaves the view where it is, and only the note's count is recomputed. A card that the opening inspector would cover is revealed by the editor (`canvas.reveal`), which pans just enough.

Probed on `tidy` at 1280 × 800: zoom out to 83 %, select a card (the inspector opens), close it. The transform stayed `translate(237.8px, 198.3px) scale(0.833)` throughout.

Edge endpoints use each card's measured height, so they stay attached after a size change. Report icons and step edges are re-placed by the card observer. `Canvas.dispose()` disconnects both observers, the card one and the new container one, and both views call it when they are disposed. The workflow view also removes its document listener for the Add node menu.

## Checks

- `checks/docsteps.ts` passes in full. It covers:
  - the icon inside the corner, and after a drag;
  - Fit includes the plan card, the icons and the final report, also after zooming;
  - step selection;
  - the policy switch, done from Workflow settings now;
  - the run view's doc states and endpoint;
  - no console errors.
- `npm run check` passes.

The step 2 approval panel had a second element with `.approval-pill[data-kind]`, which the check found ambiguous. The panel's state labels are now `.state-tag`, so the title band pills are the only `.approval-pill` elements.
