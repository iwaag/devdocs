# advance_mediagen_study p6 — Plan

Braindump: `braindump.md`. p5 proved the route (still → first+last-frame
video → 8-frame 64×64 sheet, loop closure 0.40×). p6 wants **more know-how
from repeated test generations**, and the braindump names what must be true
before any of that is worth firing:

1. The environment **repeats a generation stably** — image, then video, then
   again — without hand-holding.
2. SwarmUI's backend was switched to the **standalone ComfyUI**; something
   about model discovery looked wrong. Verify.
3. Probably drive **images through ComfyUI directly too**; SwarmUI buys
   little now.
4. Image model + video model likely do not fit in VRAM together — **free the
   card when switching**, even if slower.
5. Video at the **same aspect ratio as the still** (p5 used 16:9 for no good
   reason).
6. **The Omni Agent checks all of this before the fire**, so the run does not
   grind through the environment's problems again (p5 fire 1: 57 runs, no
   image).

Experimental, non-public environment; destructive phase, no backward
compatibility. **MUST NOT** lines are the only prohibitions; everything else
is advice the implementer may override with a stated reason.

## What the Omni Agent already found (2026-08-31, live checks)

Host literals (address, ports, model paths) stay out of this public
repository. They are in the workspace: `gentest-videoFrameExtraction/onecell.py`
constants (`DEFAULT_BASE_URL`, `UNET_NAME`, `LORA_NAME`, `CLIP_NAME`,
`VAE_NAME`, `AUDIO_VAE_NAME`) and `agforge/.local/.env`
(`AGFORGE_SWARMUI_URL`, `AGFORGE_COMFYUI_URL`). Copy them verbatim into the
fire text.

