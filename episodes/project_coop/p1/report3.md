# project_coop p1 — Step 3 report: protoprey's integration path

Date: 2026-10-26. protoprey `9a8e8ff`, `91821dc` (pushed).

## What protoprey now has

- **Cooperation entry** `docs/cooperation.md`, linked from the agent guide and the README: who offers what (playprey,
  gimmickimg, forge) and where their entries are; what a request needs (which result, where it plays, this game's
  constraints, a published delivery); how to adopt, obtain, replace and remove; how to wire a delivery into data; the
  renderer decision; the checks.
- **Adoption record** `data/deliveries.json` (tracked, empty until the trial adopts): per name the producer, identity
  (scene/profile, revision, asset, config and source-image hashes), the sha256 of the delivery's `delivery.json`,
  producer commit and tree state, source reference and WFE output id/digest, install folder, who adopted it and why,
  and the deliveries it replaced.
- **`tools/deliveries.py`** `adopt | install | status | pack | remove`. Bytes come from an explicit package
  (`--from`), a remapped share root (`PROTOPREY_SHARE_ROOTS=agshare=<path>`) or `wfe share resolve`, and are accepted
  only when the manifest hash and every file hash match. They are cached in `.local/deliveries/` and installed under
  `external/` (both ignored). gimmickimg deliveries are installed by copying the player folder; playprey deliveries
  by running the delivery's own `delivery.py embed`, whose `project.godot` settings are committed with the adoption.
  A replacement keeps the previous identity in `replaced`, and a playprey re-embed restores its settings first.
- **Game integration** (`scripts/deliveries.gd`, `main.gd`, `event_player.gd`, `forest_flow.gd`): a location's
  `gimmick` replaces its still on discovery and revisit screens, dimmed by an authored `dim`; a predation step's
  `interactive` adds a choice that hides the game, plays the delivered scene with authored properties (`set`) and
  returns to the same step. Protoprey's key handler ignores keys while a scene plays: without that, the scene's first
  Esc (releasing the mouse) also sent the game to the menu. A missing or uninstalled delivery keeps the still and
  plain choices and names the delivery in the note line.
- **Authored tuning stays in protoprey's data** (`location.json`, `predation.json`); producer bytes live in
  `external/` and are replaced only by an explicit adoption.

## Renderer decision

protoprey renders with Forward Plus. Evidence: playprey's runtime under gl_compatibility loses its fog shaders and
changes lighting (Step 1 probe); protoprey's screens under Forward Plus pass every earlier test, start with no errors,
and the location and predation screens rendered normally in windowed shots (the menu and biome select were not shot). gimmickimg's player works under both. Consequence: the game needs a
Vulkan/Metal/D3D12-capable GPU; the old compatibility renderer's reach (OpenGL-only machines, web export) is given up
for now.

## Rehearsal (scratch copy, not committed)

The Step 2 rehearsal deliveries (gimmickimg `mushroom_waterfall` r1, playprey `wolf` r1, both built from working trees)
were adopted into a copy of protoprey, and the data wired: Mushroom Waterfall's `gimmick`, the wolf `mouth` step's
`interactive` ("Look around inside the mouth", `hud: false`).

| Check | Result |
| --- | --- |
| `adopt` ×2 | verified, installed; wolf embed merged 171 settings |
| `tests/cooperation_play.gd` headless | `failed=false` |
| same, windowed with screenshots | the waterfall flows under the discovery text (dimmed); the mouth step offers choice 2; choosing it plays the wolf mouth full-window; Esc with a free mouse returns to the same step, not to the menu |
| full suite in the copy | all exit 0 after one assertion change (below) |
| `external/` removed, settings kept | starts, logs three "Failed to instantiate an autoload" errors, plays with stills; the cooperation test passes its fallback branch |

`tests/location_play.gd` asserted a visible still on location screens; it now accepts the animated scene in its place.
`tests/media_play.gd` sometimes prints "1 resources still in use at exit" at headless exit; identical setups gave 0, 1
and 0 (also on gl_compatibility without any delivery), so it is intermittent and unrelated.

## Checks on the committed state

Workspace (no deliveries adopted): headless start with no errors; all eight Godot tests exit 0 (the cooperation test
has nothing wired yet, so it is vacuous here); `python3 -m unittest tests/test_deliveries.py` 5/5 (adoption through a
remapped root, tampered bytes refused with the record unchanged, a fresh copy installed from an explicit package after
the original storage was deleted, a package holding a different delivery refused, replacement history and refusal to
re-adopt the same bytes).

## Left for the trial

The committed game has no adopted delivery. The adoption, the data wiring, the published deliveries and any
correction through delegation revision are the trial's work (Step 5), done by protoprey's own run from these entries.
