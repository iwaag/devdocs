# Project Room p1

Source: [braindump.md](braindump.md).

Build a game-style room for following projects and studies from their purpose through plans, runs, and results, and talking directly in the selected plan or run. Operation Room tracks outstanding replies; Arguing Room develops desires; Project Room follows the resulting work.

This is a breaking phase in a private experimental environment. Backward compatibility and migration of disposable state are unnecessary. Choose API shapes, component boundaries, and implementation order freely; replace obsolete code where useful. Reuse existing services and credentials without introducing a security-hardening project. Keep secrets and machine-specific facts in ignored files.

## Step 1 — Build the project read model

- Extend the existing agentroom relay and Zulip mirror with project list/detail reads. Include project/study kind when known, purpose documents, setup conversations, missions, tasks, latest activity, and source links. Older projects with incomplete metadata should still appear.
- Distinguish `goal` / `researchplan-…` documents, `workplan-setup-…` workspace preparation, ordinary `workplan-…` missions, and `workrun-…` tasks. Setup deliberately creates no mission; a research plan alone is not executable work. Show these gaps explicitly.
- Reuse `agentroom/autolab.py` for mission/task records and their current `[doc]` documents. Follow anchor message IDs for identity, parentage, replacement, and current location. Project channel IDs are useful for stable project selection. Do not infer a research-plan-to-mission relationship from similar names; expose recorded links and leave missing relationships unassigned.
- Present recorded work state separately from conversation response state, reusing the ops engine. `started` does not prove a process is running, and a resolved topic does not by itself prove successful work. Show task counts rather than invented percentage progress, with incomplete reads marked.
- Return mirror health and evidence links. Ordinary refreshes should read the existing mirror; unavailable data remains unknown rather than becoming an empty board.

Verify with focused fixtures: setup-only project, study without a mission, multiple plans/runs, renamed and replaced work, missing parents/documents, resolved history, and stale mirror data.

## Step 2 — Add conversation reads and posting

- Expose the selected mission/task's visible history and post into that source conversation as the Developer. Resolve its current location from the anchor before sending; reuse existing text checks, submission handling, and mirror-backed read helpers.
- Plan comments go to the planning conversation; run comments go to the execution conversation. Label the destination and responsible agent. For purpose/research documents without an execution conversation, provide source access and a clear path to Front for discussing how to proceed; do not imply that posting into a document topic dispatches work.
- Add a project-specific posting route: the existing `/chat` accepts routine conversations, so it cannot serve this feature unchanged. Adapt existing room patterns rather than creating another agent entrance or forwarding every run comment through Front.
- Reuse existing handling for completed conversations where appropriate and make any resume behavior explicit. Retain the draft on failure; an uncertain send should be checked before retrying. Loading a room never starts work.

Verify exact destination after rename, reused topic names, duplicate submission, uncertain delivery, and completed/deleted targets. Use mocked posting for these cases.

## Step 3 — Build the Project Room

- Add `?view=project`, navigation from other rooms, and URLs that restore the selected project/plan/run.
- Use a room background with readable board panels: project/study list on the left, purpose/setup and plan/run hierarchy in the center, selected document/results on the right, and conversation/history below. Adapt the arrangement for narrow screens; a dedicated scene is reasonable.
- Show purpose and originating argue links, recorded work state and reply state, task counts, latest report/activity, and explicit repository/report/artifact links found in source posts. Keep resolved/cancelled/replaced history accessible.
- Add a lightweight “updated since last viewed” indicator using browser-local read positions. Keep selection, scroll position, and per-conversation drafts stable during periodic refresh; reuse the dashboard's polling/lifecycle patterns.
- Reuse portraits, settings, conversation history, and IME input where useful. Display original speech in p1 so reading and replying work without character rendering. Extending Front's rendering worker to project conversations is a later option, not a dependency of this phase.

Verify in the browser: project → plan → run navigation, source/result links, direct URL reload, incoming replies, unread indicators, draft retention, Japanese composition, long plans, and desktop/narrow layouts.

## Step 4 — Integrate and soften room backgrounds

- Register `rooms/project/bg.png` as `[rooms.project]` in the settings manifest and support the new room in frontend settings/types. The image already exists; the manifest entry is missing at planning time.
- Reduce background prominence at render time, leaving portraits and panels crisp. A shared tinted overlay at roughly 20–30% opacity is a starting point; choose the color and strength through visual inspection. Apply the same treatment to existing image-backed rooms where useful.
- Keep revision-addressed settings/assets and existing fallback behavior. No image regeneration is needed.

Verify Project Room, Front Desk, and Arguing Room with real backgrounds, including failed image loading and a settings refresh.

## Step 5 — Deploy, verify the workflow, and report

- Inspect current service state through Nautobot or `pj-clusterintent/nctl`; use the ignored environment notes for deployment details. Reuse the current relay, web deployment, and mirror. Update only affected services/settings and dependency pins.
- Run focused relay tests, `npm run build`, and browser checks. Demonstrate that a project with a plan and run can be found, read, commented on, and revisited after a reply; confirm that repeated board reads do not add Zulip history sweeps.
- Use fixtures for failure cases. For a live round trip, use a deliberately selected test conversation and an explicitly authorized post; simulated reads/sends can finish independently. Report any live posting validation still pending rather than treating it as completed.
- Update relevant developer docs and ignored local notes. Write `report.md` with delivered behavior, checks, screenshots/evidence, and remaining gaps. Commit/push changed repositories and necessary submodule pointers.

## Implementation pointers

Paths below are relative to the shared projects directory.

- `pj-agdev/agfront/src/agfront/project.py`: project/study creation, purpose documents, and setup-only behavior. Project kind currently appears in the channel description; allow unknown for legacy channels.
- `pj-agdev/agdevworld/agentroom/src/agentroom/autolab.py`: mission/task note parser, author rules, state vocabulary, current documents, and replacement IDs. `closing.py` already discovers related work; reuse relevant readers without coupling board refresh to completion actions.
- Relay `realm.py`, `ops.py`, `room.py`, `argueroom.py`, and `chat.py`: mirror access, response evidence, room reads, anchor-based posting patterns, and current send restrictions.
- Frontend `roomState.ts`, `frontDesk.ts`, `scenes/FrontDeskScene.ts`, `frontDeskInput.ts`, and `frontDeskSettings.ts`: reusable room pieces. `RoomId` currently covers only `front | argue`; inspect Front-specific assumptions when extracting shared UI. `operationDashboard.ts` provides refresh and selection patterns.
- `pj-agdev/agfront/src/agfront/render.py`: automatic presentation currently targets Front Desk/Argue speech. Reusing dialogue widgets alone does not enable project rendering.
- `agdevworld-settings/manifest.toml` and `rooms/project/bg.png`: content integration. Prefer existing settings sync over bundling another hardcoded background.
