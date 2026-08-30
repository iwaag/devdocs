# Step 1 — Fire 1: the video-model route, and the card it could not get

One fire of `routine-mediagen`, 2026-08-30 14:30Z → 16:17Z. Mission **M-32**,
four tasks. Task 1 completed and committed; task 2 ended `waiting_external`;
tasks 3 and 4 never started.

**No clip was ever produced.** The braindump's claim — *first+last frame
conditioning beats first-frame-only, because start == end makes the loop
free* — is **untested**. Not disproven, not weakly supported: untested,
because **thirteen prompts** across two video models produced twelve
out-of-memory errors and one hard error, and never one frame of video.

> ## CORRECTION, 2026-08-30 16:54Z — this report's central conclusion was wrong
>
> The Developer ran MiniMax H3 on that card by hand and **it produced a clip**:
> prompt `0a8a5cf4`, `status: success`, **471.0 s**, 16:32:17Z → 16:40:08Z,
> output `MiniMax_H3_00008_.mp4`, 124 frames at 832x480 with both
> `first_frame` and `last_frame` set. I verified all of that against the
> backend rather than accepting it.
>
> **The two memory-saving levers were causing the OOM.** Every failing attempt
> carried `MiniMaxH3TurboLoRA.low_vram: true` and `CLIPLoader.device: "cpu"`.
> The successful run carries `low_vram: false` and `device: "default"` — and
> is **larger and longer** than every cell that failed (832x480 x 124 frames
> against 480x480 x 39). Nothing about the card changed.
>
> **CLIP-to-CPU was my recommendation**, argued as "the biggest and most
> targeted lever" because a 32B encoder was sitting on the GPU. It was the
> defect. Whatever those two options do inside these nodes, on this card they
> cost VRAM rather than save it.
>
> So this report contains two wrong conclusions of mine, stacked: first that a
> second process's CUDA context was the shortfall, then that the shortfall was
> irreducible. Both were reasoned from real measurements and both were wrong,
> because **every one of those measurements was taken through a defect I had
> introduced.** The text below is kept unedited beside this note.
>
> What survives: the `/free` asynchrony finding, the idle-backend squat, the
> `length % 17 == 5` grid, and that a run cannot see which graph it built.
> What does not: the ceiling, the hardware request, and the reason SwarmUI was
> stopped.

The result *as this fire measured it* was that **both video models exceed what
this card can deliver, by a small margin that no available lever recovers.**
A single ComfyUI process on this 47.26 GiB card reaches about **41.9 GiB** of
usable allocation; Wan 2.2 14B FLF2V and MiniMax H3 both want more.
**For MiniMax this is now known to be false** — see the correction above. For
Wan it still stands on its own evidence (it never ran with those two levers),
but it deserves the same suspicion until retested.

**And I got the central diagnosis wrong.** I attributed the shortfall to a
second ComfyUI process squatting on the card, and on the strength of that the
Developer took down a production service. It bought about 0.3 GiB and changed
nothing. That is recorded in full below rather than tidied away.

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
| 12 | Wan again, **whole card, SwarmUI stopped** | 38.06 GiB | 41.0 s |
| 13 | MiniMax, **whole card**, correct graph | **41.86 GiB** | 74.6 s |

Device limit **47.26 GiB** throughout. About **13 minutes** of GPU time.

Runs 12 and 13 are the honest ones: SwarmUI stopped, card verified whole at
46.4 GiB across three consecutive reads, queue empty. Run 12 was the run
rebuilding the *superseded Wan graph* from prose instead of using the
committed `onecell.py` — caught by reading the executed node list off the
backend (`unet_high`/`unet_low`, two `KSamplerAdvanced`) rather than from its
report. Run 13 is the MiniMax cell as specified.

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

## The part I got wrong

Mid-fire I diagnosed the shortfall as the second ComfyUI process's CUDA
context and predicted that stopping it would return ~5.6 GiB and let the cell
run. The Developer authorised it and stopped SwarmUI — **agforge's production
still-image path** — to open the window.

