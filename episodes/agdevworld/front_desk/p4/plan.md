# Front Desk p4 — close requests from any work view

## Goal and scope

Make p3's completion flow available for ordinary Front conversations, individual routine runs, and requests made directly to Autolab or Forge. Keep the existing Front Desk button and share discovery, preview, execution, and result presentation across the views.

This is a private experimental environment and a breaking-change phase. Backward compatibility and legacy-data migration are unnecessary. Refactor or replace existing APIs, schemas, and UI components freely; authentication redesign and a general workflow engine are outside this task. The steps specify outcomes, not mandatory implementation routes.

Completion is the human's acceptance of a selected request and its related work. Retain p3's useful behavior: show concrete changes first, preserve existing Done/Cancelled states, report unfinished Works and partial failures, and support retry. Completion does not stop an agent that is running.

## step1 — Generalize the completion target

- Replace the Front Desk id as the core input with a selected conversation, identified by channel and topic (or equivalent stable identity). Front Desk becomes one caller of the shared operation.
- Support ordinary `front-*` conversations, a routine's individual run topic, Autolab `workplan-*`, and Forge `assetplan-*` requests. Discover descendants, Plane bindings, and dedicated work channels using the existing relationship records.
- Define the boundary around the selected request. A child execution topic should offer navigation to its parent request; selecting it must not silently close its parent and siblings. Show shared or ambiguous relationships with their evidence and the resulting scope.
- Generalize p3's ownership exclusions: they currently recognize only other `front-desk-*` roots. Keep unrelated requests outside the selected request's closure, including ordinary Front and direct-agent requests.
- A routine completion closes that run and its work. Its standing request, future schedule, and previous runs remain independent. Treat a historical shared run topic as a whole conversation unless the records establish a narrower scope; show that scope explicitly.
- Reuse p3's Plane eligibility, dedicated-channel checks, preview freshness check, execution ordering, and retry behavior where useful. Adapt them to the generalized root rather than maintaining separate implementations per view.

Verify with focused fixtures: each supported root type, child-to-parent navigation, nested delegation, another request's ownership, shared topics, resolved intermediates, and missing relationship evidence. Include a routine whose previous-run link must not pull earlier runs into completion.

## step2 — Expose one completion flow in the existing views

- Front Desk: retain `finish ✔`, backed by the shared operation.
- Operation Room: add completion to the selected routine run's detail.
- Agent Room: expose completion from ordinary Front conversations and direct Autolab/Forge requests. Ensure project workplans as well as agent-owned conversations are reachable; the existing per-agent topic list alone does not cover project work.
- Reuse one preview/result panel showing the selected request, topics to resolve, channels to archive, Plane changes, exclusions, and blockers. Keep partial results actionable through refresh/retry.
- Keep Ops's existing “confirmed” action distinguishable from completion: confirmation dismisses a display row; completion changes Zulip/Plane.
- Refresh affected views after completion and retain a way to inspect the result/history. Add search or a resolved-history option only where needed to reach the supported requests; a separate management screen is not required.

Verify in browser fixtures: entry from each view, preview, success, blocked/partial outcome and retry, switching selection during a request, and desktop/narrow layouts. Preserve Front Desk draft and Japanese IME behavior.

## step3 — Integrate, verify live, and report

- Run focused relay tests for scope and completion, plus `npm run build`. Exercise shared-request exclusion, unfinished Plane children, changed previews, and retry after partial failure through the generalized entry point.
- Check resident-service state with `pj-clusterintent/nctl` or Nautobot, then deploy the frontend and restart changed services using the local environment notes.
- Use existing completed requests for live verification: an ordinary Front conversation, an individual routine run, and a direct-agent request where available. Inspect the preview, complete from the relevant GUI, and independently read back Zulip resolution, applicable channel archival, and Plane state. Verify repeat preview and reload. Record any unavailable live case and cover it with a fixture.
- Confirm routine completion leaves its standing request and future schedule intact. New paid agent runs are only useful if they establish behavior the existing examples cannot demonstrate.
- Write `report.md` with results and remaining limitations. Update affected runtime/setup documentation and ignored local notes. Commit and push affected repositories and parent submodule pointers where applicable.

## Useful findings and implementation hints

- `pj-agdev/agdevworld/agentroom/src/agentroom/closing.py`: `discover()` constructs its root with `desk_topic(ident)`; exclusions use `is_desk_topic` and `DESK_PREFIX`. These are concrete Front Desk assumptions to remove. `related_topics`, Work binding lookup, and dedicated-channel discovery are the reusable pieces.
- `agentroom/src/agentroom/close.py`: `Closer.plan()` validates a Front Desk id; API schema, locks, and operation records also use that identity. `plan_actions()` already distinguishes the selected root from related topics. `server.py` currently exposes completion only through `/frontdesk/<id>/close-plan` and `/close`; route names are discretionary.
- `src/frontDeskClosePanel.ts` holds the existing completion panel. `src/operationDashboard.ts` owns routine selection/details; `src/agentRoomState.ts` supplies Agent Room reads. `agentWork()` explicitly excludes project work, so inspect the `/work` grouping when adding direct Autolab entry points.
- `agentroom/src/agentroom/routines.py`: current routine runs have individual `front-routine-<name>-<stamp>` topics (`operation_room` p7). The standing request is `routine-<name>`. Earlier environment notes describe the old shared-topic arrangement; current code is the better guide here. A previous-run reference is context, not ownership.
- The display session graph is bounded and may omit resolved history; p3's completion discovery already supplements it with targeted reads. Reuse that distinction. `[rootchat]`, `[served]`, `[work]`, and Plane external identifiers provide relationship evidence; similar names alone do not establish closure scope.
- P3 encountered a reused plan anchored to another Front conversation, a Plane credential with project-specific HTTP 403, and an archived channel becoming unlistable. Retain those lessons when refactoring; the p3 fixtures and reports are useful examples.
- `agag.plane.reason_not_completed` is the shared completion rule. P3 completes an eligible parent whose non-cancelled children are done; it does not manufacture completion for unfinished or standalone Works. Force completion and agent cancellation are separate features, unnecessary for extending GUI coverage.
- Most work belongs in `pj-agdev/agdevworld`; shared-library or cluster changes are appropriate only when the implementation needs them. Keep machine addresses, credentials, and local evidence in ignored files.
