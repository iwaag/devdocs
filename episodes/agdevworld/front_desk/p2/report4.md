# Step 4 — scene playback and portrait history

agdevworld `1f8b785`: `src/scenes/FrontDeskScene.ts` rewritten,
`src/frontDeskPlayback.ts` and `src/frontDeskSettings.ts` new,
`src/frontDeskState.ts` demo extended (`&demorev=<sha>`), `README_DEV.md`.
Screenshots in `agdevworld/.local/shots/p2/` (ignored), taken with the
existing `.local/deskshot.mjs` driver on the vite dev server against the
live relay.

## What exists now

- **No fixed image paths.** The background, Front's portrait and every
  other face come from the settings revision the relay serves; the bundled
  `bg.png` / `agfront.jpg` remain only as the fallback for an image that
  cannot be loaded, and a generated common icon stands in for a speaker the
  settings do not know. Textures are keyed by their revision-addressed URL
  and loaded at runtime, so a new revision is a new key and nothing stale
  is drawn.
- **Which revision draws what** (`frontDeskSettings.ts`): the active
  revision is read when the screen opens and on `settings ⟳`, and draws
  plain replies and new conversations. A scene carries the revision it was
  written for; that revision is fetched by id once and kept, so a scene
  being read is never redrawn with faces that came later. A revision the
  relay no longer retains is named in the settings line and drawn with the
  current settings; a failed image is counted there too.
- **Playback** (`frontDeskPlayback.ts`, pure): a Front post is one turn
  (plain reply) or its dialogue's turns; a cursor of reply/turn/page; `step`
  moves through pages, then turns, then replies, both ways; `settle` keeps
  the reply on show when the posts change. Front speaks from the lower
  left, any other character from a portrait and box in the upper left; the
  one not speaking stays dimmed with its last line. Long turns page inside
  their box; the label reads `turn/turns pN/M · reply i/n`.
- **Queued arrivals**: a reply that lands while an earlier one is being
  read queues behind it with a `▶ n new replies waiting` chip; a reader who
  had finished the newest reply is taken to the new one as it lands; a
  send moves the reader to the end of the newest reply so the answer
  follows it.
- **History**: portrait, `name（nickname）` and text per turn; the
  Developer with a user icon, an unknown speaker with the common icon;
  acks as receipt lines; a scene's source citations under its turns and
  the reply's link chips after them; a scene's rows drawn with the
  revision it names (a note when that revision is not retained); a reply
  whose scene was unusable shown as the reply plus `⚠ scene unusable: …`.
  A post with a scene shows its turns, never the readable reply as well.
- **Frame**: sources of the turn on show as `📎 #channel › topic #id`
  chips beside the link chips; the status line says `scene unusable: …`
  for such a reply and is cut to the header's room rather than drawn over
  the speaker's name. Below 720px the frame stacks (collaborator box,
  portrait, Front's box, bar) and the history takes the full width. IME,
  draft, emoji-safe wrap and pagination are the p1 code, untouched.

## Verified in the browser (1600×1000 and 600×900)

| check | result |
|---|---|
| Front/Autolab exchange (demo scene, 4 turns) | Front lower-left, `Autolab（親方）` upper-left with its portrait and the `#work-g-13 › workrun-task1-g-13 #5203` chip; turn 3 back to Front with Autolab dimmed; turn 4 Autolab again |
| long turn / plain reply paging | `p1/2` on the p1 reply with links; ▶ pages, then moves to the next reply |
| unusable block | the readable reply shown with `scene unusable: turn 2: character 'forge' is not in settings revision …` in the status and `⚠` in history |
| third character added through settings | fixture repository + `forge` (lore, face, manifest entry), `agentroom-settings sync` → the live relay served `front, autolab, forge` at `d1724cdacfd0` with no restart; the demo scene claiming that revision drew `Forge（棟梁）` with its face |
| recorded revision after settings moved on | config switched back to GitHub, sync → active `4f3b55f654c8`; the same scene still drew Forge from the retained `d1724cda…` (`scene at d1724cdacfd0` in the settings line), history rows included |
| missing retained revision | `demorev=000…` → `settings 000000000000 — settings revision … is not retained here; drawn with the current settings` |
| unknown character on the active revision | `forge · not in these settings` in amber with the common icon, not another character's face |
| queued arrivals | stepping back to reply 1 while a reply was pending → `▶ 2 new replies waiting`; one click → reply 2 and `▶ 1 new reply waiting` |
| conversation switching | chips in the history panel open `20260908-1600` (7 replies, `reply 7/7`), `✔ 20260908-161951` listed |
| settings refresh | `settings ⟳` re-read the active revision; nothing on show changed (same revision) |
| real conversation on the live relay | `front-desk-20260908-1600`: portraits from the settings, history with `Front（姐さん）`, links, `open in Zulip` |
| narrow layout | 600×900: collaborator box on top, portrait, Front's box, prompt hint shortened, buttons moved under the caption, history full width |

`npm run build` passes.

## Notes

- The relay was kickstarted for this step (35 s sweep, `live` again), so
  `/settings` and `dialogue` are served by the launchd job now; the web
  image on `:8090` is rebuilt in step 5.
- The fixture revision `d1724cda…` (three characters) is retained under
  `.local/settings/revisions/` beside the real ones; harmless, and useful as
  a known three-character revision for later checks.
- Two things left as they are: a dimmed box with no earlier line from its
  speaker is drawn empty (the first turn of a scene by the collaborator);
  the queue chip sits above Front's box and overlaps the background art
  rather than the box.
