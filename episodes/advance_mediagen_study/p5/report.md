# advance_mediagen_study p5 — Phase report

Braindump: `braindump.md`. Plan: `plan.md`. Steps: `report1.md`, `report2.md`.

Two fires. The first produced no video at all and concluded that the hardware
could not run one. The second produced two clips in sixteen minutes on the same
hardware, unchanged. **The difference was two configuration flags that had been
switched on as an obvious precaution before anyone measured anything.**

The braindump's bet — that first+last frame conditioning gives a looping walk
cycle for free — is **won**. The control returned after this report was first
drafted and is decisive: first+last closes at **0.4006×**, first-only at
**1.4388×**, at matched length, seed and everything else.

## What was asked, and what came back

| the plan asked for | state |
|---|---|
| web survey of the method | **done** (fire 1) — 7 external URLs, one 403 recorded as unreachable |
| one timed clip before any matrix | **done** — two, in fact: 476.6 s and 477.8 s |
| frames → 64×64 sprite sheet | **done** |
| duplicate + loop-closure checks | **done** — by both parties; see the independence problem below |
| frame-for-frame against p1's `sheet_locked8.png` | **done** |
| matrix: first+last vs first-only × length | **conditioning axis done and decisive; length axis not established** — both length-39 cells OOM'd |
| `main/` summary, tips, INDEX row, publish | **done** — `main` at `15b9423`, test repository at `34ba91b`, both pushed, `publish/` untouched |

**The phase is closed.** Everything the plan asked for was delivered except the
length axis, and the reason that failed is itself one of the findings.

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

**The keying defect was found and fixed during the fire.** As first extracted
the keying was outside-in only: a flood from the border cleared the surround
but never reached the enclosed pockets under the belly and between the legs,
leaving a solid grey mass *inside* the silhouette in most cells. Replacing the
border flood with a per-frame global colour-tolerance test — safe once the
frames are quantised — cleared them. The delivered sheet is **8 unique RGBA
values**: seven opaque colours plus transparent, binary alpha with no soft
edges, 76.3% transparent, comfortably inside the ≤32 clause.

Worth noting that the fix *lowered* the colour count, from 24 to 7 opaque. The
unkeyed pockets had been spending palette budget on spurious greys. The run's
write-up initially claimed usage had risen to 32/32; the measured sheet says
otherwise, and the number was corrected before publication.

**It loses on one thing.** Pose amplitude is lower — p1's frames throw the
legs further and read more as a run; the video cells are a subtler walk. The
**cast shadow** also survives, still baked along the ground line, and it cannot
be keyed by colour: the shadow and the character outline are the same palette
entry, `(57,36,43)`, so a colour-tolerance test that removes one eats the
other. It is inherited from p1's source still rather than introduced by the
video model, and removing it needs a geometric rule or a pre-quantisation pass.

Judged against p1's own asset requirement — 64×64, side view, four-legged walk,
4–8 frames, looping, one shared ≤32-colour palette, consistent silhouette,
flat keyable background — **the video route meets more of its clauses than the
method that has held the bar since p1**, and it is the first result here with a
measured answer to "does it loop".

That closes `spriteSheetFrames/tips.md`'s oldest open line — *whether a
locked-seed sequence actually loops* — not by answering it for locked-seed
stills, but by producing the first method here that demonstrably does, with a
number.

## The braindump's claim: won

*"First+last is more promising than first-only; it will loop naturally."*

The control returned. Both length-124 cells succeeded, differing only in
whether `last_frame` was wired:

| | **first+last** | **first-only** |
|---|---|---|
| **full-clip loop closure** | **0.4006×** | **1.4388×** |
| mean adjacent-frame distance | 6.057 | 7.892 |
| exact duplicate frames | 0 | 0 |
| measured gait period | 16 | 12 |
| best stride-2 8-frame window | **0.6562×** | 0.8594× |

**A ratio below 1 means the clip's last frame is closer to its first than a
typical single step is to its neighbour.** First+last lands at 0.40; first-only
at 1.44 — it ends *further* from the start than it travels in one frame. A
3.6-fold difference on the one measure this phase exists to produce, and
first+last still wins when first-only is allowed to pick its own best window
(0.66 against 0.86).

