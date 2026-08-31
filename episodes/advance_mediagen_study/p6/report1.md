# advance_mediagen_study p6 — step report 1

Plan: `plan.md`. This file covers **Step 0, the Omni Agent preflight**, and
will carry **fire 1** when it runs.

Host literals (address, ports, model filenames) are deliberately absent here,
per the plan; they live in the workspace files the plan names.

## Step 0 — preflight, 2026-08-31

All five items done, including the optional one. **Nothing in the environment
is blocking a fire.**

### 1. Is a second Omni Agent session driving?

`agentchat read front front-routine-mediagen` — the topic ends on 2026-08-30
with p5's close ("Stand down for good on this fire") and Front's
acknowledgement. **No post today, from the Developer account or anyone
else.** Nobody else is driving; this session takes the fire.

### 2. Host state

| check | reading |
|---|---|
| `GET /queue` | running 0, pending 0 |
| `GET /system_stats` | ComfyUI 0.33.1, 123.4 GiB system RAM |
| card | Quadro RTX 8000, **46.40 GiB free of 47.26** |
| `GET /models/checkpoints` | the SDXL pixel-art checkpoint is listed |
| `GET /models/diffusion_models` | the MiniMax H3 UNET is listed |
| `GET /object_info` | 1633 node types, **zero `Swarm*`** |
| SwarmUI root | HTTP 302 — it is up |

The plan's two standing facts are re-confirmed on the day: one ComfyUI process
on the card, and the standalone instance carries none of SwarmUI's own custom
nodes. `MiniMaxH3ImageToVideo` still declares `width`/`height` step 32 and
`length` step 17 from 5, tooltip *"124 = ~5 s; trained range is ~124–362"*.

### 3. One text2img through ComfyUI, by hand

The six-node graph from the plan, at the p1 literals (1024×1024, steps 25,
cfg 4, seed 12345, `euler`/`normal`, p1's positive prompt and the smoke
matrix's shared negative prompt). Graph JSON kept for the fire text.

**It works.** `preflight_t2i_still.png` beside this file: the p1 dog, side
view, mid-stride, flat lavender background, cast shadow along the ground line
— the same picture family p1 and p5 worked from, including p5's two known
defects (lavender not white, baked shadow).

**This is the half of the pipeline nobody had run on this backend, and it
needed no SwarmUI-specific machinery at all.** Six nodes, the same
`POST /prompt` → `/history` → `/view` path the video tool already uses.

Three runs, and each one bought a number:

| run | condition | wall | vram_free before → after |
|---|---|---|---|
| 1 | first ever load of this checkpoint | **26.2 s** | 46.40 → 39.77 GiB |
| 2 | identical graph resubmitted | **0.4 s** | 39.76 → 39.76 GiB |
| 3 | after `POST /free` | **10.1 s** | 46.38 → 39.77 GiB |

- **SDXL's resident footprint is 6.63 GiB.** Against a 42-ish GiB MiniMax peak
  on a 47.26 GiB card, that is the margin the plan called real but untested —
  and it is why the free between phases is not optional bookkeeping.
- **Run 2 did not execute.** ComfyUI deduplicated the identical prompt,
  returned the *same* output file in 0.4 s and touched no VRAM. Worth knowing
  before the fire: **re-running the identical graph is not a repeat
  generation**, so a repeatability test that changes nothing measures the
  cache, not the model.
- **Run 3 is byte-identical to run 1** — same sha256 over the RGB bytes, mean
  absolute difference 0.0000 across 1024×1024×3 — across a full model unload
  and reload. **SDXL at a fixed seed is deterministic on this box**, so fire 1
  step 2's still-hash comparison has a known right answer; a mismatch there
  would be a real finding about the pipeline, not about the model.

**One thing is not identical: p1's own still.** Same checkpoint, same prompt,
same seed, steps and cfg — but the ComfyUI render differs from the SwarmUI
render p1 produced by mean absolute difference **0.125** (max 78 on one
channel). The same picture, not the same file. Something in SwarmUI's
wrapping — its own sampler defaults, a prompt-weighting convention, an extra
node — moves the pixels slightly. **It is a caution for anyone comparing a
p6 still against a p5/p1 artefact by hash**, and one more reason the fire
should compare its two runs against *each other*.

### 4. `POST /free` and the poll

`POST /free {"unload_models": true, "free_memory": true}` returned HTTP 200 in
under 10 ms with the memory not yet released — exactly the trap
`wait_for_vram_free` exists for.

`vram_free` **39.76 → 46.38 GiB**. The first poll at 1.5 s already read the
final value; the "stopped rising" detector confirmed it at 4.6 s. So the
plan's 15-second budget is roughly three times what this actually needs, and
the free costs about **5 seconds of wall time** per boundary. Against ~480 s
of clip, that is noise — *"even if slower"* in the braindump turns out to cost
nothing worth counting.

46.38 GiB, not the 46.40 GiB of a cold card: about 20 MiB stays resident. That
is the CUDA context, not a model.

### 5. Optional — the square clip

Run. Reported in the next section, because it is the answer to the one
question the fire cannot afford to discover for itself.

**Deus Ex Machina:** the Omni Agent ran the square 640×640 clip itself before
the fire, so the run does not spend a paid task discovering that the aspect
ratio does not work. Handoff candidate: none — this is the preflight the
braindump explicitly asks the Omni Agent to do.

### What the preflight changes about the fire text

- The six-node graph is now a measured literal, not a proposal.
- Add: **an identical resubmission is served from cache in 0.4 s.** Run 2 of
  the pipeline must differ in something the graph sees (a fresh
  `filename_prefix` is not enough — the sampler's own inputs must change, or
  the card must be freed first, which run 2 does anyway).
- Add: **the still is expected to be byte-identical between the two runs**,
  and the free between phases does not break that.
- The free is ~5 s, not 15 s.

### Flagged, not fixed

agforge's production still-image path goes through SwarmUI, and SwarmUI's
`GenerateText2Image` is broken by the backend switch (its `comfyui_api`
backend asks the standalone ComfyUI for nodes that are not installed there).
**The preflight proves the cheaper of the two repairs works**: a plain
six-node ComfyUI graph renders the same asset with no SwarmUI in the path at
all. The decision — install SwarmUI's custom nodes into the standalone
ComfyUI, or move the production path off SwarmUI — is the Developer's, and is
not this phase's job.
