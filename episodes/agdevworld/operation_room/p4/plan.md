# operation_room p4 implementation plan — dashboard survey and parts

Per the braindump: p4 does not aim for completion. It confirms the needed
features, surveys what exists, and prepares the parts for the dashboard —
left routine list, session list (max 3) on top, the process-flow graph as the
main center/bottom pane, chat history on the right. Those four layout points
are the only things concept.png is authoritative about; every other element
the image generator added (progress percentages, "next step" predictions,
staged pipelines, intermediate states) is explicitly disposable, and several
of them are on p1's cannot-do list anyway. UI text is English.

Where p4 ends: the mapping is settled, the graph renders real data, the
layout approach is decided and skeletoned. Assembling the four panes into the
finished dashboard is the next phase.

## Starting position (nothing here needs building)

The data side is essentially done — p3 left:

- `GET /routines` — the left pane's content: name, standing request, last
  fire, answered-or-not.
- `GET /routines/<name>` — the top pane's content: the last 3 fires as
  session trees, plus the fire topic as a chat log.
- `GET /inflight/<name>` — the only polled, non-Zulip signal.
- `POST /chat` + `chatPanel.ts` — the right pane, already proven with a real
  round trip.
- Every tree node already carries the /ops state verdict and provenance.

p4 is therefore a frontend episode with one relay-side gap check at the end.

## Step 1: the mapping table

One table in the report: concept element → real signal/endpoint → decision
(show as-is / rename / drop). Settle the vocabulary translation once:

- Progress % → **drop**. No signal in this system can produce a percentage
  (p1 cannot-do). The honest replacements are the state word, elapsed time
  (`age_seconds`), and the in-flight indicator.
- "Next step" → **drop**. There is no deterministic pipeline; the honest
  flow is the conversation tree as it actually grew.
- Session status chips → map onto the real states
  (awaiting / acked / stalled / done / unknown, plus in-flight). Consider a
  distinct treatment for p3's finding that "`acked` for longer than the
  stalled threshold is its own kind of trouble".
- Anything else the image invented → decide, but default to drop.

This step is cheap and prevents the prototype from quietly re-promising
things the system cannot say.

## Step 2: the flow-graph prototype (the risk, so it goes first)

Render a session tree as a node-edge graph: nodes = conversation topics
carrying their /ops state word and provenance line; edges = the
rootchat/served links the relay already reconstructed.

- Phaser is a game engine — nodes and curved edges are home ground for it
  (PanelGridScene precedent). An SVG/DOM renderer is an acceptable
  alternative; pick whichever survives the stress test below. The choice is
  the implementer's, but make it *after* both have been sketched against
  real data, not before.
- **Stress-test with the real outliers, not the demo shape.** concept.png
  shows 7 tidy nodes; the realm's actual worst case is mediagen — 28
  conversations in one session, 164 posts in the fire topic. A layout that
  only works for 7 nodes is not a part, it is a mockup. Decide and record
  what the graph does past ~15 nodes (collapse, scroll, cluster — any is
  fine if chosen deliberately).
- Keep the two standing principles in the prototype: unknown is never
  rendered as idle, and every node shows or can show its provenance.
- Screenshot verification from the start (`.local/opsshot.mjs`; the visual
  pass has caught defects five phases running). Remember the tween trap if
  the graph animates.

## Step 3: the layout shell decision

The current agdevworld is one full-screen view at a time, cycled with `⇄`.
The dashboard wants a sidebar plus three persistent panes — a structural
change, and the biggest design decision of the episode.

- There is precedent for a hybrid: `chatPanel.ts` and `detailPopup.ts` are
  already DOM overlays living beside the Phaser canvas. A plausible shape is
  DOM for the frame and side panes, Phaser (or SVG) only for the graph pane.
  But the split is the implementer's call — decide it here, write the
  decision and its reason in the report, and build only the skeleton
  (four empty panes wired to the existing loaders) to prove it.
- Decide how the dashboard coexists with the existing view cycle: a seventh
  view that takes over the screen, or a separate page. Either is fine;
  breaking the old ops/routines views is also fine (destructive phase) as
  long as the report says what happened to them.

## Step 4: relay gap check

Walk the mapping table and the prototype's needs against the actual
`/routines` / `/routines/<name>` / `/inflight` payloads. Expected result:
few or no gaps — likely candidates are the edge list's exposure format for
the graph, or a combined endpoint to avoid chatty pane loading. Add only
what the prototype actually consumed; nothing speculative.

## Deliverables

- report.md with: the mapping table (including the explicit drop list), the
  graph prototype's stress-test verdict, the layout decision and skeleton,
  and the parts list + remaining work for the assembly phase.
- Working prototype code in the repo (this is a parts-preparation phase, so
  landing parts behind the existing views is fine; clearly-throwaway
  sketches go to `.local/` as usual).

## Constraints (minimal)

1. Everything read-only except the already-built chat door; no new Zulip
   writes, no new Zulip polling (the event queue and the existing in-flight
   poll are the data paths).
2. unknown ≠ idle and provenance-on-screen apply to the prototype too.
3. UI text in English.
4. Do not re-promise dropped concept elements (no fake progress bars, no
   invented pipeline stages).

Everything else — renderer choice, pane proportions, how faithful to
concept.png's styling, graph interaction details — is the implementer's
discretion. concept.png is a mood reference, not a spec; the braindump says
so itself.
