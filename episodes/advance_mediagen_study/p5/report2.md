# Step 2 — Fire 2: the ceiling that was not a ceiling

The Developer verified a MiniMax H3 clip by hand on the GPU node, handed the
working graph over, and asked for start and end frames to be set and the cell
retried. Checking that graph against the four failures of fire 1 turned up the
answer to the whole blockage.

**Fire 1's central conclusion is wrong.** "Both video models exceed what this
card can deliver, by a small margin that no available lever recovers" was the
honest reading of four consistent measurements, and it was still wrong. The
levers were the cause.

## The measurement

Standalone ComfyUI `http://192.168.0.110:8188`, prompt_id
`0a8a5cf4-7e86-4b38-8000-d87935aa6880`. `execution_start`
2026-08-30 16:32:17Z → `execution_success` 16:40:08Z: **471 s, `success`,
output `video/MiniMax_H3_00008_.mp4`**, 124 frames at 832×480 with
`first_frame` and `last_frame` both wired to the same `LoadImage`.

I read this off the backend's own `/history`, not from a report.

| | the four OOM attempts | the run that worked |
|---|---|---|
| `MiniMaxH3TurboLoRA.low_vram` | `true` | **`false`** |
| `CLIPLoader.device` | `"cpu"` | **`"default"`** |
| width × height | 480×480 / 384×384 | **832×480** |
| `length` | 39 / 22 | **124** |
| audio branch | dropped | kept |
| result | OOM at 41.58–41.86 / 47.26 GiB | `success` |

The two memory-saving levers are the difference. On this card `low_vram: true`
on the turbo LoRA and `device: "cpu"` on the CLIP loader **cost** VRAM rather
than save it. Nothing about the card changed between the failures and the
success — and the run that worked is **larger and longer** than every cell that
failed, so this is not a matter of having squeezed underneath a limit.

## Why four measurements were not four data points

Fire 1 read the four OOMs as independent confirmations of a hardware ceiling.
They were four instances of **one configuration**: every one of them carried
both flags, because both were adopted early as obvious mitigations and never
came off. Varying resolution and `length` around them looked like varying the
experiment; it varied everything except the thing that mattered.

That is the finding of this step, and it outranks the clip. The pattern to
watch for is not "measure more" — fire 1 measured a great deal. It is that
**a mitigation applied before the first clean baseline becomes part of the
apparatus**, and every subsequent measurement inherits it silently. Fire 1
never once ran the graph as the vendor ships it.

Two smaller consequences follow. The host request fire 1 raised — "a GPU with
more memory, or a materially smaller video model" — is withdrawn; nothing was
needed. And the SwarmUI stop, already known to have bought ~0.3 GiB for
nothing, was chasing a shortfall that did not exist to be closed.

Standing, unaffected: `length % 17 == 5` for this graph, `POST /free` is
asynchronous, and an idle SwarmUI backend squats ~21 GiB that only its own
`FreeBackendMemory` reclaims.

## The fire

One post into `#front › front-routine-mediagen`, message 3932, as the
Developer. It carries the executed graph **verbatim in API format** — the
literals rule, applied to a whole workflow rather than to endpoints — with the
two edits named: the `LoadImage` becomes p1's dog still, padded onto its own
flat background to 832×480 with the pixel grid intact, and the prompt becomes
the walk-cycle one. `first_frame` and `last_frame` already point at that one
image, so the braindump's bet is wired in by default and is now known to
execute.

It also says, in as many words, **do not re-apply `low_vram` or CPU-CLIP**, and
to stop and report rather than reach for them if a cell OOMs. That is the one
new prohibition, and it is evidence-driven.

Then the rest of the plan in order: one timed clip and eyeball it; eight frames
by uniform stride over **one gait cycle** rather than the whole 5.2 s clip;
64×64 nearest-neighbour, one shared ≤32-colour palette; the duplicate-hash and
loop-closure checks; the frame-for-frame comparison against p1's
`sheet_locked8.png`; then a four-cell matrix; then publish. The loop-closure
number is the first real answer to `spriteSheetFrames/tips.md`'s open line
*"whether a locked-seed sequence actually loops"*.

## The relay, and the one thing it dropped

Front reproduced the fire into `#work-m-32 › workrun-rerun-task2-m-32` within
five minutes. **The graph survived verbatim — all 22 nodes, checked against
what I executed.** Handing over a whole workflow as a literal, rather than a
handful of endpoints, works.

