# advance_mediagen_study p5 — Phase report

Braindump: `braindump.md`. Plan: `plan.md`. Steps: `report1.md`, `report2.md`.

Two fires. The first produced no video at all and concluded that the hardware
could not run one. The second produced two clips in sixteen minutes on the same
hardware, unchanged. **The difference was two configuration flags that had been
switched on as an obvious precaution before anyone measured anything.**

The braindump's bet — that first+last frame conditioning gives a looping walk
cycle for free — is **supported but not yet won**, and the reason it is not yet
won is stated plainly below rather than smoothed over.

## What was asked, and what came back

| the plan asked for | state |
|---|---|
| web survey of the method | **done** (fire 1) — 7 external URLs, one 403 recorded as unreachable |
| one timed clip before any matrix | **done** — two, in fact: 476.6 s and 477.8 s |
| frames → 64×64 sprite sheet | **done**, and read here; the run had not yet reported it when this was written |
| duplicate + loop-closure checks | **done** (by the Omni Agent; see the independence problem below) |
| frame-for-frame against p1's `sheet_locked8.png` | **done** here |
| matrix: first+last vs first-only × length | **running** at the time of writing (4 cells, ~30 min) |
| `main/` summary, tips, INDEX row, publish | **drafted, not committed** — held for review |

The phase is therefore **not closed**. The matrix that settles the braindump's
actual comparison was in flight when this was written, and nothing has been
pushed. Read the verdict below with that in front of it, not behind it.

## The finding that outranks the clip

Fire 1 ran thirteen prompts across two video models, got twelve
out-of-memory errors, and concluded: *"both video models exceed what this card
can deliver, by a small margin that no available lever recovers"*, raising a
host request for a bigger GPU. Every measurement in it was real and correctly
taken.

It was wrong. Every failing attempt carried `low_vram: true` on the turbo LoRA
and `device: "cpu"` on the CLIP loader. The Developer's own hand-run, which
succeeded, carried neither — and was **larger and longer** (864×480 × 124
frames) than every cell that had failed (480×480 × 39). On this card those two
memory-saving flags cost VRAM rather than save it.

**Four consistent measurements were not four data points.** They were four
instances of one configuration. Both flags went on early as obvious
mitigations and never came off, so varying resolution and length around them
looked like varying the experiment while holding the only decisive variable
fixed. The run never once executed the graph as the vendor ships it.

This is the phase's transferable lesson, and it is not "measure more" — fire 1
measured a great deal, honestly, and wrote its numbers down. It is that **a
mitigation applied before the first clean baseline becomes part of the
apparatus.** Nothing downstream can see it, because every subsequent
measurement inherits it. The cheap defence is a single unmodified vendor-default
run before any tuning, kept as the control.

Cost of not having done that: 57 paid runs, thirteen GPU minutes, one
production service taken down, and a hardware request that was never needed.

## The clips

Both from the same graph, same seed 12345, same prompt, differing only in the
conditioning image's shape.

| | clip 1 (832×480) | clip 2 (864×480) |
|---|---|---|
| wall | 476.6 s | 477.8 s |
| frames | 124 | 124 |
| exact duplicate frames | **0** | **0** |
| mean adjacent-frame distance | 6.047 | 6.057 |
| **full-clip loop closure** | **1.12×** | **0.40×** |
| gait period | 16 frames | 16 frames |
| best stride-2 8-frame window | 105–119 at 0.82× | 232–246 at **0.66×** |

`clip_contact_sheet.png` and `clip2_gait_window.png` beside this file.

**The pictures.** Same dog throughout, side view, scale and position stable,
flat background held, legs through a real gait. Every failure mode the plan
predicted — the model zooming or panning instead of walking, subject scale
drift, identity drift across the clip, background breakup — **did not occur**.
The defects are a cast shadow and a lavender-not-white background, both
inherited from p1's source still rather than introduced by the video model,
plus a tongue that appears mid-clip and an anti-aliased edge treatment that
does not respect the pixel grid.

### Two numbers worth keeping

**Gait period 16 frames** (0.667 s at 24 fps), with the half-cycle at 8 where
a quadruped's diagonal pairs swap. Unambiguous: mean frame-to-frame distance at
lag 16 is *lower than at lag 1* — a frame resembles the frame 16 later more
than its own neighbour. It **reproduced on the second clip** from a
differently-shaped input, so it is a property of model and prompt, not of one
generation. This is what makes an 8-frame sheet at stride 2 the right
extraction and a uniform stride over the whole clip the wrong one: 124/8 ≈ 15.4
lands within a whisker of the period, so every sampled frame sits at nearly the
same phase and the sheet reads as static.

**Loop closure 0.40×** on clip 2 — the wrap from last frame to first is two and
a half times *smaller* than the average step inside the clip. 123 frame-steps
is 7.7 gait cycles, not a whole number, so left alone the clip would end
mid-stride. It does not. **The end-frame conditioning bent the gait's phase to
land on the specified pose.**

### What the re-pad bought, and why it matters beyond itself

