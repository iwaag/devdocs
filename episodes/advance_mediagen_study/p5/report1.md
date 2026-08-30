# Step 1 — Fire 1: the video-model route, and the card it could not get

One fire of `routine-mediagen`, 2026-08-30 14:30Z → 15:33Z. **63 minutes
wall clock, 31 paid runs, $11.54.** Mission **M-32**, four tasks. Task 1
completed and committed; task 2 ended `waiting_external`; tasks 3 and 4 never
started.

**No clip was ever produced.** The braindump's claim — *first+last frame
conditioning beats first-frame-only, because start == end makes the loop
free* — is **untested**. Not disproven, not weakly supported: untested,
because eleven prompts across two video models produced ten out-of-memory
errors and one hard error, and never one frame of video.

What the fire produced instead is a diagnosis of the GPU node, to the
gigabyte, and it is worth more than the matrix would have been.

## The fire

One post into `#front › front-routine-mediagen` (message 3665), as the
Developer. Subject `videoFrameExtraction`, new workflow-family subject, new
repository `gentest-videoFrameExtraction`.

**Every host literal survived the relay verbatim — 22 of 22.** I checked
each one against the workplan topic after Front reproduced the fire:
endpoint, both Wan diffusion filenames, both speed LoRAs, text encoder, VAE,
the workflow URL, the source-still literals, `WORK_TIMEOUT_SECONDS`, all five
survey URLs, the built-in template to avoid, and the two API details. That is
a direct counter-example to this routine's own standing lesson that literals
do not survive paraphrase. The lesson is not wrong, but it is not a law: the
fire said out loud which lines were literals, and they came through.

## The scope cut, and who caught it

autolab's first plan **cut the mission to the survey alone**, deferring
generation, the matrix, extraction and comparison to "a separate future
mission", reasoning from the project's own "one investigation, one
repository" rule.

That rule decides *which* repository work goes in. It has never been a rule
about deferring generation to a later mission. **Front found the mismatch
itself, named it against the fire text, and stopped without acting** —
it asked whether to push back rather than either obeying or overruling.
I sent the correction and the mission was rebuilt as four tasks: survey,
one timed clip, matrix, publish.

This is the first of three times this fire that Front did the supervision
job correctly by *not* doing the run's job.

## Task 1 — the survey, and the instruction it killed

Committed at `6fcc904`. All four survey questions answered, the 403 on one
source recorded as unreachable rather than guessed at, and question 4
reported as *not published by either tool* instead of filled in with
something plausible.

**Breadth, as information rather than a defect: seven external URLs, only
one of which I had not already handed over.** The comparable survey last
phase read about twenty-five. The question here is narrower and every
sub-question came back answered, so it blocked nothing — but a reader should
not assume this swept the field, and task 4 was told to say so.

I spot-checked the two most decision-relevant citations against their primary
pages rather than accepting the relay. Both exact: the pixel-animate LoRA
card does say **600×370** and **45 frames**; the Civitai sprite-sheet
workflow does say **"Use a 480 X 480 input image"**, downscaled through
**Nearest-Exact** to **64×64**.

**Then the survey killed one of my own instructions.** I had told the run to
upscale the 64×64 still to 512×512 for the video model. Every source found
does the opposite — generate large, downscale after. Checking the p1 matrix
settled it: the stills are generated at **1024×1024**, and the 64×64 sprite
is a downscale of that render. The p1 dog was never a 64×64 image. Feeding
an upscaled sprite would have handed the video model a blurred reconstruction
of an image the box regenerates in about eight seconds.

The first cell became: regenerate at 1024×1024 with the unchanged
`matrix.smoke.json` literals, downscale to 480×480, start = end = that image.
**The survey paid for itself in one finding, against the person who
commissioned it.**

## Task 2 — eleven prompts, no video

### The models

Wan 2.2 14B FLF2V as the official graph ships it, then — on the Developer's
call mid-fire — MiniMax H3. The switch was well aimed and better than the
fire's own premise: the box carries `minimax_h3_fl2va`, a **first-last**
model, and its `MiniMaxH3ImageToVideo` node takes `first_frame` and
`last_frame` as optional inputs. **Both arms of the experiment come out of
one node**, and it is a single UNET, so the two-model memory problem cannot
recur. agforge already drives it with a working graph.

One trap caught before it cost anything: that graph computes `length` so
that **`length % 17 == 5`**. The fire's 33 and 81 are illegal for this model;
the axis became 39 and 90.

### What actually happened

| # | model / config | allocated at failure | wall |
|---|---|---|---|
| 1 | Wan 2.2, two-stage | 37.53 GiB | 113.1 s |
| 2 | MiniMax, default | 45.35 GiB | 119.9 s |
| 3 | MiniMax, `low_vram` | 43.47 GiB | 13.6 s |
| 4 | MiniMax + SageAttention patch | *hard error, not VRAM* | 6.0 s |
| 5 | MiniMax, 384×384 | 43.58 GiB | 12.7 s |
| 6 | MiniMax, CLIP→CPU | 41.58 GiB | 69.0 s |
| 7 | same, after I freed the card | **41.58 GiB** | 68.6 s |
| 8 | collision with 7's models | 42.11 GiB | 2.1 s |
| 9–11 | repeats, and `length` 22 | **41.58 GiB** each | ~70 s each |

Device limit **47.26 GiB** throughout. About **9.4 minutes** of GPU time.

The SageAttention lever was mine and it is dead on this hardware — the card
is a Turing part and the library will not load. The run found that by trying
it; I withdrew it.

### The diagnosis, in the order it had to be found

