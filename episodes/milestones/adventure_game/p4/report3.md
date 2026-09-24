# p4 step 3 — Independent verification of the delivery

2026-09-24 ~13:55 UTC, by the Omni Agent, in a scratch clone outside every agent workspace.
Nothing was pushed or changed in the game repositories.

## Functional result: pass

Fresh clone of `autodev/protoprey` at the integrated `main` **`3bf3e90`** (5 commits on `6330a3a`:
`2f3bdff` import, `c2ecf6f` scene, `bc3cf35` wiring, `4cb4767`/`3bf3e90` VERIFY).

- `godot --headless --path . --import`: exit 0, no error lines.
- `tests/*.gd` headless, each alone: `biome_select_play`, `forest_play`, `location_play`,
  `loop_play`, `media_play`, `playthrough`, `ui_play` — **all exit 0**, no `FAIL`, no
  `SCRIPT ERROR`. `media_play` prints an ObjectDB-leak warning at exit; its script and
  `scripts/media.gd` are untouched by this mission.
- Assets: `assets/biome_view/{bg,forest_icon,forest_view,meadow_icon,meadow_view}.png` are
  byte-identical (`cmp`) to `protoprey-refs@34ae3f9`; identities in `assets/ASSETS.md`.
- **Windowed pass** (real renderer, 1280×720, a scratch driver script outside the repository
  calling the same entry points the key handlers use), states read after each hop:

  | Hop | phase | discovered | locations |
  |---|---|---|---|
  | F starts → select | pick (select shown) | — | — |
  | spawn forest → explore → found list | predators | wolf | flower_pond |
  | wolf → predation → end | **select shown** | wolf | flower_pond |
  | spawn meadow → found list | predators | wolf | + little_burrow, mound |
  | revisit a meadow location → end | **select shown** | wolf | same |
  | spawn forest → found list | predators | wolf | + mashroom_waterfall, fallen_log_hollow |

  The select appears after predation and after a revisit; discovery state survives every
  respawn; the forest's list shows the wolf and all its locations on return. Wetland is not
  offered (autolab's decision 2, confirmed by the Developer in #11317). Menu E untouched
  (`loop_play` green).
- Screenshot: [select_after_predation.jpg](select_after_predation.jpg) — bg.png full screen,
  title, two bubble cards with view and icon, Back, key hints, the "Wolf discovered" label below
  Back without overlap.

## Creative verdict (the Developer's)

"Favourably, it went straight through to completion in the front room alone, and the
functional check looks fine too." (2026-09-24, to the Omni Agent, translated from Japanese: 「front roomだけで一気に
完了までいき、動作確認でも問題なさそうでした」). No creative feedback was posted as a new cycle.

The Omni Agent's own look, recorded apart and not a verdict: the scene reads as the concept
simplified — the Earth is **static** (the Developer removed rotation from `todo.md` in
`34ae3f9`), and the flat white-chip icons sit plainly against the photographic backdrop.

## Workflow observations → step 4

- The mission is **not yet `done` on record**: Front asked the Developer for acceptance (#11402)
  and waits (trace: `AWAITING_HUMAN`). Closing it is the Developer's act in the Front Desk
  conversation, not a DEM. *(Closed 14:00:28 on the Developer's #11410; see report4.)*
- The rest are in [report2.md](report2.md) (Front as task acceptor, conditional acceptance, one
  redundant serving). Nothing here needed the Omni Agent's hands.
