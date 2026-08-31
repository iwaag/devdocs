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

**Square works.** One clip, 640×640, `length` 124, seed 12345, first+last with
start = end, the p1 still padded (here: a plain nearest-neighbour downscale,
since the canvas is square and the source is square).

| | square 640×640 | p5's clip 2, 864×480 |
|---|---|---|
| wall | **421.1 s** | 477.8 s |
| frames | 124 | 124 |
| exact duplicate frames | **0** | 0 |
| mean adjacent-frame distance | 12.971 | 6.057 |
| **full-clip loop closure** | **0.2701×** | 0.4006× |
| gait period | **9** (harmonics at 18, 27) | 16 (half 8) |
| best stride-2 8-frame window | 0.7695× (start 95) | 0.6562× (start 233) |
| `vram_free` before submit | 46.38 GiB | — |
| `vram_free` after run | 12.54 GiB | — |

`preflight_square_contact.png` (16 frames evenly across the clip) and
`preflight_square_window.png` (the stride-2 window, frames 95–109) beside this
file.

**The pictures are good.** One dog at one scale in one palette, side view,
position and scale stable, a real gait with a suspended phase, flat background
held, no zoom, pan or identity drift. The two known defects are inherited
from the source still and survive: the lavender-not-white background and the
cast shadow baked along the ground line.

**One new defect, and it is the square clip's own.** The background is not
quite flat: faint grey ghost shapes drift through it, most visible in the
upper right. They are well inside the keying tolerance's reach at full
resolution but they are *structure*, not codec noise, and whether they
survive quantisation onto a shared palette is unknown until a sheet is cut.
Worth watching in the fire, not worth stopping for.

Three numbers deserve care, because two of them look like wins and are not
straightforwardly comparable:

- **It ran 57 s faster.** 51.2 M pixel-frames against 51.4 M, so the
  arithmetic held and the footprint did too; 33.8 GiB was still resident when
  the run ended, against a card of 47.26.
- **Loop closure 0.2701× against p5's 0.4006×.** Better, but the mean adjacent
  distance also more than doubled (12.97 against 6.06) — at 640×640 the dog
  fills the frame where at 864×480 it was letterboxed, so more pixels move per
  step and the *denominator* grew. The ratio is normalised, so it is not
  meaningless, but **this is not a controlled comparison of aspect ratio** and
  should not be reported as one. What it does establish is that the square
  clip loops at least as well as the 16:9 clip did, which is all the fire
  needs.
- **The gait period is 9, not 27.** The lag scan bottoms at 9, 18 and 27 with
  scores 1.146 / 1.115 / 1.099 — one period and its two harmonics. `analyze_loop.py`
  reports the *global* minimum and therefore called it 27. **That picker is
  wrong and it will be wrong again**: on a clean periodic signal the deepest
  minimum drifts to a multiple, because averaging over fewer pairs at long lag
  is noisier and cheaper to minimise. The fire should take the *smallest* lag
  whose score is within a whisker of the global minimum. p5 was not bitten by
  this only because its period was 16 and its scan stopped at 40.

**That has a direct consequence for the sheet**, and it is the most useful
thing this clip bought: p5's "best stride-2 window" rule was tied to p5's
period of 16, where stride 2 over 8 frames spans 14 frames ≈ one cycle. Here
one cycle is 9 frames, so stride 2 spans 1.5 cycles and the sheet reads
wrong. Measured over this clip:

| stride | span | best 8-frame closure |
|---|---|---|
| 1 | 7 frames | 0.8788× |
| 2 | 14 frames | 0.7695× |
| 3 | 21 frames | **0.5392×** |

**The extraction stride must follow the measured period; it is not the
constant 2.** Note also that no lag here scores *below* 1.0 — unlike p5's
clip, where lag 16 beat lag 1 outright. This gait is faster and its
periodicity is weaker, so the period is a real signal but a shallower one.

**Deus Ex Machina:** the Omni Agent ran the square 640×640 clip itself before
the fire, so the run does not spend a paid task discovering that the aspect
ratio does not work, and the stride finding above is handed over rather than
rediscovered. Handoff candidate: none for the clip — this is the preflight the
braindump explicitly asks the Omni Agent to do. **The period picker in
`analyze_loop.py` is a handoff candidate**: it is a four-line fix in the tool
the fire is about to copy.

### What the preflight changes about the fire text

- The six-node graph is now a measured literal, not a proposal.
- Add: **an identical resubmission is served from cache in 0.4 s.** Run 2 of
  the pipeline must differ in something the graph sees (a fresh
  `filename_prefix` is not enough — the sampler's own inputs must change, or
  the card must be freed first, which run 2 does anyway).
- Add: **the still is expected to be byte-identical between the two runs**,
  and the free between phases does not break that.
- The free is ~5 s, not 15 s.
- Square is proven, with numbers, so the fire inherits it instead of testing
  it: 640×640, length 124, 421 s, 0 duplicates, closure 0.2701×.
- **Do not hardcode stride 2 in the sheet extraction**, and fix the period
  picker to prefer the smallest lag at the minimum rather than the deepest.

### The fire

Posted to `#front › front-routine-mediagen` at 2026-08-31, message 4084.
Subject: **still → loop pipeline on one backend, repeatability**. Everything
the plan asked to travel verbatim is in it — the base URL, the five MiniMax
filenames, the SDXL checkpoint, the six-node text2img graph as JSON,
"1:1, 640×640, length 124, seed 12345", the free instruction phrased as
something the script owns, and the p5 prohibition on `low_vram` and CPU-CLIP
unchanged.

Two departures from the plan's fire text, both because the preflight measured
something the plan could not know:

- The square clip is handed over **as a result**, not as a thing to attempt.
  The fire says so plainly and tells the run its job is the pipeline around
  it.
- The stride and period findings above are handed over, with the instruction
  to fix the period picker in its copy of `analyze_loop.py` rather than
  inherit the bug.

The p5 relay failure is addressed directly in the first paragraph: *do not
replace the host-state readings with an instruction to confirm them live*.
That is the exact substitution p5 recorded, and it is the one thing a
paraphrase cannot be caught doing from the receiving end.

### Flagged, not fixed

agforge's production still-image path goes through SwarmUI, and SwarmUI's
`GenerateText2Image` is broken by the backend switch (its `comfyui_api`
backend asks the standalone ComfyUI for nodes that are not installed there).
**The preflight proves the cheaper of the two repairs works**: a plain
six-node ComfyUI graph renders the same asset with no SwarmUI in the path at
all. The decision — install SwarmUI's custom nodes into the standalone
ComfyUI, or move the production path off SwarmUI — is the Developer's, and is
not this phase's job.
