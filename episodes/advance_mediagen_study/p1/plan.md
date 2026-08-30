# advance_mediagen_study p1 — Plan

Braindump: `braindump.md`. The `mediagen` study project exists and has run
exactly one hand-written generation experiment (`scheduled_routine` p9,
mission `m-4`: a prompt/CFG matrix on `pixelArtDiffusionXL_spriteShaper`).
This phase does two things:

- **A. Strengthen the framework** so the study runs as a *cycle* — gather →
  invent → verify → improve — that picks its own next topic, instead of a
  Developer hand-writing each matrix. The missing organ is a consumable
  **open-question queue**: today every run's unknowns die at the bottom of
  `tips.md` under "Still open" and nothing reads them back.
- **B. Prove the cycle** by firing it once on this round's subject:
  **animation workflows for 2D pixel-art assets that assume AI generation** —
  skeletal animation vs. sprite-sheet animation. Gather web information,
  pick the single method with the highest present-day certainty, and prove
  that one method locally in a new generation-test repository.

Experimental, non-public environment; destructive phase, no backward
compatibility. Only the **MUST NOT** lines are prohibitions — everything
else is advice the implementer may override with a stated reason.

## Background the implementer should know

### What p9 already established (do not re-derive)

- The study pattern is `agautolab/agent/project_pattern.md`: `main/` =
  publish-ready knowledge, `publish/` = reviewed copy (never pushed by an
  agent), plus repository-backed **generation tests** —
  `autolab project init-repo gentest-<subject>` + a hand-written resumable
  yaml, distilled into `main/<subject>/tips.md` (append-only, one evidence
  line per tip, no level scale). A pattern is a starting layout, not a
  constraint; new folders go into `README_PROJECT.md`.
- Missions are `workplan-…` topics in `#pj-mediagen`, posted by the
  Developer; each task gets a `workrun-` topic and **a post is what starts
  it**. A task is not closed until the requester agrees, and that acceptance
  is a *scheduling gate* — the next task will not launch until the previous
  one is in a completed Plane state. Budget accordingly: a two-task mission
  is about five paid runs, not three.
- `WORK_TIMEOUT_SECONDS = 1200`. ~20 SDXL images fit in one task. **A video
  matrix does not** — size video work per task and measure one generation
  before committing to a grid.
- A mission cannot quote its own cost; read
  `agautolab/.local/agent/<role>/run-NNNN.json`.
- **Planning paraphrases the mission, and a paraphrase drops literals.**
  p9 lost the backend endpoint between the workplan and `task1.md`, and the
  supercoder burned most of a 38-turn, $0.77 task rediscovering it. Anything
  a task cannot re-derive — endpoint, model filename, credential path —
  must be written in the workplan in a form that survives paraphrase, and
  the workplan should say so out loud.

### The findings this phase builds on (`main/pixelArtDiffusionXL/tips.md`)

The target requirement p9 judged against, worth reusing verbatim so the
animation work extends existing knowledge instead of starting a new subject:

> A **64×64 side-view walking sprite of a four-legged animal**, for a 2D
> game. Single subject, centred, whole body in frame with a little margin.
> Flat, uniform background that keys out to transparency cleanly. Readable
> silhouette at 64×64. At most 32 colours after quantisation. Consistent
> outline.

- Subject choice dominates every swept axis; deer fails where fox/dog/cat/
  horse do not.
- A cast ground shadow appears in every cell and **cannot be removed by
  negative-prompt wording** — route around it in post, not with more words.
- A flat key-able background is reachable; the specific colour "white" is
  not (0/20 cells).
- The `bg_flatness` instrument **misleads**: it ranked the one outright
  failure as the flattest cell. Do not reuse it as a sort key, and expect
  any new instrument this phase invents to need the same scepticism.
- Still open and directly relevant here: the `pixel art`-early-in-prompt
  disagreement, `pixel-art-xl` LoRA vs. the dedicated checkpoint, palette
  size / sampler / steps, and whether the ground contact responds to
  anything other than wording.

### The live generation stack (checked 2026-08-30 by the Omni Agent)

The literal endpoint, port and model filenames are in the workspace's
ignored `devlog/m-4-generation-test-repository-and-first-matrix-for/task-1/
work.md`. **Copy them verbatim into the workplan text**; they are
deliberately absent from this file (devdocs is a public repository, and p9's
own plan leaked the host and its IP — do not repeat that).

- SwarmUI is now **0.9.8.2** (p9 saw 0.9.7.4); its embedded ComfyUI is
  0.33.1 at `/ComfyBackendDirect/…`. The **standalone ComfyUI answers now**
  — in p9 it refused connections — and reports the same 0.33.1. Either
  backend is fine; the report must say which was used.
- One 48 GB workstation GPU, 128 GB RAM, idle at check time, **shared with
  agforge's production asset runs**. A 14B video generation holds the card
  for minutes.