- **SwarmUI is back up** (p5's outstanding action is done), and its backend is
  now `comfyui_api` → the standalone ComfyUI on the same host, `OverQueue 1`,
  `max_usages 2`. **There is one ComfyUI process on the card now**, not two —
  the p5 "idle backend squats ~21 GiB / `FreeBackendMemory`" finding is
  obsolete, and `onecell.py`'s `free_swarmui_backend()` is dead code.
- **SwarmUI image generation is broken, and it is not the model path.**
  `GenerateText2Image` with `pixelArtDiffusionXL_spriteShaper` fails with
  ComfyUI `missing_node_type: SwarmJustLoadTheModelPlease`. A `comfyui_api`
  backend needs SwarmUI's own custom nodes installed in the remote ComfyUI
  (`SwarmComfyCommon`/`SwarmComfyExtra` under SwarmUI's
  `src/BuiltinExtensions/ComfyUIBackend/ExtraNodes/`, symlinked into
  ComfyUI `custom_nodes/`, then restart ComfyUI); the standalone instance has
  **zero** `Swarm*` node types. The models themselves are fine: the standalone
  ComfyUI lists `pixelArtDiffusionXL_spriteShaper.safetensors` under
  `checkpoints`, all five MiniMax H3 files, both Wan 2.2 i2v 14B files and
  `umt5_xxl`; `clip_vision/` is still empty.
  **Consequence for agforge**: its production still-image path
  (`generate.py` → SwarmUI) is broken by the backend switch. Not this
  phase's job; the Developer decides between installing the Swarm nodes and
  moving `generate.py` to ComfyUI. Flag it, do not fix it in the fire.
- Card: 46.4 of 47.26 GiB free, both queues empty, 132 GB RAM, ComfyUI
  0.33.1, 1633 node types. `MiniMaxH3ImageToVideo` inputs: `width`/`height`
  step 32, `length` step 17 from 5, tooltip: *"124 = ~5 s; trained range is
  ~124–362"* — so p5's unexplained "39 OOMs, 124 works" sits below the trained
  range anyway. **Keep `length` 124**; do not spend a fire on shorter clips.
- `ResolutionSelector` options include `1:1 (Square)` (its default); at
  0.4 MP / multiple 32 that is **640×640** — 51.2 M pixel-frames at length
  124 against the known-good 864×480's 51.4 M, so the VRAM footprint should
  match p5's successful cells. Or drop the node and set `width`/`height` =
  640 on the i2v node directly. 640 = 10 × 64, an integer pixel size.
- p5's tools to reuse (copy, do not import across repositories):
  `onecell.py` (graph, upload, submit/poll/fetch, `wait_for_vram_free`),
  `extract_sheet.py` (8 frames → keyed 64×64 sheet, one palette, interior
  pockets fixed), `analyze_loop.py` (duplicates, period, loop closure with the
  per-window normalisation fix). The p1 still literals are in
  `gentest-spriteSheetFrames/matrix.smoke.json` (`dog_walkcycle_frame1_contact`:
  CFG 4, steps 25, seed 12345, 1024×1024).

## Decisions

- **Images through ComfyUI directly.** A text2img API graph is six nodes:
  `CheckpointLoaderSimple` → `CLIPTextEncode` ×2 → `EmptyLatentImage`
  (1024×1024) → `KSampler` (seed 12345, steps 25, cfg 4, sampler/scheduler
  as SwarmUI defaulted: `euler`/`normal` unless the p1 record says otherwise)
  → `VAEDecode` → `SaveImage`. Same POST `/prompt` / `/history` / `/view`
  path `onecell.py` already uses. Nothing in this phase talks to SwarmUI;
  leave it running, do not reconfigure it.
- **Free the card between phases, in code.** `POST /free`
  `{"unload_models": true, "free_memory": true}` then poll `/system_stats`
  until `vram_free` stops rising (`wait_for_vram_free`, 15 s budget). p5
  showed every *prose* instruction about `/free` got inverted in relay — so
  the pipeline script owns the call and the fire text says only "the script
  frees the card between image and video". SDXL leaves ~7 GB resident; the
  MiniMax run peaks near 42 GB; the margin is real but untested — measure
  `vram_free` before each submit and record it.
- **Square, 640×640, length 124, seed fixed.** The still is padded, never
  stretched (p5: a 3.85 % stretch cost 2.8× in loop closure).
- **One clip per task.** A clip is ~8 min GPU; `WORK_TIMEOUT_SECONDS` 1200
  fits one comfortably, two barely. If a task needs more, submit and let a
  later task poll (`matrix_finish.py` pattern) rather than sitting on it.

## Step 0 — Omni Agent preflight (before any fire)

Cheap, read-mostly, and it is what the braindump asks for. Record each
result in `report1.md`.

1. `agentchat read front front-routine-mediagen` — any Developer post you
   did not write means a second Omni session; settle who drives first.
2. `GET /queue` empty, `GET /system_stats` `vram_free` ≥ ~40 GiB, `GET
   /models/checkpoints` lists the SDXL checkpoint, `/models/diffusion_models`
   lists the MiniMax UNET. (Done today; redo on the day.)
3. **One text2img through ComfyUI by hand** with the six-node graph above at
   the p1 literals (~8 s warm, ~27 s cold). Open the PNG. This is the
   half of the pipeline nobody has run on this backend yet. Keep the graph
   JSON — it goes into the fire verbatim.
4. `POST /free` + poll; confirm `vram_free` returns to ≥ 46 GiB. Note the
   time it took.
5. Optional, ~8 min GPU: `onecell.py --size 640x640` once, to know before the
   fire whether square works. If you do it, that is a Deus Ex Machina line
   and the run still owns the timed clip.

If step 3 fails, fix or report before firing — do not hand the run an
environment problem.

## Fire 1 — the pipeline, run twice

Fire `routine-mediagen` from `#front › front-routine-mediagen` (context inside
the firing post; any post there *is* a fire). Subject: **still → loop
pipeline on one backend, repeatability**. New repository (every existing
yaml is `verified`; the workflow family stays `videoFrameExtraction`), name
the run's choice, e.g. `gentest-videoLoopPipeline`. Mission shape, ~4 tasks:

1. **`pipeline.py`** — one command, one seed: text2img (ComfyUI) → save the
   1024 still and its 64×64 NN downscale → pad to 640×640 → free → MiniMax
   first+last clip (start = end) → free → frames → `extract_sheet` (best
   stride-2 window of the measured period, as p5) → `analyze_loop` → one
   JSON record per run: wall time per stage, `vram_free` before each
   submit, prompt ids, hashes, loop numbers. No SwarmUI code at all.
2. **Run it twice back to back**, same seed. Compare: still hash (SDXL at
   fixed seed should be byte-identical — if not, say so; that is know-how),
   clip loop closure / period / mean adjacent distance (expect small drift;
   report the delta), wall times, and whether run 2 needed anything run 1
   did not. This is the braindump's line 4, with a number.
3. **One axis, two cells, on the proven pipeline** — pick from p5's open
   items, the run's choice with a reason: pose amplitude by prompt wording
   ("trotting"/"running", explicit stride language) — the cheapest and the
   one the deliverable most needs; or start-image preparation, A = 1024
   render → 640 (p5's control, non-integer scale) vs B = 1024 → 64 NN → 640
   NN (a true pixel grid in — does a grid come out?); or the shadow
   geometric key (post-processing only, no GPU). Pictures first; every
   metric here has ranked its blind spot first.
4. **Improve**: tips (evidence line + date) in
   `main/subjects/videoFrameExtraction/tips.md` including the backend
   change, the missing-Swarm-nodes fact in host-neutral words, and the
   repeatability numbers; INDEX row; publication self-check with
   `git grep --untracked` (p5: tracked-only scan missed the whole new
   directory); push `main` and the test repository.

Things to put in the fire text verbatim: the ComfyUI base URL and the five
MiniMax filenames from `onecell.py`; the SDXL checkpoint filename; the
text2img graph from preflight step 3; "1:1, 640×640, length 124, seed
12345"; "the script frees the card between image and video; measure, do not
assume"; and the p5 prohibition, unchanged: **do not set `low_vram: true`
or `CLIPLoader.device: cpu`; if a cell OOMs, stop and report with the
`vram_free` readings.**

## Fire 2 — only if fire 1 leaves one clear lever

Candidates the run should rank in its report: the `Wan2-2-pixel-animate`
LoRA (host download via `waiting_external`); a different end frame (p4's
spread-leg still, A→B and B→A concatenated); the same pipeline on Wan 2.2
14B FLF2V with the lightx2v LoRAs, to have a second model's numbers. One
fire, one lever.

## Front

Unchanged from p5: it supervises the conversation, relays, asks the run's
questions; it does not verify artefacts. Fire once, then read `workrun-`
topics without posting until the mission reports. Open the pictures
yourself before accepting any task whose claim rests on them.

## Report

`report1.md` per fire (preflight results first); `report.md`: did the
pipeline repeat (both JSON records, side by side), the VRAM readings at
each boundary and whether the free was needed, the square clip against
p5's 864×480 clip 2 (loop closure, period, sheet frame for frame), the axis
result, GPU minutes, costs from `agautolab/.local/agent/<role>/run-NNNN.json`
and `agfront/.local/agent/front/`, Deus Ex Machina lines, and the agforge
still-image path decision handed back to the Developer.

**MUST NOT**: host literals, internal repository names or agent names into
`main/` or this repository; push `publish/`; re-apply `low_vram`/CPU-CLIP;
run the axis (step 3) before the pipeline has repeated once (step 2).
