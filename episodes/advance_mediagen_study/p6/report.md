# advance_mediagen_study p6 — Phase report

Braindump: `braindump.md`. Plan: `plan.md`. Step detail: `report1.md`.

One fire, four tasks, all delivered. The braindump asked for know-how from
repeated generation, and the headline is stronger than the question:
**on this backend at a fixed seed the whole pipeline is byte-identical run to
run — still, all 124 clip frames, and the sprite sheet.** Not "small drift".
Zero.

The phase's second result is about the workflow rather than the models, and
it cost more to learn: **one task needed five attempts, and three of them
failed for the same reason — an agent cannot delegate its own continuation to
a process it spawns.** The supervisor then made the same mistake one level up,
and the mission sat idle for five and a half hours.

## What was asked, and what came back

| the plan asked for | state |
|---|---|
| Omni Agent preflight, five items | **done**, including the optional square clip |
| images through ComfyUI directly, no SwarmUI | **done** — six nodes, works, nothing in the fire touched SwarmUI |
| free the card between models, measured | **done** — 10.1 s of a 447 s run, 2.3 % |
| video at the still's aspect ratio | **done** — 640×640, no OOM, 421–425 s |
| `pipeline.py`, one command, one JSON record | **done** |
| run it twice, same seed, report the delta | **done — the delta is zero bytes** |
| one axis, two cells, pictures first | **done** — pose amplitude by prompt wording |
| tips, INDEX, untracked-safe self-check, push | **done** — `main` `31ed8a2`, test repo `422eecc`, `publish/` untouched |

**The phase is closed.** Everything the plan asked for was delivered.

## The braindump's questions, answered with numbers

**"First, the environment must repeat a generation stably."** It does, exactly.

| | run 1 | run 2 |
|---|---|---|
| mean adjacent distance | 13.1599 | 13.1599 |
| full-clip closure ratio | 0.2707 | 0.2707 |
| measured period / chosen stride | 9 / 1 | 9 / 1 |
| 124 clip frames | — | **124/124 byte-identical** |
| still, 64×64 downscale, padded still, sheet | — | **byte-identical** |

The runs are ~25 minutes apart with a `POST /free` and a full model unload
between them. The still is *also* byte-identical to the one the Omni Agent
rendered by hand hours earlier from an independently written script — so the
determinism belongs to the backend and the seed, not to one program. Both
SDXL and MiniMax H3 are exactly reproducible here, and freeing the card does
not perturb either.

**"Image and video models probably don't fit together; free the VRAM even if
slower."** Correct on the first half, and the second half costs nothing. SDXL
sits at **6.63 GiB** resident; the MiniMax run leaves ~34 GiB in use on a
47.26 GiB card. The two frees in a full pipeline run cost **10.1 s of 446.9 s
— 2.3 %**. `POST /free` returns HTTP 200 *before* the memory is released and
settles in under 5 s; the plan's 15 s budget is three times what is needed.
*"Even if slower"* turns out not to be a trade-off worth deliberating.

**"Do the video at the same aspect ratio as the still."** Done, and it works:
640×640 at length 124, 421–425 s per clip, no OOM, 0 duplicate frames. It also
removes a whole bug class — p5's worst error was a resolution literal computed
in someone's head and never measured; with `ResolutionSelector` dropped and
`width`/`height` set to 640 directly there is no derived number to get wrong.

Loop closure came out **0.2707** against p5's 0.4006, but **that is not a
controlled comparison of aspect ratio** and should not be quoted as one: at
640×640 the dog fills the frame where at 864×480 it was letterboxed, so the
mean adjacent distance more than doubled and the ratio's denominator grew with
it. What is established is that the square clip loops at least as well, which
is all the phase needed.

**"Images through ComfyUI, SwarmUI buys little now."** Confirmed. The still is
six nodes on the same `POST /prompt` → `/history` → `/view` path the video
already used. No SwarmUI-specific machinery was needed anywhere.

