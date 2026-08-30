# advance_mediagen_study p5 — Plan

Braindump: `braindump.md`. Two things:

- **A. Video model as the animator.** Feed the p1 character still into a
  video model and extract a sprite-frame animation from the clip — the
  route p1 ranked second and every phase since has deferred. The braindump's
  bet: **first+last frame conditioning** (`WanFirstLastFrameToVideo`) beats
  first-frame-only, because start == end makes the clip a loop for free.
- **B. Front's role stays narrow.** p4's finding 2 said the supervisor
  cannot verify the run's report. The Developer's answer: it should not try.
  Front relays autolab's report faithfully to the Omni Agent / human, who
  judge. No Front change this phase; the standing text and the fire must not
  ask Front to verify artefacts.

Experimental, non-public environment; destructive phase, no backward
compatibility. **MUST NOT** lines are the only prohibitions; everything
else is advice the implementer may override with a stated reason.

## Established facts (do not re-test)

- Requirement, unchanged since p1: *64×64 side-view walk cycle of a
  four-legged animal, 4–8 frames, looping, one shared ≤32-colour palette,
  per-frame PNGs + sprite sheet, consistent silhouette, flat keyable
  background.* The asset bar to beat is p1's `sheet_locked8.png` in
  `gentest-spriteSheetFrames/` — still the only result that reads as a walk.
  Its **"does it loop"** question is the open item in
  `main/subjects/spriteSheetFrames/tips.md` this phase directly answers.