Two details that strengthen it. First-only also **drifts more per frame**
(7.892 against 6.057): it is not merely failing to return, it is moving further
from itself throughout. And the **gait period differs**, 16 against 12 — the
end-frame constraint is shaping the motion itself, not just pinning the
endpoint.

The braindump said *"start == end makes the clip a loop for free."* It does.

### The length axis was not established

Both length-39 cells OOM'd, at 45.01 and 45.40 GiB, while both length-124 cells
succeeded. **The short configuration failed and the long one succeeded**, which
is backwards from the obvious expectation and is not a cache artefact: the
cells ran 124, 39, 124, 39, and cell 3 succeeded *after* cell 2 had failed, so
a progressively filling card is ruled out. Two for two each way.

On this model a shorter clip costs more memory than a longer one. The mechanism
is not visible from outside the node and neither party invented one.

**This retro-explains fire 1's second cause.** Every fire-1 generation attempt
was short — 39, then 22, then 39 — so fire 1 was simultaneously carrying the
two bad flags *and* only ever asking for the configuration that reliably fails.
The Developer's successful hand-run differed on both axes at once, which is why
it worked and why neither cause was separable until this matrix ran.

## Costs

| | runs | cost |
|---|---|---|
| fire 1 | 57 | $22.92 |
| fire 2 — autolab | 10 | $11.34 |
| fire 2 — Front | 23 | $5.85 |
| **phase total** | **90** | **$40.11** |

Plus roughly 45 minutes of GPU across five successful and thirteen failed
generations, and one production still-image backend stopped for an hour on a
diagnosis that turned out to be wrong.

Fire 2 produced two clips, four matrix cells, a published subject and the
phase's central finding for about three quarters of fire 1's cost, which
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

The arithmetic settles it rather than leaving it a preference: **7.105 / 6.057
= 1.1731 exactly**, which is the run's reported figure to four decimal places.
Its full-clip numbers also reproduce the Omni Agent's to four decimals
(6.0566, 2.4262, 0.4006, period 16/8, zero duplicates) — same metric, two
implementations, exact agreement. Nothing is broken in either. A competing
theory is in the topic record — that the run computed the window ratio over the
wrong frame set — and it does not survive the arithmetic above.

**The fault is the plan's.** Its own words were *"distance frame N → frame 0 vs.
the mean adjacent-frame distance"*, which does not say which adjacency applies
once frames have been subsampled into a sheet. A metric invented in a plan and
first implemented twice, by two parties, found its own ambiguity on contact.
Neither implementation was wrong; the specification was.

That is also a partial recovery of the independence check written off below.
The numbers reached autolab, and it still did not take them on faith.

## The independence check was mostly lost, and the second writer was a second Omni Agent session

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

One session stopped posting when it noticed the second message and asked the
Developer who was writing, rather than correct-and-counter-correct into a
record nobody could read afterwards.

**That question is now answered: the second writer was another Omni Agent
session on the same account, working the same episode from the same
instructions.** Each held the other's posts to be unaccounted for, because
neither could see the other and both signed as the Developer. Messages 3968,
3984 and 3995 came from the session that also wrote `report1.md`; it drove the
fire to publication while the session that wrote this report had gone quiet
waiting for an answer. Both were doing the job as briefed. Neither was
impersonating anyone.

This is the failure mode worth carrying forward, and it is not "an agent
misbehaved":

- **An account is not an identity.** Every safeguard here — Front's one
  requester, autolab's acceptance gate, the plan's independence check — assumes
  the Developer is one writer. Nothing in the system tests that assumption, and
  nothing can, because the credential is the same file.
- **Concurrency is invisible to everyone downstream.** autolab received a
  coherent stream of instructions that occasionally contradicted itself and had
  no way to attribute the contradiction. Front's contract — relay one principal
  faithfully — is unsatisfiable under these conditions and it never knew.
