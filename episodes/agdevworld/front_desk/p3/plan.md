# Front Desk p3 — close a conversation and its related work

## Goal and approach

Add a Front Desk completion button that previews and then resolves the current Front conversation and its related work topics, archives dedicated work channels when applicable, and closes eligible Plane Works. The human's click is acceptance of the result; discovering relationships and applying state changes are deterministic and require no agent run.

This is a private experimental environment and a breaking-change phase. Backward compatibility, legacy-data migration, authentication redesign, and a general workflow engine are unnecessary. Refactor existing code and replace formats when useful. Steps specify outcomes; module boundaries, API names, UI details, and retry storage are implementation choices.

## step1 — Discover the exact completion targets

- Start from the selected `front-desk-*` conversation. Follow `[rootchat]` and `[served]` recursively, retaining the source message and author for each relationship. Reuse the relay's graph machinery where practical; normalize Zulip's `✔ ` rename and deduplicate cycles.
- Supplement cached evidence with targeted Zulip reads, including resolved intermediate topics and their descendants. The display graph is bounded and is not a complete action list. Fetch additional pages as needed; report remaining gaps instead of treating missing history as no related work.
- Identify Plane Works from agent-authored `[work]` notes and existing external identifiers. Identify dedicated work channels from their actual contents and mission bindings. A shared project folder or a similar topic name alone is insufficient evidence.
- Return concrete targets, current states, relationship evidence, already-completed items, and exclusions with reasons. Shared or ambiguous conversations need explicit handling in the preview; do not silently close another conversation's work.

Verify: Front → Autolab plan → task → Forge delegation; resolved intermediate topics; cycles; shared topics; incomplete history. Use captured examples or fixtures without requiring paid runs.

## step2 — Plan and execute the state changes

- Add completion preview and execution operations to the existing agentroom relay. Suggested routes: `GET /frontdesk/<id>/close-plan` and `POST /frontdesk/<id>/close`. Execution uses the server's reviewed target set rather than arbitrary browser-supplied destinations.
- Reuse the existing Autolab mission-completion rule: at least one non-cancelled child exists and all such children are completed. Already-Done Works are successful no-ops; retain Cancelled states. Show unfinished children or standalone Works as blocked rather than manufacturing completion. A separate force-completion feature is outside this initial implementation.
- Resolve eligible related topics and archive dedicated work channels once their contents are accounted for and finished. Retain the shared Front, project, and agent channels. This operation closes work; it does not stop a running agent.
- Re-read relevant posts, bindings, and Plane states before execution. If the proposed changes have materially changed, refresh the preview. Surface known ongoing work; a quiet ops row alone is not proof of completion.
- Apply Plane changes first, then child topics, dedicated work channels, and finally the Front topic. Keep Front open if requested related actions remain blocked or failed. Return per-target outcomes and allow retrying unfinished actions; use a small operation record if useful. No distributed transaction or automatic rollback is needed.
- Use the existing Developer write credential for Zulip and configurable Plane credentials with access to the selected projects. Report unavailable credentials and permission errors at preview time where possible; keep configuration in ignored local files.

Verify: already-closed targets, unfinished/cancelled children, shared channels, state changes after preview, partial API failure, and retry without repeating completed work.

## step3 — Add the Front Desk completion flow

- Add a completion button for the current conversation. Show topic names, channels to archive, Plane labels and states, and any blockers in a concise preview before execution. The final click approves the concrete changes.
- Show progress and per-target results. Keep failed or blocked items visible with a refresh/retry action; distinguish partial completion from success.
- Refresh the conversation list and resolved state after completion while keeping history readable. Check how the existing post-to-resolved behavior interacts with archived children; reopening the Front conversation must not imply that archived work has also reopened.
- Preserve draft text, Japanese IME behavior, conversation switching, and usable desktop/narrow layouts. Visual treatment and component structure are discretionary.

Verify in the browser with demo/fixture data: preview, exclusions, success, partial failure, retry, resolved-history reopening, and changing the selected conversation during a request.

## step4 — Integrate, verify live, and report

- Run `npm run build` and focused relay/domain tests for discovery and execution. Extend existing tests where possible; avoid broad suites that merely duplicate implementation details.
- Check resident-service state through `pj-clusterintent/nctl` or Nautobot, then deploy the frontend and restart changed services using the local environment instructions.
- Exercise one completed delegated conversation in the real UI. Check the preview against Zulip and Plane, execute completion, and read back topic resolution, channel archival where applicable, and Work states. Verify a second attempt and page reload. Prefer an existing completed example; creating a paid delegation is unnecessary unless it tests a missing path.
- Record results and remaining issues in `report.md`. Update setup/runtime documentation and ignored local notes as needed. Commit and push affected repositories and parent submodule pointers where applicable.

## Useful findings and implementation hints

- `agdevworld/agentroom/src/agentroom/routines.py`: `children_of` and `session_tree` already walk both link types. The current graph may contain `note-only` nodes and depth/node truncation; reuse the relationship logic, not its completeness assumptions. The ops engine does not currently retain `[work]` as a dedicated binding.
- `agdevworld/agentroom/src/agentroom/frontdesk.py` owns conversation reads/posts and has an older-conversation read fallback. `server.py` owns HTTP routes. Frontend entry points are `src/frontDesk.ts`, `src/frontDeskState.ts`, and `src/scenes/FrontDeskScene.ts`.
- Autolab's `anchor.py` uses `[work] <issue-id>`; Forge's uses `[work] <project-id>/<issue-id>`. Both select their own bot's binding. Autolab `mission.py` identifies the parent with `external_source` and `work_key(channel, topic)` as `external_id`; child `parent` IDs connect the hierarchy. Reuse these contracts rather than parsing visible reply prose.
- `agautolab/src/agautolab/mission_done.py` already checks parent eligibility. Extract/reuse the small rule as appropriate rather than invoking its untargeted whole-board CLI from the button. `sub_works` excludes cancelled children; the existing Plane adapter filters parents client-side because the deployed API ignored the parent filter.
- Autolab's `zulip_listener.py::archive_work_channel` currently runs only on mission cancellation. `pyagag/src/agag/zulip.py` supplies resolve/archive operations and reads across resolve renames; `agag.plane` supplies the Plane adapter. Avoid coupling the relay to an entire agent listener merely to reuse a helper.
- Planning-time checks found that a Plane credential could read one project but received HTTP 403 for another. Confirm project access before choosing live verification targets; one successful read does not establish universal coverage. Some delegated Works were already completed, so the flow must also support Zulip-only cleanup.
- p2 observed a reused plan topic whose first root note still pointed to an older Front conversation. This is a concrete shared-topic case, not a hypothetical risk. Also, unresolving a topic can generate a Zulip notification that wakes an agent; keep verification centered on completion rather than repeated resolve/unresolve cycles.
- Most implementation belongs in agdevworld, with small shared/Autolab changes if they simplify reuse. pj-clusterintent changes are needed only for actual deployment/configuration needs. Keep local addresses, credentials, and machine-specific evidence in ignored files.
