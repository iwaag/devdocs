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

## Fire 1 — the mission

Front relayed the fire faithfully. I checked the relayed text against my own
word for word: the six-node graph JSON, the base URL, the model filenames,
`421.1`, `0.2701`, the stride table, the period-picker bug, the ghost-shape
defect and both prohibitions all arrived intact, split over five posts
because the payload exceeded one send.

**Front caught its own truncation before relaying anything.** The fire post
was cut off in its chatlog mid-sentence, and rather than pass on a partial
literal it stopped and asked for the tail. That is the exact opposite of p5's
failure, where a missing paragraph was replaced with a plausible instruction
of its own. Asked for, and got, the right behaviour.

autolab planned, then **caught that its own first plan had been written
against the truncated text and rewrote it** before Front raised either of the
two objections it had prepared.

### Task 1 — the pipeline, and a rule I failed to state

Task 1 built `pipeline.py` and the three adapted tools without touching the
GPU. It wrote the six-node still graph fresh (no such graph existed anywhere
in the project — every prior still went through SwarmUI), fixed the period
picker and verified against my own 9/18/27 numbers that it returns 9.

Then it did something reasonable that was wrong, and the fault is mine.

I had told it *"the stride must follow the measured period"* and given three
measured numbers — but **I never said what the rule was.** So it invented one:
sweep every stride from 1 to period−1 and keep the lowest closure ratio. On
my own clip that rule picks **stride 5**:

| stride | span | cycles spanned | best closure ratio |
|---|---|---|---|
| 1 | 7 | 0.78 | 0.8788 |
| 2 | 14 | 1.56 | 0.7695 |
| 3 | 21 | 2.33 | 0.5392 |
| 4 | 28 | 3.11 | 0.4257 |
| **5** | **35** | **3.89** | **0.3608 ← the sweep's winner** |
| 6 | 42 | 4.67 | 0.5179 |
| 7 | 49 | 5.44 | 0.6480 |
| 8 | 56 | 6.22 | 0.5540 |

Stride 5 advances **0.56 of a gait cycle per sheet frame**. The aliasing limit
is 0.5. It clears it by 0.06 — no margin at all; move the period by one frame
and the same rule returns a sheet that reads backwards or stands still, which
is p5's own "every sample at the same phase" failure restated.

**The metric cannot see this by construction.** Closure ratio is the wrap
distance over the window's *own* mean adjacent distance, so a larger stride
inflates the denominator and the ratio falls for reasons unrelated to whether
the sheet animates. `stride_rule_evidence.png` beside this file shows the
stride-1 and stride-5 windows together.

The rule I should have written: **stride ≈ round(period / 8)**, so the eight
frames span about one gait cycle; closure ratio chooses only the *window
start*. For period 9 that is stride 1 — which the sweep ranks **last**.

**p5's "stride 2" was never a constant.** p5's period was 16 and 16/8 = 2. It
was period/8 the whole time, nobody wrote the rule down, and it travelled as
a magic number. My fire told the run to stop treating it as a constant and
still did not say what it was — the same shape as p5's loop-closure metric,
where the specification, not either implementation, was at fault.

### Task 2 — the pipeline repeats, exactly

**Every number is identical, and so is every byte.**

| | run 1 | run 2 |
|---|---|---|
| frames | 124 | 124 |
| exact duplicate frames | 0 | 0 |
| mean adjacent distance | 13.1599 | 13.1599 |
| full-clip closure distance | 3.5630 | 3.5630 |
| **full-clip closure ratio** | **0.2707** | **0.2707** |
| measured period | 9 | 9 |
| chosen stride | 1 | 1 |
| best 8-frame window | 44, ratio 0.9096 | 44, ratio 0.9096 |

Beyond the metrics: **124 of 124 clip frames are byte-identical between the
two runs**, as are all three stills. The runs are ~25 minutes apart with a
card free and a full model unload between them.

