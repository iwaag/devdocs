# Step 1 — register and load Forge

agdevworld-settings `2c21078` (pushed to `main`). No code change in
agdevworld or agfront; no rebuild, no listener/relay restart.

## What was found

- The active revision at the start was `4f3b55f654c8` (front, autolab).
- `origin/main` already carried `206cbde` "forge", which added
  `characters/forge/lore.md` and `characters/forge/face.jpg` (1024×1024
  JPEG) but no `[characters.forge]` table, so the sync had nothing to
  register. The lore is the supplied one: a media-generating robot with
  deep, mysterious phrasing, conflicted about imitating an artist while
  pretending to feel nothing, called ピーちゃん by everyone.
- agforge's live introduction in `#agents` (message 4926, revision
  `2ceb949`) declares `agent: agforge`, `instance: agforge-agstudio1`,
  `bot_id: 13`, prefixes `assetrun-`, `assetplan-`. The fixture manifest
  from p2 (`d1724cda…`) used the same `agents = ["agforge"]`.

## What was done

- Added to `manifest.toml` in the settings repository:
  `[characters.forge]` with name `Forge`, nickname `ピーちゃん`, the two
  existing file paths and `agents = ["agforge"]`. No `[overrides]` needed
  in `.local/settings.toml`; the realm's roster name matches.
- `uv run agentroom-settings sync` from `agdevworld/agentroom`: active
  `4f3b55f6` → `2c21078ae237`, `changed: true`, characters
  `autolab, forge, front`, room `front`. `status` lists
  `character forge  Forge agents=['agforge'] face=characters/forge/face.jpg`.
- Relay `GET /settings` (no kickstart) reports active `2c21078ae237`, the
  manifest entry for `forge` with the full four-line lore, and
  `face = /settings/2c21078ae237…/characters/forge/face.jpg`; that URL
  returns `200 image/jpeg`, 491 482 bytes, the 1024×1024 portrait. (The
  relay answers `501` to `HEAD`; use `GET`.)
- Screen: `http://localhost:8090/?view=frontdesk&conv=20260908-ex1`
  (the `:8090` image built in p2 step 5, untouched) shows
  `settings 2c21078ae237 (main)` in the header; an in-page fetch of the
  relay returns `front/autolab/forge forge=ピーちゃん`. Screenshot
  `agdevworld/.local/shots/p2ex1/s1-desk-fresh.png` (ignored).

## Result

Settings-only addition works end to end: the character is registered,
served with its lore and revision-addressed portrait, and picked up by the
running screen without any application change. agfront reads the same
tree (`active.json` now `2c21078…`), so the next `character_talk` run will
pin this revision. The Forge portrait itself will first be drawn when a
scene names `character: "forge"` (step 3).

Note for the driver: `DESKSHOT_STEPS` splits on commas, so an `eval`
expression containing commas has to go through `evalfile`.
