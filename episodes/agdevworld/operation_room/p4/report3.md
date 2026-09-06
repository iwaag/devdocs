# Step 3 — layout shell

Selected a separate page (`/?parts=shell`) with a DOM/CSS-grid frame. The
existing seven-view Phaser cycle at `/` is unchanged. This keeps the graph's
scroll/focus/evidence behavior native and gives the persistent side panes their
own scroll containers; no Phaser scene lifecycle has to own the frame.

The skeleton has four slots: routine sidebar on the left; latest-three session
selector above the central graph slot; chat-history slot on the right. It wires
`loadRoutines`, `loadRoutine`, and `loadInflight`, showing loaded counts and
explicit error/unknown text. Async generation guards prevent late selections
from overwriting a newer one. It does not assemble the graph or chat panel:
there is a link to the graph part and placeholders for the next phase.
Host observations are clearly latest activity, not the selected historical fire.
Reads occur on selection or explicit refresh; no new interval or write exists.

`npm run build` passed. The 1600 × 1000 screenshot at
agdevworld `.local/p4/shell-shots/initial.png` was inspected: all four slots stay
visible, the center reports 28 linked conversations, and the right reports 164
loaded posts. The reused graph-only probe correctly found no graph viewport
on this intentionally empty shell; the page text and screenshot confirm the
slots loaded, rather than treating that diagnostic as a rendering failure.
Below desktop width the shell scrolls horizontally; mobile composition remains
assembly work. The root view cycle is preserved by branching before scene and
chat initialization, so the prototype starts neither hidden scenes nor chat.
