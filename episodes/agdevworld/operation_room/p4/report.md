# Operation room p4 — parts ready for assembly

All four planned steps are complete. The result is a signal mapping, a working
SVG/DOM conversation graph, a separate-page four-pane skeleton, and one relay
parent-link correction. This phase deliberately stops before dashboard assembly.

## Signal mapping and explicit drops

| Concept element | Real signal / endpoint | Decision |
|---|---|---|
| Left categories | `/routines`: name, request, last_fire, answer, state | Rename to routines; use actual names and standing requests |
| Three active sessions | `/routines/<name>`: sessions, capped at three | Rename to recent sessions; this is history, not concurrency capacity |
| Session identifiers | fire.message_id / fire.at; session.note for manual activity | Show real fire identity; no fabricated S-001 IDs |
| Status chips | routine.state; nodes[].state and rows[].provenance | Show awaiting / acked / stalled / done / unknown; retain quiet only for swept topics with no owed row |
| Running indicator | `/inflight/<name>`: known, in_flight, reason | Show separately; missing observation is unknown, never idle; do not attribute latest-run observations to historical fires |
| Ack waiting too long | answer.state, age_seconds, settings.stalled_seconds | Annotate “Ack overdue” while preserving relay state; not a new state verdict |
| Progress bars and percentages | None | Drop |
| Elapsed execution timer | answer.age_seconds or fire.at | Rename to time since fire; never claim harness execution duration |
| Current stage / next step / intake-plan-review-finish pipeline | None | Drop stages and prediction |
| Process graph | sessions[].nodes: parent, via, link_id, state, known, rows | Rename to conversation flow; edges reflect reconstructed links, not execution order |
| Per-node evidence | rows[].provenance, known, resolved, last_post, via/link_id | Show provenance line plus full inspectable details; note-only stays unknown |
| Right chat history | detail.chat_log and routine.fire_topic | Show routine fire conversation; child-topic chat is not exposed here |
| Send | Existing chatPanel and POST /chat | Reuse in assembly; no new write path or live test post in p4 |
| Next scheduled event | routine.schedule.next, overdue; board.schedule health | Available as schedule evidence, not “next processing step” |
| New session button / three-run limit / account tier / settings / decorative categories | None needed by prototype | Drop |

## Renderer and stress verdict

Both Phaser and SVG/DOM were sketched against the real 28-conversation,
164-post outlier. SVG/DOM was retained for native scrolling, keyboard focus,
selectable evidence and integration with the surrounding DOM panes. Beyond
15 nodes every returned card stays readable at a fixed size; the surface
scrolls instead of shrinking or hiding nodes. Final served build: 29 cards
including origin, 28 edges, nine visibly unknown nodes, 3816 CSS pixels high;
scroll-to-bottom and evidence disclosure passed. See [step 2](report2.md).

## Layout and retained parts

| Part | Location in agdevworld | What is ready |
|---|---|---|
| Session graph | `src/sessionGraph.ts`, `/?parts=graph` | Real session selection, SVG edges, state/evidence cards, scrolling |
| Frame and loader controller | `src/operationParts.ts`, `/?parts=shell` | Four empty slots, routine/session selectors, loader counts and failure handling |
| Styles | `src/operationParts.css` | Native grid, scroll areas and accessible focus |
| Routine/in-flight reads | `src/routineState.ts` | Existing endpoints reused; headline accepts shared board/detail metadata |
| Chat | Existing `src/chatPanel.ts` | Reuse during assembly; no new write path |
| Relay relationship data | `agentroom/src/agentroom/routines.py` | Immediate parent corrected at every depth |

The DOM frame is a separate URL, keeping all seven existing Phaser views at
`/`. Left routine sidebar and right history persist beside recent sessions
above the graph slot. Narrow screens scroll the desktop skeleton horizontally.
See [step 3](report3.md).

## Verification and delivery

The three-hop parent regression failed before the fix; all **92 relay tests**
passed afterwards. TypeScript/Vite and the Docker web build passed. The
existing relay and local web service were updated and checked against live
read payloads. CDP also verified read-failure recovery, the four-pane positions,
and the preserved ops/routines/nodes cycle. Initial shell loading uses three
existing reads; no new polling or Zulip posts were introduced. See
[step 4](report4.md) for the gap matrix and evidence paths.

Reports are [step 1](report1.md), [step 2](report2.md), [step 3](report3.md),
and [step 4](report4.md), committed per step along with the corresponding
agdevworld changes and pj-agdev submodule pointers. Screenshots, payloads and
throwaway comparison sketches remain in ignored `.local/p4/` in agdevworld.

## Remaining assembly work

- Put the graph and existing chat panel into the shell; bind routine and
  session selection across the finished panes.
- Turn routine/session placeholders into the mapped request, elapsed-time,
  state and schedule presentation, including the overdue-ack annotation.
- Add the latest-only in-flight presentation and its existing allowed poll
  lifecycle; do not attach it to historical session status.
- Refine branch organization, edge routing, full-name presentation and narrow
  screen layout with the real outlier, without fake stages or percentages.

Current topic verdicts are not historical session outcomes. Manual activity
without a dispatcher fire is not one identified scheduled run. The relay
still bounds reconstruction to 40 nodes and four hops; no complete-history
claim is made. Snapshot selection/explicit refresh is the prototype lifecycle;
continuous dashboard refresh and any cap-exhaustion presentation need deliberate
assembly decisions. These are stated boundaries, not unfinished p4 steps.