One caution found in preflight and worth keeping: **the ComfyUI still is not
byte-identical to p1's SwarmUI still** at the same checkpoint, prompt, seed,
steps and cfg — mean absolute difference 0.125, max 78. The same picture, a
different file. Anything comparing a p6 artefact to a p1/p5 one by hash will
be misled.

**"The Omni Agent must check all this before the fire."** It did, and the
preflight paid for itself three times over — see below.

## The axis: pose amplitude by prompt wording

The run chose it from the three candidates, with a stated reason: cheapest
(cell A was task 2's already-verified run, reused from disk, so the axis cost
one clip rather than two) and the deliverable's weakest point, since p5's walk
read subtler than p1's.

| | A — "walking, contact pose" | B — "running, trotting, extended stride" |
|---|---|---|
| mean adjacent distance | 13.1599 | 15.4608 (+17 %) |
| measured period | 9 | 12 |
| chosen stride | 1 | 2 |
| **8-frame window closure** | 0.9096 | **0.6308** |
| unique RGBA in sheet | 8 | 8 |

`fire1_axis_sheets.png` beside this file, A above B. **The wording does real
work**: B throws the front leg far forward and the hind leg far back, with an
airborne phase in several cells, where A is a tight low walk. It is visible
before any number is read, which is how it was checked.

**The result the axis was not run to find:** B is also the better deliverable
*on the metric* — its sheet closes a third tighter, at the same colour count.
"The more energetic prompt also loops better" is not a trade-off anyone
predicted.

Left deliberately unexplained: the period **lengthened** 9 → 12 under
"running" wording, the opposite of the naive expectation. It is recorded as
counterintuitive and picture-checked, with no mechanism attached. p5's lesson
was that a confident mechanism invented at this point is what the next phase
pays for.

## The stride rule, and a specification failure that was the plan's

p5 extracted its sheet at "the best stride-2 window". p6's fire told the run
that stride 2 was not a constant and must follow the measured period — and
**did not say what the rule was.** So the run invented a reasonable one: sweep
every stride and keep the lowest closure ratio.

On the Omni Agent's own clip that rule picks **stride 5**, which advances
**0.56 of a gait cycle per sheet frame** against an aliasing limit of 0.5. It
clears the limit by 0.06 with no margin; move the period by one frame and the
same rule returns a sheet that reads backwards or stands still — p5's "every
sample at the same phase" failure, restated.

**The metric cannot see this by construction.** Closure ratio is the wrap
distance over the window's *own* mean adjacent distance, so a larger stride
inflates the denominator and the ratio improves for reasons unrelated to
whether the sheet animates. `stride_rule_evidence.png` shows the stride-1 and
stride-5 windows together.

The rule, stated properly: **stride ≈ round(period / 8)**, so eight frames span
about one gait cycle; the closure ratio chooses only the window *start*.

Two things make this more than a bug fix:

- **p5's "stride 2" was never a constant.** p5's period was 16 and 16/8 = 2. It
  had been period/8 all along, nobody wrote the rule down, and it travelled
  onward as a magic number.
- **The rule then validated itself on an independent period.** The axis's cell
  B measured period 12; round(12/8) = 2, so it chose stride 2 — p5's number,
  reached by rule rather than inheritance, on a clip the rule was not derived
  from.

This is the same shape as p5's loop-closure ambiguity: **the specification was
at fault, not the implementation.** A rule given as three measured numbers
instead of a rule is a rule the receiver must invent.

## A metric ranked its blind spot first, twice

Related and worth separating from the above, because it happened a second time
in a different tool. `analyze_loop.py` picked the gait period as the
*global minimum* of its lag scan. On a clean periodic signal that minimum
drifts to a harmonic: the square clip's scan bottoms at lags 9, 18 and 27
scoring 1.146 / 1.115 / 1.099, so the tool called the period **27** when it is
**9**. A longer lag averages fewer pairs and is cheaper to minimise.

Fixed to take the smallest lag within a whisker of the minimum. Both defects —
the period picker and the stride sweep — are the same failure: **a score
optimised without a constraint the score cannot express.**

## Five attempts to finish one task, and the finding underneath

Task 2 took five attempts. They were not the same failure and the tally
matters:

