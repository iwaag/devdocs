# Project Room p1 — Step 4 report: backgrounds integrated and softened

Plan: [plan.md](plan.md), step 4.

## Delivered

- **Settings manifest**: `agdevworld-settings` commit `47d5d71` adds
  `[rooms.project]` (`rooms/project/bg.png`, the 3168×1344 image that was
  already in the working checkout but untracked). Synced by hand
  (`uv run agentroom-settings sync` in `agdevworld/agentroom`): active
  revision `47d5d713fab1`, rooms `argue, front, project`. The running relay
  reads the active revision per request, so `GET /settings` served it with
  no restart.
- **Frontend**: `src/scenes/roomTint.ts` holds the one tint
  (`0x0d0f14` at **0.32**) and both scenes lay it over their background —
  `ProjectRoomScene` (already tinted in step 3, now on the shared value) and
  `FrontDeskScene`, which serves the Front Desk and the Arguing Room. The
  Project Room asks the manifest for room id `project` and falls back to the
  Front Desk's background for a revision that predates it
  (`FrontDeskSettings.background`). Revision-addressed assets and the
  bundled fallback pair are unchanged; `RoomId` in `roomState.ts` names
  `project` (step 3).
- The strength was chosen by looking: 0.25 (step 3's start) still let the
  Project Room's bright set fight the panels; 0.40 flattened the Arguing
  Room's izakaya; 0.32 keeps each room recognisable with the text and the
  portraits crisp in front of it.

## Checks

Screenshots in `agdevworld/.local/shots/project-p1/step4/` (ignored):

- `project-real-bg` — the Project Room on its own background at
  `47d5d713fab1`, Autolab's portrait, a stalled plan selected.
- `frontdesk-real-bg`, `argue-real-bg` — the two rooms at the same
  revision and tint (`settings 47d5d713fab1 (main)` on their settings line).
- `project-relay-down`, `frontdesk-relay-down` — vite on `:5174` with
  `VITE_AGENTROOM_URL` pointing at a dead port: both rooms draw the bundled
  fallback background and say `settings unknown — the agentroom relay is not
  answering on …`; the Project Room's health line carries the same note.
- Settings refresh: the Project Room re-reads `/settings` on load and the
  Front Desk's `settings ⟳` button does by hand; the revision line moved
  from `1af317b66219` to `47d5d713fab1` after the sync without a rebuild.
- `npm run build` passes.

No image was regenerated. The web image on `:8090` is rebuilt in step 5.