| | allocated at OOM | unaccounted |
|---|---|---|
| second process alive | 41.58 GiB | ~5.6 GiB |
| second process stopped | **41.86 GiB** | ~5.36 GiB |

**It bought about 0.3 GiB and the cell still failed.** The ~5.4 GiB is
intrinsic to a running ComfyUI process — its own CUDA context and caching
allocator — not memory a second process was holding.

The squatting finding itself is real and stands: an idle backend did hold
~21 GiB, ComfyUI's `/free` did not reclaim it, only SwarmUI's
`FreeBackendMemory` did, and it re-acquired the memory within ten minutes
having served nothing. **But it was never the thing standing between us and a
clip.** It was a real problem that was not *this* problem, and I stacked a
prediction on it that did not survive contact with the measurement.

The cost of that error was a production service taken down for nothing. The
lesson is not "measure more" — every number here was measured. It is that a
correct finding sitting next to an unexplained gap is not a licence to assume
the finding explains the gap.

### A second correction, smaller and more useful

**`POST /free` is asynchronous.** It returns HTTP 200 before the memory is
released, so an immediate read shows the old figure and looks exactly like a
no-op. That is why I had concluded the endpoint did nothing for memory it did
not own — the one time I saw it work, I happened to wait three seconds.

This propagated: the run adopted my too-strong conclusion, read the card once
straight after calling `/free`, saw 21.77 GiB, and correctly stopped rather
than run on a shrunken card. The card was in fact whole. **The stop was right
behaviour on a condition I had underspecified**, and the fix is to poll until
the figure stops rising rather than read once.

## Where it stopped

Task 2 closed as **`waiting_external`**, not `failed` — nothing here is
broken, it is blocked on hardware. `onecell.py` committed at `b758867`.

**The host request has been replaced.** "Stop the second process" was tried,
measured, and did not work. What this actually needs is **a GPU with more
memory, or a materially smaller video model.** Stopping things has been
exhausted.

**SwarmUI must be restarted.** It was taken down for a hypothesis that did not
hold, so agforge's production still-image path is offline for no benefit.
Restoring it is the first thing to do, and the saved backend configuration is
`id 0 / comfyui_selfstart / enabled true / AutoRestart true / GPU_ID 0 /
OverQueue 1 / max_usages 2 / StartScript ../ComfyUI/main.py`.

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
this fire is the case for it. The run's reports were honest every single time.
It earned its keep once for real: run 12 executed the **superseded Wan graph**
while reporting in good faith, and only the executed-node list showed it. A
run cannot see which graph it built; the backend can.

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
| autolab | 20 | $11.02 |
| Front | 37 | $11.90 |
| **total** | **57** | **$22.92** |

Plus about 13 minutes of GPU on a card that had a production service stopped
for it. **57 paid runs and no image.**

Front is 65% of the runs and 52% of the cost. Much of that is the supervision
loop waking on acknowledgements and re-reporting "nothing has changed" — it
needed an explicit stand-down to stop. That is the one clear efficiency defect
of the fire, and it is structural rather than Front's judgement: two of its
wake-ups paid for themselves outright.

## Still open

- The claim itself. Untested, and not testable on this hardware as configured.
- What actually accounts for the ~5.4 GiB a single ComfyUI process cannot use.
  Measured consistently; not explained.
- Whether a materially smaller video model exists on this box that would fit
  inside ~41.9 GiB. Not surveyed — the survey was aimed at method, not at
  memory footprint, because nobody knew memory was the binding constraint.
- Whether MiniMax H3's pixel-art LoRA axis is worth anything, which cannot be
  asked until one clip exists.

## Closed, and worth carrying forward

- **`length % 17 == 5`** for MiniMax H3 on this graph — 33 and 81 are illegal.
- **`/free` is asynchronous**; verify by polling, never by one immediate read.
- **An idle SwarmUI backend squats ~21 GiB and re-acquires it after being
  freed**, and only its own `FreeBackendMemory` reclaims it. True, and not the
  cause of anything here.
- **A run cannot see which graph it built.** Run 12 reported in good faith
  while executing the superseded model; only the backend's executed-node list
  revealed it.