- p1's reusable code: `gentest-spriteSheetFrames/{runner.py,
  sequence_runner.py, consistency_instrument.py}` (SwarmUI text2img →
  nearest-neighbour 64×64 → ≤32-colour quantise → sheet). The instrument
  is a secondary signal only; it is blind to duplicate frames and to
  same-palette wrong content. **Look at the pictures first.**
- The p1 source still: subject **dog**, tag-style prompt with a contact-pose
  clause, CFG 4, steps 25, seed 12345 on `pixelArtDiffusionXL_spriteShaper`
  (`matrix.smoke.json`). p4 showed a *spread-leg* pose clause is also
  generable in seconds. Either is a fine start frame.
- Endpoint/port/model literals live in the workspace's ignored
  `devlog/m-4-…/task-1/work.md`. **Copy them verbatim into the fire text**;
  planning paraphrases and drops literals (p9/p1 lesson).

## The live stack (checked 2026-08-30 by the Omni Agent)

Standalone ComfyUI 0.33.1 answers on the GPU node (48 GB card, shared with
agforge's production runs — a 14B clip holds it for minutes).

Present and sufficient for Wan 2.2 FLF2V, **no download needed**:

- `diffusion_models/wan2.2_i2v_{high,low}_noise_14B_fp8_scaled.safetensors`
  (also `smoothMixWan22I2VT2V_i2vHigh` / `smoothMix2vLow`, a community merge)
- `loras/wan2.2_i2v_lightx2v_4steps_lora_v1_{high,low}_noise.safetensors`
  — the 4-step speed LoRAs; use them, a 20-step 14B run costs minutes.
- `text_encoders/umt5_xxl_fp8_e4m3fn_scaled.safetensors`,
  `vae/wan_2.1_vae.safetensors`
- Nodes: `WanFirstLastFrameToVideo` (inputs: positive, negative, vae,
  width/height step 16, `length` step 4 default 81, optional `start_image`,
  `end_image`, optional `clip_vision_*`), `WanImageToVideo`, `SaveVideo`,
  `VHS_VideoCombine`, `LoadVideo`, `FrameInterpolate` (no weights).

Absent: **`clip_vision/` is empty** — irrelevant, the Wan 2.2 14B FLF2V
workflow does not use CLIP-vision (the Wan 2.1 FLF2V one does; the only
built-in template on the box, `wanvideo_FLF2V_720P_example_01`, is that
older one — do not use it). No Wan 2.2 5B ti2v, no RIFE/FILM weights, no
ControlNet.

**Workflow file.** Take the official Wan 2.2 14B FLF2V graph:
<https://comfy.org/workflows/video_wan2_2_14B_flf2v-7016f027bcf1/>
(walkthrough: <https://docs.comfy.org/tutorials/video/wan/wan2_2>, also
<https://www.nextdiffusion.ai/tutorials/wan-22-first-last-frame-video-generation-in-comfyui>
and the HF mirror
<https://huggingface.co/GegenDenTag/comfyui-wan-video-2.2-t2v-first-last-frame-workflow>).
It is UNETLoader×2 + LoraLoader×2 + CLIPLoader(umt5) + VAELoader +
`WanFirstLastFrameToVideo` + two `KSamplerAdvanced` (high-noise steps then
low-noise steps, `ModelSamplingSD3`) + VAEDecode + save. Every model name it
wants matches a file above. Export **API format** and drive it the way
`agforge/src/agforge/comfy_video.py` does (POST `/prompt`, poll `/history`,
GET `/view`; patch nodes by `class_type`, never by id); that module is
reference, not a dependency. Upload start/end images with POST `/upload/image`
and reference them from `LoadImage`.

## Web research first — the survey is a deliverable

The braindump says so explicitly: many people have done this; look before
running. Starting points found in ten minutes (the run should go further,
and cite what it read in `summary.md`):

- <https://huggingface.co/styly-agents/Wan2-2-pixel-animate> — a **Wan 2.2
  I2V LoRA fine-tuned for pixel-art sprite animation**. Not on the box;
  downloading a LoRA is allowed (see host-install path). A candidate
  second axis, not the first run.
- <https://civitai.com/models/2425415/sprite-sheet-generator> — "Sprite
  sheet generator v1.2 First and last", a Wan workflow doing exactly the
  braindump's idea (loop from a reference, N frames, nearest-exact 64×64,
  horizontal sheet). Worth reading its node choices; a Civitai download
  needs a login — if the JSON is not reachable, say so and move on.
- <https://pixie.haus/resources/creating-movement-engineering-pixel-art-animation-ai>
  (already cited by p1's summary) — temporal prompting: describe the
  *motion*, not the character.
- <https://huggingface.co/morphic/Wan2.2-frames-to-video> — multi-keyframe
  conditioning on Wan 2.2; relevant if 2 keyframes are not enough.
- <https://stable-diffusion-art.com/wan-2-2-first-last-frame-video/>

Questions the survey should answer: what resolution people feed Wan for
pixel art (upscale the 64×64 still nearest-neighbour to ≥ 512, or generate
the still large first?), typical `length` for a loop, whether anyone reports
identity drift across an FLF clip, and how frames are picked (uniform
stride vs. motion-based).

## Fire 1 — one clip, then a small matrix

Fire `routine-mediagen` naming the subject: **video-model motion for a
pixel-art walk cycle — `videoFrameExtraction` workflow family**, new subject
and new repository (`gentest-videoFrameExtraction`; every existing yaml is
`verified`). Advice to the run, in rough order:

1. **Gather** as above; rewrite nothing in `spriteSheetFrames/`, it gets a
   tip + a closed "Still open" line at the end.
2. **Measure one generation before any matrix** (standing text already
   says this). Recommended first cell: start = end = the p1 dog still
   upscaled nearest-neighbour to 512×512 (or 640×400 — keep `width`,
   `height` multiples of 16), `length` 33 or 49, 4-step lightx2v LoRAs,
   prompt in the braindump's shape: *"2D pixel art video game sprite, a dog
   running in place, side view, seamless walk cycle loop, flat solid
   background, no camera motion"*. Record wall time, VRAM, and whether the
   card was free (agforge shares it).
3. **Matrix, two axes at most:** conditioning `{first-only (WanImageToVideo),
   first+last same image}` × `length {33, 81}`. Anything else is a later
   fire. `~4–6 clips` is one task; a 20-clip grid is not (WORK_TIMEOUT 1200 s).
4. **Frame extraction → sprite frames.** Decode with ffmpeg or `LoadVideo`;
   pick 8 frames at uniform stride over the clip (for FLF, drop the last
   frame — it duplicates the first); downscale nearest-neighbour to 64×64;
   quantise all frames to one ≤32-colour palette (reuse p1's code path);
   key the background; emit per-frame PNGs + sheet. Add a **per-frame hash
   duplicate check** and a **loop-closure measure** (distance frame N → frame
   0 vs. mean adjacent-frame distance) — the two checks the instrument
   lacks and this subject needs most.
5. Compare with p1's `sheet_locked8.png` frame for frame, pictures first.
   Expected failure modes to look for: pixel-grid drift / blur (Wan does not
   know about the grid; the nearest-neighbour downscale will hide some),
   background not flat, subject scale change, the model "zooming" instead
   of walking, and a first-only clip that never returns to the start pose.
6. **Improve**: `main/subjects/videoFrameExtraction/{summary.md,tips.md}`,
   row in `subjects/INDEX.md`, and a tip in `spriteSheetFrames/tips.md`
   that answers or re-scopes "does it loop". Publication self-check, push
   `main` and the test repository.

If the first clip is already unusable (not a dog, not side view, not
pixel art), one cheap lever before giving up: generate the start still
larger (the SDXL checkpoint at 1024, no downscale) so Wan sees crisp edges;
another is the pixel-art LoRA above via the host-install path.

## Fire 2 — only if fire 1 leaves a clear next lever

Candidates, ranked by the run in its report, the Developer picks: the
`Wan2-2-pixel-animate` LoRA; a *different* end frame (the p4 spread-leg
still as the mid-stride extreme, run twice: A→B and B→A, concatenate);
`morphic/Wan2.2-frames-to-video` for 3–4 keyframes; LTX-2 19B distilled as
a faster second model for the same test. One fire, one lever.

## Host installs

As the standing text: a needed LoRA/model download goes out as
`waiting_external` in the test repository's yaml and a post in the
`workplan-` topic naming the Developer, with the read-only check that
proves it landed (`GET /models/loras` listing the filename is enough).
The Developer does the download; the GPU node's ComfyUI is user-session,
`observe_only` in Nautobot — dropping a file changes no desired state.

## Front

No change to `agfront` or its guide this phase. The fire text must not ask
Front to verify a render, count hashes, or confirm the survey happened;
it supervises, relays the run's report and asks the run's questions of the
Developer. The Developer opens the pictures (they land in the test
repository; copy the sheet into this folder for the report). If Front
volunteers a verdict on an artefact anyway, note it in the report as
information, not as a defect to fix here.

To avoid p4's 33 supervision runs: fire once, then read `workrun-` topics
**without posting** until the mission reports, or post only after Front's
own report has landed.

## Report

`report1.md` per fire; `report.md`: whether the survey happened (URL count),
the clip(s) as generated (keep one MP4 in the test repository, a contact
sheet here), the extracted sheet against `sheet_locked8.png` frame for
frame, the loop-closure and duplicate numbers, GPU wall time per clip,
costs from `agautolab/.local/agent/<role>/run-NNNN.json`, Deus Ex Machina
lines, and whether first+last actually beat first-only — the braindump's
claim, tested.

**MUST NOT**: host literals, internal repository names or agent names into
`main/` or this repository; push `publish/`; run a video matrix before one
timed clip.
