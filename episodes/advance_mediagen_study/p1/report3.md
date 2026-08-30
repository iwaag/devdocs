# Step 3 — Fire it once, on animation

Fired 2026-08-30 06:39Z. Mission **M-9**, four tasks, finished 07:20Z —
**41 minutes wall clock**, 20 paid runs, **$7.44**.

## How it was fired

By hand, as a Developer post into `#front › front-routine-mediagen`
carrying the subject note — **not** `trigger.sh mediagen` and not an
`rtschedule` event. The note was written to be posted *before* the trigger,
but Front's listener is event-driven and began serving the instant the note
landed; running `trigger.sh` afterwards would have double-fired the routine
into a second mission. The two paths are the same mechanism anyway —
`trigger.sh` is itself only a Developer post into that topic — so what this
fire did not exercise is the dispatcher, not the routine.

Worth recording as a small trap for anyone scripting this later: **you
cannot post context and then fire.** Any post in a `front-` topic is the
fire. Context has to be inside the firing post.

## The cycle, end to end

| task | what | wall | turns | cost |
|---|---|---|---|---|
| plan | superdirector: `plan.md` + four tasks | 124 s | 13 | $0.35 |
| 1 | Gather + Invent: survey, ranking, name the family | 274 s | 35 | $1.01 |
| 2 | Verify 1: new repository + one timed generation | 188 s | 28 | $0.59 |
| 3 | Verify 2: the Q7 matrix — **first attempt, stalled** | 222 s | 21 | $0.45 |
| 3′ | Verify 2 resumed: matrix + Q8 instrument | 450 s | 57 | $1.58 |
| 4 | Improve: tips, queue, index, commit and push | 169 s | 30 | $0.66 |
| — | 6 further autolab runs (commit-after-approval, acceptance) | | | $1.05 |
| — | 11 Front runs (supervision, one correction) | | | $1.75 |
| | | | **20 runs** | **$7.44** |

p9's estimate was "a two-task mission is about five paid runs". A four-task
mission cost twenty. The ratio held.

## Gather — the survey and the ranking

Task 1 ran nine web searches and four fetches, and wrote
`main/spriteSheetFrames/summary.md`. The finding, in its own framing:

- **Sprite-sheet-of-generated-frames is what actually ships.** Every
  consumer-facing pixel-art generation tool it found (Seele,
  Sorceress/Quick Sprites, Sprite-AI, PixelLab, spritegen.ai, PixExact,
  pixie.haus) returns a frame sequence or a sheet, consumed as ordinary
  engine flipbook animation. No bone rig anywhere in that set.
- **Skeletal rigging is the standing professional 2D pipeline, and AI
  enters it upstream of the rig, not inside it.** Every skeletal example
  found starts from a diffusion-generated *still* reference or part image
  that a human cuts and rigs by hand in dedicated software. The one
  directly relevant paper — Sprite Sheet Diffusion, arXiv 2412.03685 — is
  not pixel-art and reports that its own pose-estimation step "does not
  generalize well to illustrations in sprite sheets".
- Consistency techniques found in the wild: locked seed with pose-only
  prompt edits; image-to-video as a motion prior; ControlNet/pose-guided
  multi-frame rendering.

The **ranking by present-day certainty**, which is the part the phase
actually wanted the run to produce on its own:

1. **Locked-seed, pose-shifted multi-still generation** — highest. No
   download, runs through the existing text2img call and the existing
   post-process pipeline.
2. **Image-to-video as a motion prior** (Wan 2.2 I2V) — medium. No
   download either, but holds the GPU for minutes per generation and needs
   a new frame-extraction and pixel-grid-snap step.
3. **ControlNet / pose-guided multi-frame rendering** — *the technique with
   the best public evidence for frame-to-frame consistency*, ranked third
   anyway because the weights are not on the box.
4. **Skeletal rig over generated parts** — lowest. No scriptable rigging
   path on this host, and the survey found the step is predominantly manual
   even in pipelines that use it.

That third line is the ranking earning its keep. The run had every reason to
pick the method with the best literature and did not, because "present-day
certainty" was defined as *runs on what is here*. It also declined to open a
`waiting_external` for the ControlNet weights, on the correct ground that a
method it did not select needs no handoff.

The workflow family was named **`spriteSheetFrames`** — the run's own
choice, filled back into Q6/Q7/Q8's subject fields, which is exactly the gap
`report1.md` predicted would trip something. It did not trip it.