- ComfyUI exposes ~1630 node types, including `WanAnimateToVideo`,
  `Wan22FunControlToVideo`, `WanFirstLastFrameToVideo`, `FrameInterpolate` /
  `FrameInterpolationModelLoader`, `SDPoseKeypointExtractor`,
  `ControlNetApplyAdvanced`, and the `easy-use` pack. **A node existing does
  not mean its weights are on the box.**
- Weights actually present that matter here: `wan2.2_i2v` / `t2v` 14B fp8
  (high+low noise) with the `lightx2v` 4-step speed LoRAs, LTX-2 19B
  dev/distilled (+ distilled speed LoRA, camera-control LoRA), MiniMax H3
  (+ turbo LoRA), `flux1-kontext-dev` (instruction-guided editing),
  `pixelArtDiffusionXL_spriteShaper`, `pixel-art-xl-v1.1`.
- **No ControlNet weights at all (0 files), and no pose-estimator or RIFE/
  FILM interpolation weights.** So the textbook OpenPose-ControlNet route is
  not runnable today without a download — and OpenPose does not cover
  quadrupeds anyway. Downloading a model to the GPU node is *allowed* this
  phase, but the download cost and the missing-weights risk are legitimate
  reasons to rank a method lower on "present-day certainty".
- Reusable code, not to be re-invented: the matrix runner in
  `gentest-pixelArtDiffusionXL/runner.py` (SwarmUI text2image → 64×64
  nearest-neighbour downscale → ≤32-colour quantise → contact sheet →
  `results.csv` upsert, resumable, skips already-generated cells);
  `agforge`'s `src/agforge/comfy_video.py` (submit an API-format workflow,
  poll `/history`, fetch outputs) and its API-format workflows under
  `agforge/.local/resources/comfywf/`.

### Candidate method families (hints, not a decision)

The point of the phase is that **the run picks the method from web evidence**
and says why. These are only the shapes worth checking, with what makes each
cheap or risky *here*:

1. **Video model → frame extraction → sprite sheet.** i2v (Wan 2.2 or
   LTX-2) animating a p9-style sprite, then downscale/quantise every frame.
   Everything needed is already on the box. Risk: temporal palette flicker
   and pixel-grid drift between frames — exactly what the new instrument has
   to measure.
2. **Keyframe editing → in-betweens.** `flux1-kontext-dev` to produce pose
   variations of one character with identity preserved, or
   `WanFirstLastFrameToVideo` between two drawn extremes. Local, image-rate
   cost.
3. **Pose/skeleton-conditioned generation** (ControlNet, WanAnimate driven
   by a pose video). Highest ceiling, but needs weights that are not here.
4. **Cut-out / skeletal rigging.** AI generates the character *parts*
   (segmented or inpainted limbs), and the animation itself is deterministic
   code or a rig (Godot `Skeleton2D`, Spine/DragonBones formats). The
   animation stops being a model output, which is why its certainty may be
   the highest of the four — and it is the honest test of "skeletal
   animation on the AI-generation premise".

## Step 1 — Framework: workflow subjects and an open-question queue

Harness-side Developer work on `agautolab` (doc change; no restart).

1. `agent/project_pattern.md`: state that a study subject may be a
   **workflow family**, not only a checkpoint or LoRA set — same
   `main/<subject>/{summary.md,tips.md}` shape, same `gentest-<subject>/`
   route. `INDEX.md` already has an empty "Workflow families" section for it.
2. Add the queue to the pattern: **`main/QUESTIONS.md`**, publish-ready like
   the rest of `main/`. One entry per open question: subject, the question,
   why it matters, what evidence would close it, the date it was raised.
   A run **closes** entries by appending the tip that answers them and
   **raises** the new ones it created. Say plainly that each `tips.md`'s
   "Still open" section remains the local narrative, and `QUESTIONS.md` is
   the cross-subject queue a routine reads first.
3. Seed `QUESTIONS.md` from the existing `tips.md` "Still open" list plus
   this phase's animation questions. Doing this by hand as the Developer is
   fine and faster; note it as a Deus Ex Machina handoff line if so.
4. Commit and push `agautolab` and `main`.

Report `report1.md`: the pattern diff, `QUESTIONS.md` as landed, and whether
the queue reads as something a cold run could act on without this plan.

## Step 2 — The routine: one fire = one cycle

Write the standing text in `#front › routine-mediagen` (latest post is the
standing request; `front-routine-mediagen` holds the runs). Shape it as one
cycle per fire:

1. Read `main/INDEX.md` and `main/QUESTIONS.md`; pick **one** question (or
   the subject the Developer named in the fire), and say which and why.
2. **Gather** — web research on that question; paraphrase, cite URLs, no
   long quotations (publication conditions apply from day one).
