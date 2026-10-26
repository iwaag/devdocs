# project_coop p1 — Step 2 report: producers usable by a consumer

Date: 2026-10-26.

## playprey (`63bf63c4`, pushed)

Entry for other projects: `README.md` "For other projects" (what it offers, how to ask, what arrives, limits),
linking `knowledge/delivery.md`. The agent guide points to both and states how a request from another project is
served.

Delivery contract (`playprey_delivery` v2, `tools/delivery.py`):

- **Defect found and fixed.** Deliveries left out the profile's generated tissue GLB
  (`settings.tissue_asset.glb`); the profile check does not list it. A wolf delivery built from published main
  stopped with "tissue asset … missing" (exit 3). The same applies to cheetah and horse profiles. Earlier published
  deliveries (eagle_beakfit, anaconda) have no generated tissue and were not affected. `build` now includes it and
  refuses when it is absent.
- **Dotfiles reconciled.** The code already delivered dotfiles; the document said both that it did and that
  `game/` carried none. The document now matches the code and WFE's share contract (dotfiles carried since
  workflow_editor p11).
- **Embedding contract added**, in `delivery.json` `embed` and the document's "Embedding" section: entry
  `scenes/embed.tscn`, properties `model` and `hud`, signal `finished("exit"|"error")`, controls, ownership of camera,
  mouse and keyboard, required settings, renderer statement.
- **`embed` / `unembed`** ship inside every delivery. `embed` verifies the delivery, copies the runtime into one folder
  of the host project with `res://` paths rewritten (Godot's cache path excepted; Python/shell/Node tools left out),
  merges the `Harness` autoload, 164 global shader parameters, `[physics]` (Jolt) and `playprey/embedded=true` into
  the host's `project.godot`, and writes `embed.json` (delivery identity, written files by sha256, previous setting
  values). Re-embedding into the same folder or `unembed` restores the previous values first. A different host
  renderer is reported, not changed.
- **Embedded mode** in the runtime: `Harness` reads no host command line, runs no 600 s watchdog (it would have quit
  the host game), and reports failures through a signal instead of `quit(3)` (three sites in `moc_lab.gd`).

Checks (rehearsal deliveries in the scratchpad, not published):

| Check | Result |
| --- | --- |
| wolf delivery built from the working tree, verified | 1550 files, 86.6 MiB, verified |
| run as its own project (copied, imported twice, headed shot) | exit 0, mouth renders; one pre-existing TOOTH band warning (`side_r`) as in the producer |
| embedded into a protoprey clone (gl_compatibility) | refused nothing, reported `RENDERER differs` |
| same clone switched to Forward Plus, scratch host script | the mouth played in the protoprey project; Esc with a free mouse → `finished("exit")`; after freeing, the root held only the autoload |
| protoprey's seven tests with the runtime embedded | all exit 0; `media_play` printed one "resources still in use at exit" line (to recheck against a baseline in Step 3) |
| `unembed` | `project.godot` returned to its previous bytes (only the deliberate renderer edit remained) |

Not checked: Linux, the browser editor in an embedded scene, long sessions.

## gimmickimg (`2e0a52f`, pushed)

Entry for other projects: a root `README.md` (the repository had none) with "For other projects", linking
`docs/delivery.md`; a project guide `agent/guide.md`, declared in `project.yaml`.

- **Player** `player/gimmick/gimmick_scene.gd`: a relocatable `Control` that plays a `gimmick_scene` v1 description
  (layers `image`, `video` with a separate alpha image, `frames`), fits like a TextureRect (`cover`, `contain`,
  `stretch`), inherits `modulate`, ignores the mouse, and lists missing files in `problems` instead of failing. No
  autoload, project setting or global class. `player/` is also a standalone demo project.
- **Recipe** `scenes/mushroom_waterfall/recipe.json`: run-013's first=last H3 waterfall (arm a, seed 1) with its
  feathered alpha, run-008's Wan mushroom frames (16 fps, ping-pong) over the mushroom-free plate, with provenance and
  the limits the runs recorded.
- **Delivery** `tools/delivery.py build|verify`, format `gimmickimg_delivery` v1: scene identity and revision, source
  image sha256, producer commit and tree state, provenance, limits, every file by sha256, usage. The waterfall delivery
  is 58 files, 5.8 MiB.

Two player defects were found by comparing with the experiment's own capture and fixed before the commit: the video
control grew to the clip's native size when `expand` was set after `size`, and the alpha shader multiplied the
texture twice (mean brightness of the water region 45 vs 92). After the fixes the region matches the capture
(92/93/92 vs 92/93/92) in both renderers, and the demo reports `ok=true` with no errors under gl_compatibility and
Forward Plus.

## Delivery vs embedding

Both producers now keep the two contracts apart. The delivery (identity, provenance, hashes; transported as a WFE run
output, whose `.wfe-output.json` covers the same bytes) is separate from the embedding: playprey's entry scene,
signal and settings merge; gimmickimg's relocatable player. Neither producer contains protoprey-specific code.

## Deferred to later steps

- G9 (gimmickimg's ComfyUI default host name) and the environment context: Step 4.
- No delivery has been published yet: the trial (Step 5) commissions them through delegation, so the published bytes
  come from in-system runs.