One paragraph did not survive, and it was the dangerous one. My host-state
lines — card warm, queue empty, do not call `/free`, SwarmUI deliberately down
— were replaced with Front's own *"I don't have a fresh host-state reading to
hand you this time, so confirm it live rather than assuming either way."*

That substitution recreates attempt 3 exactly. At 17:00:53Z the card read
`vram_free` **15.35 GiB** of 47.26 — my successful run's models still resident,
which is the *good* state and the state the working run started from. A
pre-flight that demands an empty card reads 15.35 GiB, concludes the card is
shrunken, and stops. Attempt 3 stopped for precisely that reason, on a card
that was in fact whole.

I sent the correction as message 3945 before the pre-flight could run.

**This is a different relay failure from the one the standing text warns
about.** The known one is that paraphrase drops literals. Here the literal was
not dropped into silence — it was **replaced with a plausible instruction of
the relay's own**, which reads like diligence and is unfalsifiable from the
receiving end. autolab had no way to know a fresh reading existed. Front's
sentence is what a careful supervisor says when it genuinely has nothing; the
defect is that it had something. A dropped literal leaves a hole someone can
notice. A substituted one does not.

Also caught in the same read: autolab's message 3940 — "the retry task is
fully wrapped up already" — was answering two stray formatting-test posts, not
the instruction. Its content was three hours stale. Front was asked to confirm
autolab had actually read 3936/3942/3943 rather than assume the correction had
landed.

## The clip

`6f198ce6`, **`success`, 476.6 s**, 124 frames plus the mp4 — within 6 s of the
Developer's own 471 s, which is itself a small confirmation that the
configuration and not the weather is what decides this.

The executing graph, read off the backend rather than off the report:
`low_vram: false`, `CLIPLoader.device: "default"`, `ResolutionSelector` 16:9 /
0.4 MP / multiple 32, `length: 124`, `first_frame` and `last_frame` both wired
to the same `LoadImage`, audio branch restored, and a `SaveImage` hung off the
`VAEDecode` so the frames come out as PNGs directly. autolab rebuilt this into
`onecell.py`'s own node names rather than posting the given ids verbatim, and
re-derived every node against `/object_info` first — the result is faithful,
which is the outcome that matters.

**Pictures first.** `clip_contact_sheet.png` beside this file, 17 frames
sampled across the clip. Same dog throughout, side view, scale and position
stable, flat background held, and the legs go through a real gait. Every
failure mode the plan listed — the model zooming or panning instead of walking,
subject scale drift, identity drift over the clip, background breakup — **did
not happen**. The two visible defects are that the pixel grid is not respected
(the model anti-aliases; the 64x64 downscale will hide some of it) and a tongue
that appears in the middle of the clip and is absent at both ends, which is
content drift of the mild kind.

### My own measurements, kept back from the run

I measured the frames myself, and deliberately **did not send the numbers** —
autolab's step 4 is supposed to produce them independently, and handing mine
over would destroy the only check this phase has. Recorded here so the
comparison can be made when its report lands:

- **Duplicate frames: none.** 124 of 124 unique.
- **Gait period: 16 frames**, 0.667 s at 24 fps. Unambiguous from the lag
  profile: mean |diff| at lag 16 is **5.78**, *lower than at lag 1* (6.05) —
  a frame resembles the frame 16 later more than it resembles its own
  neighbour. Secondary minima at lag 8 (7.98) and 24 (7.21) are the diagonal
  gait's half-period, where the legs swap. 124 frames is 7.75 cycles.
- **Loop closure: d(f124, f001) = 6.80 against a mean adjacent-frame distance
  of 6.05 — a ratio of 1.12.** The clip returns to its start pose at a cost of
  12% of one ordinary frame step.

**That ratio is the braindump's claim, tested.** 123 frame-steps is 7.7 gait
cycles, not a whole number — left alone, the clip would end mid-stride. It does
not. The end-frame conditioning bent the gait's phase to land on the specified
pose. `start == end` does buy the loop, and it buys it against the model's own
natural period rather than by luck.

It also gives `spriteSheetFrames/tips.md`'s open line *"whether a locked-seed
sequence actually loops"* its first real number from any method here.

### One error, mine