- **Two sessions finding each other by noticing anomalies in their own record**
  is a slow and unreliable detector. One found it in a message it did not
  write; the other found it in a `git log` while about to overwrite the first's
  committed report — and did overwrite it, restoring from git immediately after.
  No content was lost, but only because the work was committed.

**The generation half of the fire was unaffected** — both clips, all four
matrix cells, both measurements and the re-pad decision are real and stand.
What was lost is the clean version of step 4, and it cannot be recovered in
this fire.

## The publication check verified less than it appeared to

`main/` was cleared for commit on this evidence:

```
$ git grep -nE 'gentest-|report\.md|agforge|…|:8188|/Users/'
$ echo $?
1
```

Exit 1, zero hits, pasted with the command as the standing text requires. It
was still not a check of what was about to be published. **`git grep` searches
tracked files only**, and the entire new `subjects/videoFrameExtraction/`
directory was untracked at that moment — it appears as `?? subjects/videoFrameExtraction/`
in the very `git status` output taken alongside the grep. The new subject, the
whole reason for the commit, was the one thing not scanned.

autolab caught it, re-ran with untracked files included, and found **17 further
hits**: nine `gentest-videoFrameExtraction` repository names, six `report.md`
pointers, an internal IP-and-port, and a source path — plus a Zulip message id
it dropped on the same principle although no pattern matched it. All were
masked before the commit landed, so nothing leaked; `15b9423` is clean, and a
filesystem-wide scan that does not rely on git's index confirms it
independently.

This is the same lesson p4 recorded, one turn further on. p4's was *a redaction
scoped to the leak you already found verifies only that leak*. This round adds:
**a check scoped to tracked files verifies only tracked files** — and a clean
exit code from the wrong scope is more dangerous than no check at all, because
it is pasted as evidence and reads as diligence. The durable form is
`git grep --untracked`, or a plain filesystem walk that never consults the
index.

autolab also flagged, unprompted, that it had committed the extra fixes without
a second explicit sign-off, reading the original "commit and push" as acceptance
of the publication step as a whole. Naming the judgement call rather than
quietly making it is the behaviour the contract wants.

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

## Closed

- **The braindump's claim.** First+last 0.4006× against first-only 1.4388× at
  matched length 124. Won.
- **`spriteSheetFrames`'s oldest open line.** Re-scoped rather than closed: the
  tip records the first loop-closure number any pipeline here has produced, and
  says plainly that it does not answer the same question for a locked-seed
  *still* sequence, because "the last pose handing back to the first" is not the
  same continuous-distance question for independently generated stills. It
  hands over the method — per-lag scan, closure ratio against mean adjacent
  distance — so that question is now answerable by whoever wants it.
- **The keying defect.** Fixed during the fire; interior pockets cleared,
  silhouettes clean, 8 unique RGBA values.
- **Publication.** `main` at `15b9423`, test repository at `34ba91b`, both
  pushed, `publish/` untouched, `localtest.yaml` `verified`.

## Still open

- **SwarmUI is still down**, and restoring it is the one outstanding action
  from this phase. It is agforge's production still-image path and has been off
  since fire 1's disproven squatting hypothesis. Restarting it and re-running
  one cell also answers whether the working configuration survives sharing the
  card with ~5.4 GiB of another process's CUDA context — untested, because every
  successful run had the card to itself. The saved backend configuration is
  `id 0 / comfyui_selfstart / enabled true / AutoRestart true / GPU_ID 0 /
  OverQueue 1 / max_usages 2 / StartScript ../ComfyUI/main.py`.
- **`length` as an axis.** Not established. 124 works, 39 and 22 do not, and
  the short-costs-more mechanism is unexplained.
- **Pose amplitude.** The video walk is subtler than p1's; whether prompt
  wording ("running" vs "trotting", explicit stride language) moves it is
  untested.
- **The cast shadow.** Survives keying because it shares a palette entry with
  the outline. Needs a geometric rule or removal before quantisation.
- **Two Omni Agent sessions on one account.** The immediate confusion is
  resolved and recorded above, but nothing prevents a recurrence, and no agent
  in the system can detect it.