- **Three orphanings.** The run launched `pipeline.py`, posted *"I'll wait for
  the Monitor's notification"*, and its serving ended — taking the pipeline
  process with it. One cost ~7 GPU minutes and produced only a still.
- **One crash** — a frame-numbering bug in `build_sheet`, found and fixed by
  the run itself.
- **One false belief.** The last attempt used a harness-tracked background
  task and reported *"a reply here is not required to resume me — the
  background task itself will trigger my next turn."*

**That belief is the finding, and it generalises past this mission.** An
in-system agent's serving is one-shot: **nothing resumes it but a post into a
topic it watches, so it cannot delegate its own continuation to a process it
spawns.** Every stall landed in the same place for this one reason. The
working shape is to block inside the run — one clip is ~450 s against a 1200 s
work timeout, which is what "one clip per task" was for — or to end the run
saying plainly that a post is required to resume it.

**Then the supervisor made the same mistake.** After task 2 was signed off,
Front reported *"I'll keep watching `workrun-task3-m-39` and report back once
autolab picks an axis and starts."* It could not keep watching; its run ended
with that sentence. autolab had correctly said "task 3 is clear to start", and
it was — and nobody started it. **The mission sat idle for five and a half
hours**, and the party that made the second instance was the one whose job was
to prevent the first.

The durable phrasing, which now sits in the fire's own vocabulary: **"I will
watch and report back later" is not a report, it is the end of the work.**
Either do the thing inside the run, or post what someone else needs in order
to act.

Nothing in the system detects this. A stalled mission and a working one look
identical from every topic.

## What the run did well

- **It caught its own bad plan.** Its first plan was written against a
  truncated relay; it noticed and rewrote before the supervisor raised either
  of the two objections it had prepared.
- **It reported a number it could not measure as unmeasured.** Run 1's stage
  timers died with the crashed process. Rather than invent plausible timings,
  it reconstructed bounds from file mtimes and labelled them approximations,
  naming what was lost. That would have been easy and undetectable to fake.
- **It preferred the picture to the score.** The delivered sheet scores 0.9096,
  a poor number by the metric that chose it. The run opened it, said it was a
  genuine walk cycle, and reported the picture as decisive — which was the
  instruction, and which the Omni Agent independently confirmed at 4×.
- **Its untracked-safe publication scan found four leaks that predated the
  task** and had nothing to do with its own edits.

## Front

Two clear improvements on p5, and one repeat of p5's own diagnosis.

**It refused to relay a partial literal.** The fire post was truncated in its
chatlog mid-sentence; instead of passing on what it had, it stopped and asked
for the tail. That is the exact inverse of p5's failure, where a missing
paragraph was replaced with a confident instruction of its own. Asked for, and
got, the right behaviour — and the fire's opening paragraph had asked for it
explicitly, naming p5's substitution.

**It relayed a long payload faithfully.** The six-node graph JSON, the base
URL, five model filenames, the measured readings, the stride table and both
prohibitions all arrived intact, split over five posts. Verified word for word
against the source.

**It held two correct objections rather than posting "start" over them** — and
the run had already fixed both before it posted, which is the better outcome.

Against that: the "I'll keep watching" stall above, which is the mission's
most expensive single event in wall time.

## Publication

`main` at **`31ed8a2`**, test repository `gentest-videoLoopPipeline` at
**`422eecc`**, both pushed and in sync with origin. `publish/` untouched.

Publication was verified twice, independently: the run's `git grep --untracked`
and the Omni Agent's plain filesystem walk that never consults git's index.
Both clean on the two published files — no addresses, ports, local paths,
internal repository names, agent names, phase labels or `report.md` pointers.

**One inconsistency found, deliberately not fixed here.** An older subject page
carries 17 internal phase labels from a previous cycle. The run stripped that
class of label from its own two files this task, which is right, and the
catalogue is now inconsistent as a result. That is a finding about the rule
rather than a defect in the work: **the publication convention has only ever
been applied per-file by whoever last touched a file, so it has never been
applied to the corpus.** Either the labels are a leak everywhere and the whole
catalogue needs one pass, or they are acceptable and this task over-redacted.
That is a Developer decision and is listed as open below.