The Omni Agent's fire said the graph wanted 832×480. That was arithmetic in the
head rather than a measurement; `ResolutionSelector` at 16:9 / 0.4 MP /
multiple 32 emits **864×480**. The run followed the stated figure, so clip 1's
conditioning image was stretched horizontally by 3.85%.

Removing that stretch improved full-clip loop closure **2.8-fold**, with the
mean adjacent distance unchanged (6.047 → 6.057) — the clip did not get
smoother overall, only its ends got closer. **A conditioning image that does
not match the model's own output geometry degrades the one property first+last
conditioning exists to provide, and it is invisible in the pictures**: clip 1
looks fine.

The shape of the error is its own lesson. A literal that was *measured* — the
whole 22-node graph — survived two relays byte-intact. A literal that was
*computed in someone's head* travelled just as faithfully and was wrong the
whole way, and was restated twice more after correction. Being verbatim
protects a number from the relay; it does not protect it from its author.

## The sheet, against p1's

`sheet_vs_p1_locked8.png` beside this file: p1's `sheet_locked8.png` on top,
the video-derived sheet below (magenta shows through where the background was
keyed to transparency). Both 512×64, eight 64×64 cells.

**The video route wins on everything the sprite-sheet route was weakest at.**
p1's eight cells drift: the background is a slightly different grey in each
one, cell 4 carries an orange ground artefact, and cell 6 is a visibly smaller,
differently-proportioned dog. The video-derived cells are one dog at one scale
in one palette, cell to cell, because they are frames of a single continuous
generation rather than eight independent ones. Its background keyed cleanly to
transparency; p1's is opaque and unkeyable as it stands.

**It loses on two things.** Pose amplitude is lower — p1's frames throw the
legs further, and read more as a run; the video cells are a subtler walk.
And the keying left a grey lump attached to the dog in every cell: the cast
shadow plus surrounding pixels that survived the colour key, which reads as a
rock the dog is standing on. That is a defect of this extraction, not of the
method, and it is the first thing a next pass should fix.

Judged against p1's own asset requirement — 64×64, side view, four-legged walk,
4–8 frames, looping, one shared ≤32-colour palette, consistent silhouette,
flat keyable background — **the video route meets more of its clauses than the
method that has held the bar since p1**, and it is the first result here with a
measured answer to "does it loop".

That closes `spriteSheetFrames/tips.md`'s oldest open line — *whether a
locked-seed sequence actually loops* — not by answering it for locked-seed
stills, but by producing the first method here that demonstrably does, with a
number.

## The braindump's claim: supported, not won

*"First+last is more promising than first-only; it will loop naturally."*

**Supported.** A first+last clip closes at 0.40× over its full length and 0.66×
over the best gait window, on a clip whose length is not a whole number of gait
cycles. The conditioning is doing the work.

**Not won**, because **the first-only control had not returned.** Nobody here
has yet seen what first-frame-only conditioning produces on this subject, so
"beats first-only" is still an untested comparison — exactly the shape of
mistake this phase's own headline finding is about. autolab's four-cell matrix
was running when this was written and contains that control; its result decides
the braindump, and nothing before it does.

## Costs