## Invent — the requirement, stated before generating

Reused verbatim from the plan's suggestion, which itself extends p9's
requirement: a 64×64 side-view walking cycle of a four-legged animal, 4–8
frames, looping, one shared ≤32-colour palette across all frames, per-frame
PNGs plus a sprite sheet, consistent silhouette, flat keyable background.

Matrix: dog (p9's best-behaved subject), tag-style prompt, CFG 4, steps 25
— all inherited rather than re-swept — **frame count {4, 8} × seed strategy
{locked, varied}**, 24 generations. One axis that matters, one control.

## Verify — the repository and the runner

`gentest-spriteSheetFrames/`, created with `autolab project init-repo` plus
a hand-written `localtest.yaml`, recorded in `README_PROJECT.md`. Not an
extension of `gentest-pixelArtDiffusionXL`, whose yaml is `verified` — the
pattern line added in step 1 held.

`runner.py` was copied from the checkpoint test and its post-processing
reused unchanged. Two new files on top:

- `sequence_runner.py` — generates a pose sequence, computes **one shared
  palette per sequence** and force-quantises every frame onto it, writes
  per-frame PNGs and assembles a sheet.
- `consistency_instrument.py` — palette drift, silhouette area/centroid
  variance, pixel-grid alignment.

Invocation is the inherited one:
`uv run --with pillow --with requests python sequence_runner.py matrix.walkcycle.json`.

**Backend used: SwarmUI's text2image API** (not ComfyUI — the method is pure
SDXL text2img, so the simpler surface was the right one). Timings: **25.6 s**
for the first generation after a cold checkpoint, **7.9–8.5 s** warm — both
matching the figures carried verbatim in the workplan, which is a small
independent confirmation that those literals were still true.

## The result

![locked8 sprite sheet](sheet_locked8.png)

*`sheet_locked8.png` — 8 frames, seed locked at 12345, pose clause the only
thing changing. Same dog throughout: same white-and-brown markings, same
build, same scale.*

![varied8 sprite sheet](sheet_varied8.png)

*`sheet_varied8.png` — the control. Same prompts, seed incremented per
frame. Scale swings wildly, markings change, several frames read as a
different animal; two frames are a distant tiny dog against a different
background.*

The two sheets are the whole finding and they need no metric to read.

**Q7 — closed.** Locked-seed sequences meet the requirement at both 4 and 8
frames: 64×64, side-view, four-legged, per-frame PNGs plus a sheet, one
shared palette of 29–32 colours (all ≤32), background flat within every
sequence. Varied-seed sequences do not. **Seed-locking, not the pose-clause
wording, is what carries identity across frames.**

What was honestly *not* met, and said so in `tips.md` rather than glossed:

- **"Looping" was not verified.** The last pose hands back to the first no
  better or worse than adjacent poses do. Raised as Q9.
- The background is flat but **lavender, not white**, and the cast ground
  shadow is present in every frame — both inherited from
  `pixelArtDiffusionXL` and on record there, neither new to this method.

## The instrument and its verified blind spot

This is the part of step 3 that most needed to go well, and it did.

`consistency_instrument.py` was tested against **two** known-bad cases
before being trusted:

1. a **deliberately broken** sequence — `locked4`'s intact frames 1/2/4 with
   `deer_sentence_cfg9` (p9's own on-record outright failure, a forest
   scene) spliced in as frame 3, resized but not re-quantised;
2. the **naturally bad** `varied8`.

| signal | spliced alien frame | `varied8` (real failure) |
|---|---|---|
| palette drift | **1.0000** vs 0.0000 — unambiguous catch | **0.0000** — completely blind |
| silhouette area stdev | 41 → 170 — real but much smaller jump | 259 px on mean 691 (≈38%) vs 94 on 940 (≈10%) for `locked8` — catches it |
| pixel-grid alignment | 0.115, **inside** the intact range 0.10–0.28 — no flag at all | 0.215 stdev on mean 0.56 vs 0.041 on 0.18 — catches it |

**No single signal covers both failure modes**, and the blind spot is
structural rather than incidental: palette drift reads 0.0000 on the worst
real sequence *because `sequence_runner.py` force-quantises every frame onto
one shared palette before saving*. The instrument is blind to precisely the
failure the pipeline it measures is most likely to produce. Recorded as
such, with the instruction to use palette drift plus one of the other two
together, always beside the images, never as a sort key.

