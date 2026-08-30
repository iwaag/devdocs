# advance_mediagen_study p4 — Plan

Braindump: `braindump.md`. p3 proved the rig step scripts, but the result
was low-level: one still cut at a belly line, one rotation per leg pair,
two of four frames byte-identical. p3 also did **no web research** — the
p3 plan's "do not redo the survey" hint travelled verbatim into the task
and overrode the standing text's Gather step. This phase does the survey
p3 skipped, decides from it whether any idea is worth running, and runs it
if one is. Two fires at most; nothing here changes the framework.

## What p3 left as fact (do not re-test)

- Godot 4.7.2 is on the Developer's Mac; `.tscn` is script-writable;
  `--headless --import` then `--write-movie` (without `--headless`) renders
  frames in ~2 s. That path stays reusable — `gentest-skeletalRig/build_rig.py`.
- The ceiling was **segmentation**, not rigging: no segmentation weights on
  the box, and a global cut of a contact-pose still leaves the legs in the
  body. Any idea that still starts from "cut one still" will hit it again.
- The consistency instrument cannot see duplicate frames (p3) or
  same-palette wrong content (p1).

## Fire 1 — survey, then decide

Fire `routine-mediagen` naming the subject: *2D skeletal animation on the
AI-generation premise — a broad survey, then a go/no-go.* Make the workplan
say, in these words, that **Gather is the deliverable of this fire** and
that `skeletalRig/summary.md`'s "reuse, not restate" is what this fire
replaces. The survey belongs in `main/subjects/skeletalRig/summary.md`
(rewrite it; p3's text stays as the "Verify" record), with URLs, and with
what could not be found.

Directions the survey should at least look at — hints, the run ranks them:

1. **Segmentation into parts.** Interactive/open segmentation (SAM-family,
   `rembg`/BiRefNet), part-aware cutting for characters, and pixel-art
   specific approaches (connected components on a quantised palette worked
   in p3 up to the leg problem). A *passing*-pose or spread-legs source
   still, generated on purpose, may beat a better segmenter for free —
   the p1 runner can generate that today.
2. **Generating parts instead of cutting them.** `flux1-kontext-dev` is on
   the box and untested for "same character, isolated part"; the public
   record on Kontext identity-preserving edits is the question, not the
   model's existence. Also layered/part-wise character generation papers
   and tools (anything that emits a PSD/layer stack or a Spine/DragonBones
   export directly).
3. **Rigging automation.** Auto-rigging for 2D (PolyRig-class tools, the
   Spine2D-vertex paper p1 found, Live2D-style mesh deformation), and
   what their inputs are — most want layered art, which is direction 2's
   output.
4. **Motion from elsewhere.** Video models as *motion source* — Wan 2.2 i2v
   on the p1 still, then per-frame pose extraction — with the rig only
   used for cleanup; or `SDPoseKeypointExtractor`-class nodes (present in
   ComfyUI, weights unknown) to drive bone angles. Also mesh-deform rather
   than bone-rotate: Godot `Polygon2D` with `Skeleton2D` weights, which
   turns "fragments swinging" into "one mesh bending".
5. **Engines besides Godot.** Whether Spine/DragonBones/Phaser have a
   scriptable, headless-renderable path comparable to p3's — p3 left it
   open and Phaser is the developer's own engine.

The fire ends with a **ranked go/no-go**: for each idea, what it needs that
is not on the box (weights, host installs, paid tools), the expected gain
over p3's result against the same requirement, and one line of reasoning.
If nothing clears "worth one task's budget", say so and stop — that is a
complete fire. A host install it wants goes out as `waiting_external` in
the fire's own report, so the Developer can perform it before fire 2.

## Fire 2 — run the best idea, if there is one

Fire again naming the idea the Developer picks from the ranking (or the
run's top pick). Same subject, same requirement as p1/p3 — 64×64
side-view four-legged walk, ≤32 colours, keyable background — so the
result lands beside `sheet_locked8.png` and p3's four frames. Extend
`gentest-skeletalRig/` (its yaml is not `verified`) rather than opening a
new repository unless the idea is a different workflow family. Judge with
the pictures first and the instrument second; if the idea produces more
than four frames, add the duplicate-frame check the instrument lacks — it
is one hash per frame.

## Report

`report1.md` per fire; `report.md`: the ranking as delivered, whether the
survey actually happened this time (URL count is the cheap test), what was
run and how it compares to p1/p3 frame for frame, costs, and Deus Ex
Machina lines.

**MUST NOT**: host facts or internal repository names into `main/` or this
repository — and `main/` must not point at a test repository's `report.md`
either, which p3's summary does; push `publish/`.