Fire 2 only (fire 1's 57 runs / $22.92 are in `report1.md`), and **partial** —
the mission was still running when this was written.

| | runs | cost |
|---|---|---|
| autolab (supercoder) | 5 | $2.48 |
| Front | 11 | $3.38 |
| **total so far** | **16** | **$5.86** |

Plus ~16 minutes of GPU for two clips. Fire 2 produced two clips, a sheet and
the phase's central finding for about a quarter of fire 1's cost, which
produced no image at all.

## Front

Fire 2 is a second clean test of the braindump's instruction that Front relay
rather than verify artefacts, and the answer is more interesting than fire 1's.

**It relayed a whole 22-node workflow graph verbatim, twice, without damage.**
That is the strongest evidence yet that handing over literals wholesale works.

**But it dropped one paragraph and replaced it with something plausible of its
own.** The host-state lines — card warm, queue empty, do not call `/free`,
SwarmUI deliberately down — became *"I don't have a fresh host-state reading to
hand you this time, so confirm it live rather than assuming either way."* The
card at that moment read 15.35 GiB free of 47.26, which is the *good* state
(the successful run's models resident), and a pre-flight demanding an empty
card would have stopped exactly as attempt 3 stopped. The correction went out
in time.

**This is a different failure from the one the standing text warns about.** The
known one is that paraphrase drops literals, leaving a hole someone can notice.
Here the literal was replaced by a confident instruction that reads like
diligence and is unfalsifiable from the receiving end — autolab had no way to
know a fresh reading existed. A dropped literal is visible; a substituted one
is not.

Related, and twice: instructions about `POST /free` do not survive. Front
turned "here is the reading, do not call it" into "confirm it live", and
autolab then confirmed it live by calling `/free` — the one thing it had been
told not to do. It did not gate on the result, so the cost was a cold model
load. The endpoint attracts diligence, and a prohibition on it reads as
carelessness to everyone downstream.

## autolab

It earned its keep this fire. It re-derived the handed graph against
`/object_info` before running rather than pasting it, caught nothing wrong and
ran it faithfully; when told the resolution figure was wrong it chose, on its
own, to re-pad and re-run rather than record the stretch as a known defect —
the more expensive and correct call, and the one that produced the 0.40×
number. It built its own extraction tooling by reading p1's `runner.py` and
`sequence_runner.py` rather than writing a fourth copy.

## The metric was underspecified, and the run caught it

autolab was handed the Omni Agent's window-closure figure of **0.66×**. It
re-measured rather than accepting it, got **1.173×** for the same eight frames,
and — this is the part that matters — **left the gap on the table as an
unresolved finding instead of quietly reconciling to the number it had been
given.**

Both are right. Same numerator, different denominators:

| | wrap distance | divided by | ratio |
|---|---|---|---|
| Omni Agent | 7.105 | mean step *within the sheet* (stride 2) — 10.828 | **0.656** |
| autolab | 7.105 | mean adjacent distance *of the whole clip* (stride 1) — 6.057 | **1.173** |

For a sprite sheet the first is the meaningful one: the deliverable's frames
are two apart, so "does the wrap read as an ordinary step" has to be asked
against the step the sheet actually shows, not against one the viewer never
sees. Measured on the delivered 64×64 keyed cells themselves the ratio is
**0.608** on RGB and **0.530** with alpha — the deliverable closes tighter than
the full-resolution frames it came from.

**The fault is the plan's.** Its own words were *"distance frame N → frame 0 vs.
the mean adjacent-frame distance"*, which does not say which adjacency applies
once frames have been subsampled into a sheet. A metric invented in a plan and
first implemented twice, by two parties, found its own ambiguity on contact.
Neither implementation was wrong; the specification was.

That is also a partial recovery of the independence check written off below.
The numbers reached autolab, and it still did not take them on faith.

## The independence check was mostly lost, and by whom is unresolved

Step 4 was designed so autolab would measure loop closure and duplicates
independently, with the Omni Agent's numbers held back for comparison. The
Omni Agent did hold them back, and said so in the topic.

They went out anyway. **Three messages were posted to the routine topic as the
Developer that the Omni Agent did not write** (3968, 3984, 3995), each within a
minute of one of its own. They are competent supervision — one correctly warns
against carrying clip 1's frame indices onto clip 2, and correctly attributes
the shadow and background to the source still. Two of them also restate the
superseded 832×480.

Message 3995 settles what was happening: it reports loop closure **0.40×**,
mean adjacent distance **6.057**, and window **232–246 at 0.66×** — the Omni
Agent's values to four significant figures and the exact frame indices,
computed minutes earlier in its session and **never posted anywhere, nor even
reported to the Developer at that point**. It also describes the eight-frame
contact sheet the Omni Agent had just rendered and opened.

Two consequences, both structural:

- **Step 4 was compromised as an independent check**, though not entirely:
  autolab re-derived rather than accepted, and the disagreement it surfaced is
  the section above. What is lost is the clean version of the test — a number
  produced with no knowledge of the Omni Agent's — and that cannot be recovered
  in this fire.
- **Front cannot relay faithfully for a requester that is two writers.** Its
  entire contract is one conversation with one principal; two writers on one
  account, occasionally disagreeing, is invisible to it by construction.

The Omni Agent stopped posting when it noticed the second message and asked the
Developer who was writing, rather than correct-and-counter-correct into a
record nobody could read afterwards. That question is unanswered. **The
generation half of the fire was unaffected** — both clips, both measurements
and the re-pad decision are real and stand.

## Deus Ex Machina

- **Diagnosed the OOM from the backend's execution history** — the flag diff
  between the failing graphs and the succeeding one. Hard to hand off: it
  needed a working example from outside the run's own attempts. A run cannot
  see the graph it did *not* build.
- **Measured gait period, duplicates and loop closure on both clips, and
  opened the frames.** This was meant to be the run's own step 4 and became the
  Omni Agent's; the handoff candidate is the measurement code, which is ten
  lines of numpy and belongs in the test repository.
- **Read `/history` and `/queue` directly throughout**, so each clip's outcome
  and its executing graph were known before any report arrived. Not a handoff
  candidate — this is the disinterested-eyes role, and it caught the graph
  faithfully reproduced under different node names.
- **The working graph came from the Developer's own hand-run.** Not a handoff
  candidate; it is the human's box and the human's tool.

## Still open

- **First-only conditioning.** In the running matrix; unsettled until it
  returns and is read.
- **The keying defect** — the cast shadow survives the colour key as a grey
  lump attached to the sprite in all eight cells.
- **Pose amplitude.** The video walk is subtler than p1's; whether prompt
  wording ("running" vs "trotting", explicit stride language) moves it is
  untested.
- **Whether the working configuration survives SwarmUI being up.** It has been
  down throughout, which is ~5.4 GiB of CUDA context the successful runs never
  had to share. Restarting it and re-running one cell answers this and restores
  the production still-image path, which has been offline since fire 1.
- **`length` as an axis.** Only 124 was ever run.
- **`main/` was not touched.** No `videoFrameExtraction` summary, tips, or
  INDEX row; nothing published.
