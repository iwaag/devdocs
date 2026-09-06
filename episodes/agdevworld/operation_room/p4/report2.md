# Step 2 — graph prototype and renderer decision

Implemented `src/sessionGraph.ts` and the selectable `/?parts=graph` workbench
in agdevworld. Nodes are topics, edges use the relay's parent identity, and
node selection exposes complete provenance. Health failures display unknown.
The fire topic is an explicit session origin, without a fabricated historical
state. Missing parents are counted rather than silently connected to a root.

Both renderers were sketched against the same captured real outlier before
choosing: 28 linked conversations, 164 fire-topic posts, depths 1 and 2.
The ignored Phaser sketch uses the installed engine, text, cubic edges and a
scrolling camera. It renders the shape, but text clipping and the custom
camera/input path would need additional work for accessible evidence reading.
The SVG/DOM sketch uses native scrolling and focusable cards, and directly
supports selectable text and a details disclosure. Selected SVG/DOM for the
retained part; no graph dependency was added.

Past 15 nodes: keep every node at 280 × 132 CSS pixels, allocate columns by
relay depth, scroll horizontally and vertically. This deliberately prioritizes
legibility over fitting the whole tree on screen. The real capture rendered
29 cards (28 children plus origin), 28 edges and a 3816-pixel tall surface.
CDP verified scrolling to the bottom and opening node evidence. Screenshot
inspection verified distinct amber unknown cards with unread-topic evidence,
readable quiet cards and no progress/pipeline claims. Long topic names are
ellipsized, with full names on hover and in selectable evidence.

Evidence stays ignored in agdevworld: `.local/p4/shots/phaser.png`,
`.local/p4/svg-shots/{initial,checked}.png`, the captured payload and the
throwaway Phaser sketch. Used the existing `.local/opsshot.mjs` real-time CDP
path, without animation or virtual-time waits. `npm run build` passed; the
existing Phaser bundle-size advisory remains.

The comparison exposed the expected relay parent defect: depth-2 nodes name
an ancestor rather than their immediate parent. The frontend faithfully draws
that payload for now; step 4 will fix and re-verify the relay, rather than
infer topic relationships in a second place. Dense edge routing and optional
branch organization can be polished during assembly; all nodes remain usable.
