# Step 1 — signal mapping

The concept's four spatial relationships are retained. Its invented workflow
semantics are not. Inspected the braindump, concept image, frontend types,
relay implementation and live routine payload. `nctl status --json` passed:
Nautobot authentication/catalog/GraphQL and worker checks succeeded.

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

The prototype must also show relay health and the observation timestamp.
Historical session nodes carry current reconstructed topic states, not a saved
verdict at fire time. A manual-activity session is not one identified dispatcher
run. These limits belong on screen.

Step 2 will compare both renderers on the captured outlier before selecting
one. Inspection identified a possible deep-node parent error for the step 4
gap check; no state rule will be duplicated in the browser.