The plan expected "small drift; report the delta". **The delta is zero.** Not
approximately — the same bytes. The braindump's first requirement, that the
environment repeat a generation stably, is answered as strongly as it can be:
on this backend, at a fixed seed, both SDXL *and* MiniMax H3 are exactly
reproducible, and a `/free` between phases does not perturb either.

The still is also byte-identical to the one I rendered by hand in the
preflight, hours earlier from an independently written script — so the
determinism is a property of the backend and the seed, not of one program.

Run 2's per-stage timings, its own measurements:

| stage | s |
|---|---|
| still | 10.3 |
| pad | 0.0 |
| free before video | 3.3 |
| **video** | **422.3** |
| free after video | 6.8 |
| loop analysis | 4.0 |
| sheet extract | 0.1 |
| **total** | **446.9** |

The two frees cost **10.1 s of a 447 s run — 2.3 %**. The braindump's *"even if
slower"* is answered: freeing the card between models is not a trade-off worth
agonising over, it is free.

### The sheet

`fire1_sheet_4x.png` beside this file (512×64 shown at 4×, nearest-neighbour).
**512×64 RGBA, eight 64×64 cells, 8 unique RGBA values** — seven opaque plus
transparent, inside p1's ≤32-colour clause, background keyed clean.

It is a real walk cycle: legs advance coherently cell to cell, one dog at one
scale in one palette. **This matters because its closure ratio is 0.9096** — a
poor number by the metric that chose it. autolab opened the picture, said so,
and reported the picture as decisive rather than reconciling to the score.
That was the instruction and it followed it.

**The ghost-shape defect I flagged in preflight survived quantisation**, as
feared: grey specks sit in the keyed background of several cells, clearest to
the right of the dog. They are background structure, not codec noise, and a
corner-sampled colour-tolerance key does not remove them. Along with the cast
shadow — still sharing a palette entry with the outline — that is two
inherited defects the keying cannot reach.

### Five failures to finish one task, and only one was a bug

Task 2 took five attempts. The tally matters because they were not the same
failure:

- **Three orphanings.** autolab launched the pipeline, posted *"I'll wait for
  the Monitor's notification"*, and its run ended — taking the pipeline
  process with it. One of those cost ~7 GPU minutes and produced only a still.
- **One crash**: a frame-numbering bug in `build_sheet`, which autolab found
  and fixed itself.
- **One belief.** Its last attempt used a harness-tracked background task and
  reported *"a reply here is not required to resume me — the background task
  itself will trigger my next turn."* It will not.

**That belief is the finding.** An in-system agent's serving is one-shot;
**nothing resumes it but a post into a topic it watches, so it cannot delegate
its own continuation to a process it spawns.** Every stall landed in the same
place for this one reason. The working shape is to block inside the run — one
clip is ~450 s against a 1200 s work timeout, which is what the mission's "one
clip per task" was for — or to end the run saying plainly that a post is
required.

I corrected myself once here: I first reported the crash as a fourth
orphaning. A crash and an orphaning carry different lessons and I had merged
them.

Worth recording on the other side of the ledger: run 1's record could not
carry its own stage timings, because the timers died with the crashed process.
autolab **reconstructed them from file mtimes and labelled them as
approximations, naming what was lost**, where inventing plausible numbers
would have been easy and undetectable.

### Deus Ex Machina, fire 1

- **Read the card and the disk directly at every stall**, so each resume
  instruction carried the true state — including "run 1's GPU work is already
  on disk, do not regenerate it", which saved a second wasted clip. Not a
  handoff candidate; this is the disinterested-eyes role.
- **Diagnosed the stride rule and proved it on my own clip** before task 2
  spent GPU on a wrong sheet. Handoff candidate in part: the rule now exists
  in the code, but nothing would have caught it except somebody re-deriving
  the metric's blind spot.
- **Unstalled the mission four times through Front.** Handoff candidate,
  urgently: a supervisor that polls its own agent's liveness would have caught
  every one of these, and Front had no reason to look.