Model filenames throughout the catalogue are **not** a leak and stay: public
model names are its subject matter and `INDEX.md` has a column for them. The
prohibition is on host literals — addresses, ports, paths.

## Costs

| | runs | cost |
|---|---|---|
| autolab supercoder | 36 | $29.63 |
| autolab superdirector | 9 | $2.30 |
| Front | 87 | $22.87 |
| **total** | **132** | **$54.80** |

Plus roughly **36 GPU minutes** across five clips — three delivered (task 2's
pair and the axis cell), one preflight, one orphaned — and about a minute of
stills.

Front's 87 runs against autolab's 45 is the number to look at. A large share
of them were the stall-and-resume cycle: every orphaning cost a Front run to
notice, a Developer post to diagnose and another Front run to relay.

## Deus Ex Machina

- **The preflight itself** — the braindump asked for it explicitly, so it is
  not a handoff candidate. It paid three times: it proved the six-node still
  graph before the fire depended on it, it found the cache-hit trap that would
  have silently faked the repeatability test, and it ran the square clip so the
  fire inherited the aspect ratio as a result rather than a risk.
- **Diagnosed the stride rule and proved it on the Omni Agent's own clip**
  before the fire spent GPU on an aliased sheet. Partial handoff candidate: the
  rule is in the code now, but nothing except re-deriving the metric's blind
  spot would have caught it.
- **Read the card and the disk directly at every stall**, so each resume
  instruction carried true state — including "run 1's GPU work is already on
  disk, do not regenerate it", which saved a second wasted clip. Not a handoff
  candidate; this is the disinterested-eyes role.
- **Unstalled the mission four times, and started task 3 that nobody had
  started.** **Handoff candidate, urgently.** A supervisor that checked whether
  its agent was actually alive would have caught every one of these, and Front
  had no reason to look. This is the clearest workflow gap the phase found.
- **Opened every picture** — the preflight still, the square clip's contact
  sheet and gait window, the delivered sheet at 4×, and both axis sheets
  side by side. Not a handoff candidate.

## Closed

- **Repeatability.** Byte-identical across every stage, twice, with a card
  free between. The braindump's first requirement, answered harder than asked.
- **The VRAM question.** 2.3 % of wall time. Free the card; it is not a
  trade-off.
- **Aspect ratio.** Square works, and removes the derived-resolution bug class.
- **Images without SwarmUI.** Six nodes, no SwarmUI anywhere in the pipeline.
- **The stride rule.** period/8, stated, and independently validated at
  period 12.
- **Publication.** Both repositories pushed, verified by two independent
  methods.

## Still open

- **agforge's production still-image path is broken** by the backend switch:
  SwarmUI's `GenerateText2Image` fails on a missing node type, because a
  `comfyui_api` backend needs SwarmUI's own custom nodes installed in the
  standalone ComfyUI, which has none of them. The models are fine, and this
  phase proved the cheaper repair works — a plain six-node ComfyUI graph
  renders the same asset with no SwarmUI in the path. **Install the nodes or
  move agforge off SwarmUI: a Developer decision.**
- **Nothing detects a stalled mission.** The single most expensive failure here
  was silence, twice, and no agent in the system can see it.
- **The publication convention has never been applied to the corpus**, only
  per-file. One older subject page is now inconsistent with the rest.
- **Two inherited defects the keying cannot reach.** The cast shadow shares a
  palette entry with the outline; and the background's faint grey ghost shapes
  are structure rather than codec noise, so a corner-sampled colour-tolerance
  key leaves them in the delivered cells. Both survive into the sheet.
- **The period lengthening under "running" wording** (9 → 12) is recorded and
  unexplained, deliberately.
- **Fire 2 was not fired.** The plan made it conditional on fire 1 leaving one
  clear lever; fire 1 answered its own questions and the ranked candidates
  (the pixel-animate LoRA, a different end frame, the same pipeline on Wan 2.2
  FLF2V) are all new experiments rather than a lever this fire exposed.