3. **Invent** — propose the method/matrix that would answer it, with the
   asset requirement it will be judged against stated up front.
4. **Verify** — run it in the subject's `gentest-<subject>/` repository,
   bounded to one task's budget, seed fixed, one axis at a time.
5. **Improve** — append `tips.md` (evidence line + date), update the
   subject's row in `INDEX.md`, close and raise entries in `QUESTIONS.md`,
   commit and push `main` and the gentest repository.

Carry the p9 lessons into the standing text itself: literals survive
verbatim into the workplan; video work gets its own task and one measured
generation before any grid; a metric is a secondary signal beside the image,
never a sort key; cost numbers come from the run JSON.

Report `report2.md`: the standing text, and what it deliberately does *not*
prescribe (the routine should choose its method, not be handed one).

## Step 3 — Fire it once, on animation

The real content of the phase. Fire `routine-mediagen` (a one-shot
`rtschedule` event through the production dispatcher, or `trigger.sh
mediagen` by hand — either is fine, say which) with a Developer note naming
this round's subject: **animation workflows for 2D pixel-art assets on the
AI-generation premise, skeletal vs. sprite-sheet.**

Expect the cycle to produce, across as many missions/tasks as it needs:

- `main/<workflow-subject>/summary.md` — what the two animation families
  are, how each is authored in practice today, which tools consume which
  artefact (sprite sheet vs. skeletal formats such as Spine/DragonBones/
  Godot `Skeleton2D`), where AI generation actually enters each pipeline,
  and what the public reports as working *now* — with URLs. Include what
  could **not** be found out; p9's summaries did this well.
- **A single chosen method, with its reasoning recorded.** "Highest
  present-day certainty" should weigh: can it run on the weights already
  here; does it produce a frame set with a stable silhouette and palette;
  can its output be consumed by a real 2D engine; how many moving parts can
  fail.
- A **new** generation-test repository — `autolab project init-repo
  gentest-<subject>` plus a resumable yaml, recorded in
  `README_PROJECT.md`. Do not extend `gentest-pixelArtDiffusionXL`; that one
  is a checkpoint test and its `localtest.yaml` state is `verified`.
  Reuse its `runner.py` post-processing as a starting point rather than
  rewriting it.
- **The local proof**: one animated asset meeting a stated requirement.
  Suggested, so it extends p9's subject rather than replacing it — *a 64×64
  side-view walking cycle of a four-legged animal, 4–8 frames, looping, one
  shared ≤32-colour palette across all frames, delivered as both per-frame
  PNGs and a sprite sheet, with a consistent silhouette and a flat keyable
  background.* Adjust the frame count or the subject if the chosen method
  argues for it, but state the requirement before generating, not after.
- **A consistency instrument, and its own blind spot.** Single-frame
  quality is not the question here; frame-to-frame stability is. Candidates:
  per-frame palette drift against the sheet's shared palette, silhouette
  area/centroid variance across frames, pixel-grid alignment. Whatever is
  built, test it against a case known to be bad and say where it lies —
  `bg_flatness` was trusted for one run before p9 caught it.
- `tips.md` and the `QUESTIONS.md` updates that close this cycle.

Report `report3.md`: the web survey and the ranking, the method chosen and
why, the repository layout and how the runner is invoked, the matrix and its
per-generation timings, the sprite sheet (committed, or a re-signed URL),
the instrument and its verified blind spot, backend used, frictions (GPU
contention with agforge, model load time, timeouts, failed cells), and cost
numbers from the run JSONs.

## Step 4 — Phase report

`report.md`:

- **Did the cycle close?** Did the routine pick its own question, produce
  evidence, and leave the queue in a state where a second fire has something
  to do — without the Developer hand-writing the matrix? Name every point
  where a human had to step in; those are the next phase's targets.
- Whether `QUESTIONS.md` was actually consumed or merely appended to.
- The animation result: which method won, what it produced, and what an
  agforge-side consumer would need to use it (hand-off material, not action).
- Costs, timings, GPU time consumed; whether one cycle fits a routine fire
  or wants a mission chain.
- Recommend only: a standing cadence for `routine-mediagen`; whether the
  next subject should be video/audio; whether `publish/` deserves its first
  review run.
- Deus Ex Machina interventions, each with its one-line handoff note.

**MUST NOT**: commit host facts, IPs, ports, paths, internal repository
names or credentials into `main/`, `publish/`, or this public `devdocs`
repository; push `publish/` (the Developer pushes it by hand); work around
the permission classifier (stop and report — `localrule.md`).

## Out of scope unless a live run forces it

Feeding results into agforge (new toolsets or agents); a recurring
`routine-mediagen` cadence beyond the fires this phase needs; a `publish/`
review run; renaming `init-localtest`/`paper_id`; any scoring scale for
generation tests; dispatcher or schedule-schema changes; rigging or
importing the result into a real game project.
