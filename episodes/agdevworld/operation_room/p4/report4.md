# Step 4 — relay gap check and final verification

The consumed payload has one correctness gap and no missing endpoint:
`session_tree` stored the grandparent for nodes deeper than one hop. Fixed
`parent` to the topic currently being expanded and removed the redundant
ancestor from the traversal queue. A three-hop regression using both link
kinds failed against the old code and passes after the correction, including
an unread final topic remaining unknown. No topic-name inference was added.

| Part needs | Existing payload | Gap result |
|---|---|---|
| Routine names / current verdict / standing request / elapsed time / ack age / schedule | `/routines`: routines, settings, schedule, health | Present; no addition |
| Up to three sessions / manual-activity explanation / fire identity | `/routines/<name>`: sessions | Present; no addition |
| Graph node identity / depth / immediate parent / link evidence | sessions[].nodes | Corrected immediate parent; no separate edge endpoint |
| State / evidence / unread-topic distinction | nodes[].state, rows, known; detail.health | Present; graph masks stale observations as unknown; missing provenance cannot claim no owed row |
| Chat history / write availability | detail.chat_log, detail.chat | Present; existing chatPanel is the assembly part |
| Latest host observation | `/inflight/<name>` | Present; kept separate from historical session selection |
| Combined pane load | Detail already includes sessions and history | No combined endpoint needed; no speculative API |

Validation:

- Entire relay suite: **92 passed**. The new regression failed before the fix.
- Frontend TypeScript/Vite build passed; Docker production-style build and
  local web update passed (existing bundle-size advisory only).
- Restarted the existing relay to apply the change. It returned to event-queue
  `live`. The real outlier still has **28 nodes and 164 posts**; exactly four
  parent identities changed, and every depth-2 node names a depth-1 parent.
- Updated served frontend: CDP found **29 cards, 28 edges, nine unknown cards**,
  and a **3816 CSS-pixel** scroll surface. The final bottom card and evidence
  remain reachable. Screenshots inspected after relay correction.
- Shell resource trace showed exactly `/routines`, `/routines/<selected>` and
  `/inflight/<selected>` for initial loading. Browser-only simulated read
  failure displayed `Unknown`; restoring fetch and refreshing recovered the
  164-post history count. No service outage was induced for that check.
- Four-pane bounding rectangles preserve the requested left/top/center/right
  relationship. Existing ops → routines → nodes cycle was screenshot-checked
  on the rebuilt web service. No chat was sent.
- `git diff --check` passed. Live payloads, host facts and screenshots remain
  ignored; no credentials or local deployment identifiers were added to docs.

Final local evidence in agdevworld: `.local/p4/final-graph/`,
`.local/p4/final-shell/`, `.local/p4/existing-views/`, and the before/after
payload captures. No pj-clusterintent source change was necessary; nctl supplied
preflight status. The relay's existing 40-node/four-hop limits remain: p4 proves
rendering of returned trees, not exhaustive history or a new unbounded crawler.