**First: a dead run leaves its weights on the card.** With both queues
completely empty, only 25 GiB of 47.26 was free — roughly 22 GiB held by the
failed Wan run. ComfyUI's own `/free` returned HTTP 200 on both surfaces and
**reclaimed nothing**, which is the trap: the endpoint succeeds and does
nothing, so the memory looks genuinely in use.

**Second: there are two ComfyUI processes on this GPU.** SwarmUI's backend
is a `comfyui_selfstart` — it launches and owns its own ComfyUI — alongside
the standalone one the task drives. The two surfaces reported **different
free VRAM for the same physical card, seconds apart**. That disagreement is
the tell, and *the run found it first*: it noticed the CUDA-free figure was
identical in every OOM regardless of allocation, chased it, and wrote
"part of that shortfall isn't even visible to the surface we're allowed to
drive." That sentence is why this got solved.

**Third: only SwarmUI's own API reclaims it.** `FreeBackendMemory` took the
card from 25.03 → **46.13 GiB free**, and afterwards the two surfaces finally
agreed. Proof that the disagreement *was* the squatting backend.

**Fourth — and this is the finding: freeing did not help.** Run 7 executed on
that freshly-freed card and died at **41.58 GiB, byte-identical to run 6 on
the dirty card.** The error text reconciles it: `41.58 + 0.078 ≠ 47.26`. About
**5.6 GiB is held by the second process's CUDA context** — the cost of merely
being alive, which no free endpoint returns.

**Fifth: the reprieve expires.** Ten minutes later the backend was back to
holding ~21 GiB, `seconds_since_used` **10873 — idle three hours**, both
queues empty, having served nothing. It re-acquires memory on its own.
Freeing is not a fix; it is a few minutes of grace.

**Sixth: `length` 22 failed identically.** Same 41.58 GiB, same final 147 MiB
request. The OOM happens during **weight loading**, before the sampler
touches the length-dependent latent — so shortening the clip cannot help.
That rules out the last in-scope lever, definitively, and it is the run's own
best piece of work.

**The shortfall is small.** MiniMax H3 needs only slightly more than 41.6 GiB.
It is not that the model is far too big for a 47 GiB card — an idle second
process is standing on just enough of it.

### One correction I had to make to the run

The run's write-up attributed the residual ~20 GiB to *"real, currently-in-use
memory... genuinely shared with agforge's production runs"* — live contention.
It is not. I checked as the report posted: both queues empty, that backend
idle for three hours and one minute. Nothing was competing. The memory came
back **on its own, having served no request**, which is a materially different
and more useful finding than "somebody else is using it."

## Where it stopped

Task 2 closed as **`waiting_external`**, not `failed` — the run's own call,
and the right one: nothing here is broken, it is blocked on a host decision.
Committed at `b758867`, with both free-memory calls wired permanently into
`onecell.py`. The host request raised is **not** "free the memory" —
we now know that does not hold — but **stop the SwarmUI-owned
`comfyui_selfstart` backend so a single process owns the GPU**, with
read-only checks that prove it: no `running` `comfyui_selfstart` in the
backend listing, and the two surfaces reporting the same free VRAM.
On these numbers that should be enough for the planned cell to run.

**That decision is the Developer's and it is genuinely costly** — SwarmUI is
agforge's production still-image path. It is open at the time of writing.

## Deus Ex Machina

Two, both on the host, both mine, both handoff candidates the run should own
in `onecell.py`:

- **Verified the `/free` lever on the GPU node** rather than asking the run to
  prove an endpoint while it was spending clips.
- **Diagnosed the cross-process squat and called SwarmUI's
  `FreeBackendMemory`.** The run could not have found this from inside the
  surface it is allowed to drive — but it got close enough to name the gap,
  which is the part that matters.

There is a third, smaller one: I read the backend's `/history` directly
throughout, so I knew each failure before the report arrived. That is not a
handoff candidate — it is the disinterested-eyes role the plan gives me, and
this fire is the case for it. The run's reports were honest every single time;
I checked because checking is cheap, not because it was wrong.

## What this says about Front

The braindump's instruction was that Front relays and does not verify
artefacts, and this fire is a fair test of it.

- It **caught the scope cut** and asked instead of acting.
- It **caught a routing gap I created** — my revised first cell had gone into
  task 1's topic while task 2's topic still held the pre-revision instruction.
  It re-posted the correction before the run read the wrong one. That saved a
  clip.
- It **relayed the OOM diagnosis verbatim** and caught the run mid-way through
  the wrong lever.
- It said, unprompted, "that's autolab's question to you, not one I judged."

One process defect, cheap: restarting a run that had already submitted put two
prompts on the card at once (run 8, dead in 2.1 s). Check the backend queue
before telling a run to resubmit.

**Front never opened a render and never needed to.** Everything it caught was
a mismatch between what was asked and what was planned, or between where an
instruction was sent and where it would be read — supervision of the
conversation, not of the artefact. That is the division the braindump asked
for, and it worked.

## Cost

| | runs | cost |
|---|---|---|
| autolab (supercoder + superdirector) | 11 | $5.89 |
| Front | 20 | $5.65 |
| **total** | **31** | **$11.54** |

Front is 65% of the runs and 49% of the cost, for a fire that produced no
image. Much of that is the supervision loop being woken by acknowledgements.
Worth watching, not yet worth changing — two of its wake-ups paid for
themselves outright.

## Still open

- The claim itself. Untested.
- Whether stopping the second ComfyUI process actually clears the shortfall.
  Predicted yes on these numbers; unverified.
- Whether MiniMax H3's pixel-art LoRA axis is worth anything, which cannot be
  asked until one clip exists.