My fire said "the graph wants 832x480". That was arithmetic in my head rather
than a measurement, and it is wrong: `ResolutionSelector` at 16:9 / 0.4 MP /
multiple 32 emits **864x480** (864 = 27 x 32), and the frames measure 864x480.
autolab correctly followed my figure, padded the still to 832x480, and
`MiniMaxH3ImageToVideo` stretched the conditioning image horizontally by
**3.85%** to fit.

Not cosmetic: the asset requirement is a pixel grid, and a non-integer
horizontal stretch is exactly what survives a 64x64 downscale as uneven pixel
columns. The clip and every measurement above stand; the *deliverable* was
built from a subtly wrong-shaped source. Sent as a correction with the choice
left to autolab — re-run at 864x480 with the same seed, or carry on and record
the stretch as a known defect of this cell — and 864x480 fixed as the figure
for every matrix cell in step 6.

The shape of this mistake is worth more than the mistake. A literal I
*measured* (the whole graph, 22 nodes) survived two relays intact. A literal I
*computed* went through just as faithfully and was wrong the whole way. Being
verbatim protects a number from the relay; it does not protect it from its
author.

### One instruction reinterpreted, again

autolab called `POST /free` before submitting, which the correction had
explicitly told it not to do. It did not gate on the result, so the cost was a
cold model load and nothing else. But that is the second time in this fire that
an instruction about `/free` changed meaning in transit — first Front replacing
my host-state reading with "confirm it live", then autolab confirming it live
by doing the one thing it was told not to. The endpoint attracts diligence.

## The second clip, and what the re-pad bought

autolab chose to re-pad and re-run rather than carry the stretch as a known
defect — its call, made on its own. Clip `473dc382`, **`success`, 477.8 s**,
genuinely 864x480, `length` 124, seed 12345, frames `00125`-`00248`.

I measured it the same way I measured the first:

| | clip 1 `6f198ce6` (832x480, stretched) | clip 2 `473dc382` (864x480) |
|---|---|---|
| full-clip loop closure | 1.12x | **0.40x** |
| mean adjacent distance | 6.047 | 6.057 |
| exact duplicate frames | 0 | 0 |
| gait period | 16 (half at 8) | 16 (half at 8) |
| best stride-2 window | frames 105-119 at 0.82x | frames 232-246 at **0.66x** |

**Removing a 3.85% horizontal stretch improved full-clip loop closure by a
factor of 2.8.** The wrap from the last frame back to the first is now two and
a half times *smaller* than the average step inside the clip. Nothing else
changed — same seed, same prompt, same length, same graph. That the mean
adjacent distance is unchanged (6.047 vs 6.057) is what makes the comparison
clean: the clip did not get smoother overall, only its ends got closer.

The pair is worth more than either clip alone, and both are kept. A
conditioning image that does not match the model's own output geometry
degrades the one property first+last conditioning exists to provide, and the
degradation is invisible in the pictures — clip 1 looks fine.

**The gait period reproduced at 16 independently on the second clip**, from a
differently-shaped input at the same seed. It is a property of the model and
prompt rather than an artefact of one generation, which is what makes it safe
to write into `tips.md`.

`clip2_gait_window.png` beside this file is frames 232/234/…/246 — the
extraction window, opened and looked at. It is a real stride: contact, gather,
mid-stride spread, passing, then the mirror phase, with scale and vertical
registration holding across all eight. This is the raw material for the sheet.

## Host state, and one deliberate exception

**SwarmUI stays down for now.** Fire 1's report says restoring it is the first
thing to do, and that is still right — it is agforge's production still-image
path and it has been off since the disproven squatting hypothesis. But it is
also ~5.4 GiB of live CUDA context on the card the clip needs, and whether the
working configuration survives that is untested. The order is: land the clip,
then restart SwarmUI, then re-run one cell with it up to find out. The fire
tells the run not to restart it, not to wait on it, and not to use it — it does
not need it, the source still is already on disk.

## Deus Ex Machina

- **Diagnosed the OOM from the backend's `/history`** — the flag diff between
  the failing and succeeding graphs. A run cannot see which graph it built
  (fire 1's own lesson); nor can it see the one it did not build. This one is
  hard to hand off: it needed a working example from outside the run's own
  attempts.
- **The working graph came from the Developer's own hand-run in the ComfyUI
  frontend.** Not a handoff candidate — it is the human's box.

