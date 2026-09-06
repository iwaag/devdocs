# Step 3 — bounded reconstruction and readable branches

Each relay session now includes `truncation`: whether an eligible link was
omitted, the node/depth reasons, and the existing 40-node/four-hop limits.
The walker checks held link evidence at the boundary; it performs no additional
Zulip calls. Exactly reaching a cap without omitting a child is not truncation,
and cycles or links outside a fire's window do not falsely exhaust it.
The graph names truncation, and treats missing metadata from an older relay as
unknown completeness. Unread linked topics still carry their own unknown state.

Graph layout allocates a contiguous vertical band to each parent subtree,
keeping siblings together and reducing cross-branch routing. Cards wrap long
names; selecting one exposes the full name and source evidence in the details
heading. Every returned node remains scrollable at a fixed readable size.
Historical topic observations and manual activity retain their distinct copy.
Narrow screens stack the panes and scroll the routine list horizontally.

The first post-deploy screenshot caught startup's partial routine list replacing
a URL selection and an empty chat claiming no history while unknown. Both were
corrected. The live outlier screenshot also caught verbose host evidence
squeezing the graph; it now has a compact count summary and expandable per-topic
evidence. Screenshots are ignored under `.local/p5/step3/` in agdevworld.

All 95 relay tests passed, including new node-cap, depth-cap, exact-boundary,
cycle, fire-window and manual/scheduled metadata cases. TypeScript/Vite passed.
Deployed relay code with `launchctl kickstart`; its event queue returned to live,
and the real 28-conversation outlier reports untruncated metadata. Inspected
desktop and 390-pixel viewport captures. No real chat post was made.
