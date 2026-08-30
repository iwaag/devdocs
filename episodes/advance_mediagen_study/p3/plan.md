# advance_mediagen_study p3 — Plan

Braindump: `braindump.md`. p1's survey ranked skeletal rigging **lowest** of
four methods, on the grounds that "no scriptable/API path to rigging exists
on this host". That is a claim about this host, not about the method, and
it was never tested. This phase tests it: one fire of `routine-mediagen`
naming the subject, and the proof is a rig that actually moves in Phaser or
Godot.

Same shape as p2's routine cycle; nothing here changes the framework.

## Hints for the run (advice, not shackle)

- The p1 survey (`spriteSheetFrames/summary.md`) already holds the public
  record: AI enters the skeletal pipeline **upstream** of the rig (stills or
  part images), the rig itself is manual or done in tools we do not have.
  Do not redo that survey; the open question is narrower — *can the rig step
  be scripted here, and does AI-generated part art survive it?*
- **The rig step is text.** A Godot `Skeleton2D` scene is a `.tscn` file;
  DragonBones and Spine export are JSON. A run can write them without an
  editor. Godot runs headless (`godot --headless`) and can render an
  animation to frames with the movie writer; Phaser runs in a browser and
  `agdevworld` already captures it with a one-line Playwright screenshot
  (`pj-agdev/.local/devenv.md`). Phaser 4.2.1 is already in `agdevworld`;
  Phaser has no built-in bones — its skeletal path is the Spine plugin,
  which needs Spine export. **Godot is the cheaper proof; Phaser is the
  one the developer's own project uses.** Either is fine; say which and why.
- Godot is **not installed** on the Developer's Mac or the GPU node. That is
  a host install → `waiting_external` per the standing text, and this time
  the Developer will perform it (`brew install --cask godot` is enough).
  Plan the mission so the install request arrives before the parts are
  generated, not after.
- **Parts generation is the AI half.** Candidates on the box:
  `flux1-kontext-dev` for "same character, isolated part" edits of one
  p1-style still; or generate the whole sprite and cut parts by colour/
  connected components with Pillow (no segmentation weights are present —
  `rembg`/BiRefNet would be a download). Judge parts against p1's asset
  requirement, then judge the rendered animation with p1's
  `consistency_instrument.py` — this is the first time it meets frames that
  are *deterministically* consistent, which is itself a check on the
  instrument.
- Same subject and requirement as p1 where possible (64×64 side-view
  four-legged walk, ≤32 colours, keyable background), so the two methods
  are comparable cell to cell. The comparison against `sheet_locked8.png` is
  the finding.
- New subject → new `gentest-<subject>/`, new `main/subjects/<subject>/`
  row + summary + tips, per the p2 layout. `main/` gets no engine project
  files; the Godot/Phaser project lives in the test repository.

## Steps

1. Fire `routine-mediagen` naming the subject: *skeletal animation of an
   AI-generated pixel-art character, proven by a rig that moves in Godot or
   Phaser.* Supervise; perform the host install when the handoff arrives and
   re-fire. (`report1.md`: workplan, handoff as received, install done,
   resume.)
2. `report.md`: did the rig step script; the rendered frames beside p1's
   sheet; what the instrument said; whether p1's "lowest" ranking stands;
   costs; Deus Ex Machina lines.

**MUST NOT**: host facts or internal repository names into `main/` or this
repository; push `publish/`.