That is a strictly better outcome than `bg_flatness`, which was trusted for
a whole run before p9 caught it. Here the run caught its own instrument
inside the same task that built it.

## Frictions

**1. The task-3 stall — the expensive one.** The first task-3 run launched
the 24-generation matrix with `run_in_background`, then set up a monitor,
ran `echo waiting` and `true`, posted "I'm waiting for the background
generation to finish, I'll continue once it completes", and **ended its
turn**. A `workrun-` serving is one-shot: the turn ending *is* the run
ending, and the background process was killed with it. 19 of 24 images
survived on disk; every timing from those 19 was lost, because the runner
had not yet flushed a results file.

It was **not** a timeout. The run used 222 s of a 1200 s budget and 21 turns
of a 200-turn cap — 18% and 10%.

**2. Front then stalled too, for eight minutes, and its reasoning was
sound.** Front read the post, checked the topic directly, and concluded:

> "posting into that topic now would just restart autolab's turn for no
> reason. I'll wait until it comes back with actual progress."

Every step of that is correct except the premise. autolab could not come
back. This is p9's failure with the polarity flipped — there a supervisor
read "opened …" as "running now"; here it read "waiting in the background"
as "running now". Both are the same missing distinction: **an agent saying
it is waiting is not progress, it is a stall.** Progress is a result or a
question.

Neither side could recover alone, so this needed a human. See Deus Ex
Machina below.

**3. A push that reported success and had not landed.** Task 3's report said
it had pushed `gentest-spriteSheetFrames`. Task 4 checked with
`git ls-remote origin`, found **zero refs on the remote**, and pushed for
real. Front had asked for that double-check on its own initiative, having
noticed task 3's report only mentioned pushing `main` — good supervision,
and the only reason this was caught. Had it not been, both commits would
have existed on one laptop only while every report said otherwise.

**4. No GPU contention with agforge.** The card was free throughout; warm
timings never drifted from ~8 s. Not a friction this time, but the matrix
was small enough that it would not have shown.

**5. One bogus timing, self-flagged.** `varied8_f3_passingR` recorded 0.56 s
— a resume artefact from the killed first attempt. The run identified it as
such, excluded it, and wrote it into `tips.md` as a caveat rather than
leaving it in the results table looking like evidence. That is the same
discipline p9 applied to `cat_sentence_cfg9`.

## Queue hygiene — one defect in my own seed format

`QUESTIONS.md` now holds 11 entries: Q1–Q5 untouched, Q6/Q7/Q8 marked
`closed 2026-08-30` with pointers to the answering tips, Q9/Q10/Q11 newly
raised (does the cycle loop; a deliberate same-palette content-drift test
case for the instrument; does the shadow/background defect move frame to
frame).

But the file's `## Closed` section still reads `*(none yet)*`. The run
marked the entries closed **in place** in the `## Open` section rather than
moving them. That is my fault, not the run's: I seeded the file with *both*
a per-entry `Status:` field *and* `Open`/`Blocked`/`Closed` section
headings, which is redundant, and gave no instruction about which one wins.
The run picked the one that preserves reading order and did not touch the
other. Its choice is defensible; the format was ambiguous. Fix in the next
phase: drop the section headings and keep the status field, or say plainly
that the headings are the index and entries move.

## Deus Ex Machina

- **Broke the task-3 / Front deadlock.** After eight minutes of neither side
  moving, I posted into `front-routine-mediagen` as the Developer explaining
  that a `workrun-` serving is one-shot, that the post Front was waiting on
  *was* the end of autolab's run, that 19/24 images were on disk and the
  runner is resumable, and that the retry must run the generation in the
  foreground. **I deliberately injected information and not action** — Front
  restarted task 3 itself under its standing authorization, which kept the
  question "can this run without a human?" answerable rather than moot. The
  answer is no, not yet, and this is exactly where. Handoff candidate, in
  two one-line pieces:
  - agfront's guide learned in p7 that waiting happens inside the run;
    autolab's supercoder guide never did.
  - Front has no rule saying an agent's "I'm waiting" is a stall to restart,
    not progress to wait on.
- Fired the routine by hand post rather than through the dispatcher —
  handoff candidate, minor; the dispatcher path is proven and was skipped
  only to avoid a double fire.
- Copied two sprite sheets into this public episode folder for the report.
  No host facts in them.
