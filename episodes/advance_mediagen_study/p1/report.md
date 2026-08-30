# advance_mediagen_study p1 — Phase report

The `mediagen` study now runs as a cycle. One fire on 2026-08-30 picked up
this round's subject, surveyed it, chose a method, proved it locally, and
left the queue with three new questions — in 41 minutes and $7.44, with one
human intervention that is precisely locatable and worth its own next phase.

Steps: `report1.md` (framework), `report2.md` (the routine), `report3.md`
(the fire).

## Did the cycle close?

**Yes, and it closed on its own content.** The routine picked the question,
ran the survey, ranked four candidate method families, chose one, named the
workflow family itself, built the repository, ran the matrix, built an
instrument, found that instrument's blind spot, and wrote everything back.
No Developer wrote a matrix, chose a method, or named a subject.

Two things are worth separating out, because "the cycle closed" could be
read too generously:

- **The subject was named by the Developer, not picked from the queue.** The
  fire said "animation, skeletal vs. sprite-sheet", which the standing text
  explicitly permits. So this fire proved the *gather → invent → verify →
  improve* half of the loop. The *pick your own question* half is still
  untested — that is what a second, bare fire would prove, and the queue now
  has eight open entries for it to choose from.
- **The method choice was genuinely the run's.** The ranking put ControlNet
  / pose-guided rendering third while calling it "the technique with the best
  public evidence for frame-to-frame consistency", because its weights are
  not on the box. A run that was pattern-matching on the literature would
  have picked it and opened a `waiting_external` for a download. This one
  weighed present-day certainty as instructed and picked the boring method
  that runs on what is here. That is the single strongest signal in the
  phase.

### Where a human had to step in

**One place, for eight minutes, and it was a genuine deadlock.**

The first task-3 run launched its 24-generation matrix with
`run_in_background`, posted "I'm waiting for the background generation to
finish, I'll continue once it completes", and ended its turn. A `workrun-`
serving is one-shot: ending the turn ended the run, and killed the
generation process with it. Front then read that post, checked the topic,
and decided — correctly, from a false premise — that posting would "restart
autolab's turn for no reason". Neither side could move.

I broke it by posting **information, not action**, into
`front-routine-mediagen`: that a serving is one-shot, that the post Front
was waiting on was the end of autolab's run, that 19/24 images were on disk
and the runner is resumable, and that the retry must generate in the
foreground. Front restarted task 3 itself. Keeping the intervention to
information is what leaves "can this run unattended?" a real question rather
than a moot one.

**The next phase's targets, both one-line guide edits, no code:**

1. **autolab's supercoder guide has no "waiting happens inside the run"
   line.** agfront's guide learned exactly this in `agent_standardize` p7,
   when Front backgrounded a wait and ended its run at 242 s; two guide lines
   fixed it with no code change. The same lesson never travelled to autolab.
   The supercoder had 1200 s of budget and used 222 s.
2. **Front has no rule for reading "I am waiting" as a stall.** Its heuristic
   is "wait for a result or a question", which is right, and it has nothing
   that says a run which produces neither is finished rather than working.
   "An agent saying it is waiting is not progress" is the missing sentence.

This is p9's failure with the polarity flipped — there a supervisor read
"opened a topic" as "running now"; here it read "waiting in the background"
as "running now". Two rounds, same missing distinction. It is worth fixing at
the level of *what does a supervisor treat as evidence of life*, rather than
patching a third phrasing next round.

A second, non-blocking friction is worth naming because it nearly became
silent data loss: **task 3 reported pushing `gentest-spriteSheetFrames` and
had not.** Task 4 checked with `git ls-remote origin`, found zero refs, and
pushed for real. Front had asked for that double-check on its own initiative,
having noticed task 3's report mentioned only `main`. Good supervision caught
it; nothing structural would have.

## Was `QUESTIONS.md` consumed, or merely appended to?

**Consumed.** Concretely:

- Task 1 read it as its first substantive act, before the web research.
- It **closed** Q6, Q7 and Q8, each with the date and a pointer to the tip
  that answers it — not by deleting them.
- It filled the `spriteSheetFrames` name back into all three subject fields,
  which had been seeded as `-` because the family had no name yet. That was
  the specific weakness `report1.md` predicted would trip a cold run. It did
  not trip it: the run treated naming the family as part of the work and went
  back to update the entries.
- It **raised** Q9, Q10 and Q11 out of its own results — does the cycle
  actually loop; a deliberate same-palette content-drift case for the
  instrument; does the inherited shadow/background defect move frame to
  frame. All three are things this fire genuinely could not answer, not
  filler.

The queue now holds 11 entries, 8 open. A second fire has real work waiting
without anyone writing it.

**A second defect, caught on verification.** `main/spriteSheetFrames/`
named the internal generation-test repository in three places — once in
`tips.md`'s evidence line and twice in `summary.md`. The pattern requires
`main/` to stand on its own when copied out: it names evidence files, never
the repository holding them. Redacted by hand (commit `5a51567`, text only,
no finding changed) and recorded here as a Deus Ex Machina. No host facts,
IPs, ports or credentials were present anywhere in `main/` or in this
episode folder — checked by grep across both.

This is a gap in the routine's step 4 rather than a slip: the standing text
says "commit and push `main`" and never says "and check it against the
publication conditions before you do". The `publish` routine would catch it
later, but `main/` is supposed to be written publish-ready in the first
place. One line in the standing text closes it.

**One defect, and it is mine.** The `## Closed` section still reads
`*(none yet)*` — the run marked entries closed *in place* rather than moving
them, because I seeded the file with both a per-entry `Status:` field and
`Open`/`Blocked`/`Closed` section headings and never said which one wins. The
run's choice preserves reading order and is defensible; the format was
ambiguous. Fix: drop the headings, keep the status field.

Untested: the `blocked` path. Nothing this phase needed a host install, so
neither the `waiting_external` handoff nor the "a fire that finds only
blocked entries says so and stops" rule was exercised. The run did make the
adjacent correct call — it ranked ControlNet down for needing weights and
declined to open a handoff for a method it had not selected.

## The animation result

**Sprite-sheet-of-generated-frames is what ships; skeletal rigging stays
manual.** Every consumer-facing pixel-art generation tool the survey found
returns a frame sequence consumed as ordinary engine flipbook animation. AI
enters the skeletal pipeline *upstream* of the rig — generating still
reference or part art that a human cuts and rigs by hand — and the one
directly relevant paper is not pixel-art and reports its pose estimation
"does not generalize well to illustrations in sprite sheets".

**The method that won: locked-seed, pose-shifted multi-still generation.**
Pure SDXL text2img through the existing runner, no download, no new backend.

**What it produced:** 4- and 8-frame 64×64 side-view walking sequences of a
dog, per-frame PNGs plus an assembled sheet, one shared palette of 29–32
colours across each sequence. The varied-seed control breaks identity
visibly — scale, markings and framing swing frame to frame. **Seed-locking,
not pose wording, is what carries identity.** The sheets are in `report3.md`
and need no metric to read.

Not met, and said so: **looping was not verified** (Q9), and the background
is flat but lavender rather than white with the cast ground shadow present
in every frame — both inherited from `pixelArtDiffusionXL`, neither new.

**The instrument did its job on itself.** `consistency_instrument.py` was
tested against a deliberately broken sequence *and* a naturally bad one
before being trusted, and the result is that **no single signal covers both
failure modes**: palette drift catches an alien frame at 1.0 vs 0.0 but reads
**0.0000** on the worst real sequence, because the runner force-quantises
every frame onto one shared palette before saving. It is blind to exactly the
failure its own pipeline most likely produces. That is strictly better than
`bg_flatness`, which was trusted for a whole run before p9 caught it — here
the blind spot was found inside the task that built the instrument.

### What an agforge-side consumer would need (hand-off material, not action)

Out of scope this phase; recorded so it does not have to be re-derived:

- **The artefact is a sheet plus per-frame PNGs on one shared palette.** Any
  engine that does flipbook animation consumes it directly. No rig, no
  skeletal format, no importer.
- **The generation contract is: one seed, N pose clauses.** An asset request
  would need to carry the subject, the frame count, and the pose vocabulary;
  everything else is inherited settings.
- **Two known defects travel with it** — non-white flat background and a cast
  ground shadow in every frame. Both are checkpoint properties that
  negative-prompt wording does not remove (p9 closed that route). A consumer
  wants a post-process key/crop step, not different prompts.
- **The consistency instrument is a gate, not a score.** Use palette drift
  plus one of silhouette variance / grid alignment, always beside the images,
  never as a sort key.
- **Unresolved before this is production-ready:** whether the cycle loops
  (Q9). A walk cycle that does not hand back to its first frame is not usable
  as a looping animation, and nothing has checked.

## Costs, timings, GPU

- **20 paid runs, $7.44, 41 minutes wall clock** (06:39Z → 07:20Z). Split:
  autolab $5.69 across 9 runs, Front $1.75 across 11.
- **The stall cost $0.45 directly** (the abandoned task-3 run) and roughly
  another $0.14 for the Front run that reported the non-progress. The resumed
  task 3 was the mission's largest single run at 450 s / 57 turns / $1.58,
  partly because it re-did work whose timings had been lost.
- **GPU time: about 3.6 minutes total** — 25 generations, one cold at 25.6 s
  and the rest warm at 7.9–8.5 s. The card was free throughout; no contention
  with agforge appeared, though a matrix this small would not have shown it.
- The cold/warm figures **matched the literals carried verbatim in the
  workplan**, which independently confirms those numbers were still true.
- **No task came close to its budget.** The largest used 450 s of 1200 s and
  57 turns of 200. Timeout was never the constraint — the one run that ended
  early ended by choice.

### Does one cycle fit a routine fire, or want a mission chain?

**It wants a mission chain, and it got one — that is fine.** A fire is one
Front conversation; the cycle underneath it was a four-task mission and
twenty runs. Front supervised across all four without a second human
instruction, and the acceptance gate between tasks worked as designed.

The honest limit is not budget but **serialisation**: Front's listener is
serial, so a supervising fire is also how long the next `front-` post waits.
41 minutes is tolerable; it would not be if two routines fired near each
other. Worth watching before any cadence is set, not worth fixing yet.

## Recommendations only

- **Cadence for `routine-mediagen`: not yet weekly — fire it once more, bare,
  first.** The half of the loop this phase did not test is "pick your own
  question", and one bare fire against the eight open entries answers it for
  ~$7. If that fire picks a sensible question and closes it, a weekly cadence
  is justified; the queue has enough in it to feed one. Setting a cadence
  before that test would be scheduling something whose selection behaviour is
  unobserved.
- **Fix the two guide lines before the next fire, not after.** They are
  one-line edits with no code and no restart, and the failure they prevent
  cost eight minutes of deadlock and ~$0.6 in this fire alone. Doing them
  first makes the bare fire a cleaner test of everything else.
- **Next subject: stay on images one more round; do not jump to video or
  audio yet.** Q9 (does it loop) is the question standing between this result
  and something agforge could use, and it is cheap. The video route is
  already ranked second on the board with its costs written down (Wan 2.2
  I2V, minutes per generation, needs a frame-extraction and grid-snap step) —
  it is a good *third* fire, with its own task, and the standing text already
  says to measure one generation before committing to a grid.
- **`publish/` deserves its first review run soon, but not next.**
  `main/` now holds two subjects, a tips file each, and an 11-entry
  `QUESTIONS.md` — enough that a review has something to say, and
  `QUESTIONS.md` in particular has never been checked against publication
  conditions by anything but its author. Worth doing once the queue-format
  ambiguity above is fixed, so the review is not reviewing a shape that is
  about to change.
- **Fix the `QUESTIONS.md` format ambiguity** (status field vs. section
  headings — pick one) as part of whatever touches the pattern next.
- **Add one line to the routine's step 4**: check `main/` against the
  publication conditions *before* committing, not after. See the second
  defect below.

## Deus Ex Machina

- **Broke the task-3 / Front deadlock** by posting an explanation, not an
  action, into `front-routine-mediagen`; Front restarted the task itself —
  handoff candidate, and the phase's main finding (two one-line guide edits,
  named above).
- **Wrote `agent/project_pattern.md`'s two additions as the Developer**
  rather than asking autolab to edit its own pattern file — handoff
  candidate.
- **Seeded `main/QUESTIONS.md` and edited `main/INDEX.md` by hand**, as the
  plan permits — handoff candidate; note that the queue's first *consumer*
  was an agent, which is the part that mattered.
- **Fired the routine by a hand post rather than through the dispatcher**,
  because Front is event-driven and a separate `trigger.sh` call would have
  double-fired — handoff candidate, minor.
- **Redacted the internal repository name from `main/spriteSheetFrames/`**
  after the run had pushed it — handoff candidate, and the reason the
  recommendation above adds a publication-conditions line to the routine's
  step 4.
- **Copied two sprite sheets into this public episode folder** for the
  report. No host facts in them.
