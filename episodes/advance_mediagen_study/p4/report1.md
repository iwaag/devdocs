# Step 1 — Fire 1: the survey p3 skipped, and the ranking it produced

One fire of `routine-mediagen`, 2026-08-30 11:47Z. Mission **M-23**, three
tasks. The fire's whole deliverable was **Gather** — no generation matrix,
nothing to verify with pixels — followed by a ranked go/no-go.

The plan's premise was that p3 did no web research because my own p3 fire
said "do not redo the survey", and that hint travelled verbatim into the
mission and overrode step 1 of the standing request. That is what happened,
and it is my defect rather than the run's. This fire says so in its own text
and buys the survey back.

## The fire

One post into `#front › front-routine-mediagen` (message 3449), as the
Developer. One post, all the context — p1 established that you cannot post
context and then fire, because **any** post in a `front-` topic *is* the
fire.

It carried: the subject; the instruction that Gather is the deliverable and
that `skeletalRig/summary.md`'s "reuse, not restate" section is what this
fire replaces; the five directions as **rankable hints, explicitly not an
ordering**; p3's established facts as do-not-re-test; the three-part
go/no-go specification; the stop-if-nothing-clears clause; the
`waiting_external` handoff path; and the publication conditions.

Front opened `#pj-mediagen › workplan-skeletal-animation-survey` (3452)
four minutes later, reproducing the fire close to verbatim and saying out
loud that it was doing so. autolab planned M-23 with three sub-works —
survey, ranking, publication-check-and-push — and opened all three run
topics. Front approved and started task 1 under standing authorization.

**One paraphrase drift worth recording.** I asked for the workplan to say
"in these words" that *Gather is the deliverable of this fire*. Front wrote
"**Gather is the whole deliverable**". The meaning survived intact and the
"reuse, not restate" clause came through literally, so I did not intervene
— but it is a live, miniature instance of the routine's own standing lesson
that literals do not survive paraphrase, this time applied to an instruction
that had asked to be quoted. The lesson generalises: **a request to use
particular words is itself a literal, and it is subject to the same drift
as an endpoint or a filename.**

## Did the survey actually happen? The cheap test says yes

The plan named URL count as the cheap test. Roughly **twenty-five distinct
URLs** were fetched or read, across all five areas, and I spot-checked the
two most decision-relevant citations against their primary sources myself
rather than trusting the report. Both are exact:

- spine-canvaskit's docs do say it can be used in "backends (to render
  skeletons headlessly)", with a working `spine-canvaskit/example/headless.js`.
- Spine's CLI docs do say "to export images or video an OS windowing system
  and OpenGL are required."

Three findings are worth more than the count:

- **spine-canvaskit is a genuinely headless, first-party, code-example-backed
  render path for Spine** — and it answers p3's open engine question *against*
  my expectation. I assumed Spine's CLI would be the answer; it is not,
  because Spine's own CLI needs a windowing system and OpenGL to export.
  That is the same shape as p3's Godot `--headless`/`--write-movie` finding,
  reached this time by reading rather than by crashing.
- **A self-correction against a primary source, unprompted.** The sibling
  survey's "pixel art was not in scope" overstated SpriteToMesh's exclusion:
  the paper's training data does list pixel art, while its limitations
  section still confines generalization to Spine2D game assets. The run
  corrected its own project's earlier text.
- **An honest negative that saves a download.** No surveyed segmenter does
  *internal* anatomical boundaries between two limbs of the same colour.
  `rembg`/BiRefNet would sharpen the outer silhouette and would not close
  p3's actual gap. This is the finding that makes the eventual winner
  cheap.

The survey ends with a **"What this survey could not settle"** list of six
named holes — including one where the run marks its *own* reasoning as an
untested inference rather than dressing it as a source. A survey with a
named hole is worth more than one that reads complete, and marking your own
inference as inference is the difference between a survey and a summary.

## The ranking

Thirteen ideas, each with what it needs beyond this host, the expected gain
against p1/p3's asset bar, and one line of rank reasoning.

The move that makes it a ranking rather than a list is stated before the
table: three of the four bar clauses (64×64, ≤32 colours, keyable
background) are already met **by construction**, so "gain over p3" collapses
to one question — *does this give the rig more genuine per-limb degrees of
freedom without re-opening a clause p3 already has solved for free?* Every
row is then scored against that one question.

| rank | idea | verdict |
|---|---|---|
| 1 | Deliberately spread-leg generation pose, re-run the existing cut script | **Go** — zero cost, targets the diagnosed mechanism |
| 2 | `flux1-kontext-dev` generates isolated per-limb parts | **Go**, below #1 — on the box, but the capability is unverified anywhere and risks re-opening solved clauses |
| 3–13 | mesh-deform, video-motion + pose extraction, PolyRig, Spine/spine-canvaskit, Phaser, layered-generation SaaS, SpriteToMesh, Live2D, DragonBones | **No-go** |

The no-go reasoning is the valuable half. Rows 3, 6 and 7 are ranked no-go
**not because they are blocked but because even fully unblocked they would
not touch the bottleneck.** Ranking a real, documented, genuinely-headless
capability (spine-canvaskit) as a no-go on those grounds is the best
judgement call in the table — the survey's own best discovery is ranked
last-but-not-viable *on the criterion*, which is exactly what a criterion
is for.

Two things I put on the record when accepting:

- **The winner is not a hedge, and the run said so.** "Zero cost" is the
  usual way a run manufactures a yes when the honest answer is no. Here
  idea 1 is ranked first on a *stated mechanism* — more real leg length
  falls inside the posable parts, which is the literal cause of p3's
  byte-identical frame pair — and its caveat is named as **unverified**,
  not as *cheap*.
- **No handoff was opened, and that was recorded rather than skipped.**
  The task was primed to open a `waiting_external` and instead wrote down
  why it did not. Nothing for me to install before fire 2.

## What I did as the requester

I read every artefact rather than accepting the reports. Acceptance gates
the next task, so each acceptance carried the review with it.

Reviewing `main/` myself turned up **a leak nobody was looking for**:

```
$ grep -rn 'agforge' main/
subjects/spriteSheetFrames/summary.md:142: ...and agforge's reusable `comfy_video.py` submit/poll/fetch code; ...
```

`agforge` is an internal agent and repository name, and it has been in
published `main/` since commit `165a689` — through p3's entire redaction
round **and** through my own independent check afterwards. That check was
too narrow: I grepped for `gentest-`, hostnames, IPs, ports and ignored
paths, and never for our own agents' names. The redaction round found what
it was told to look for and stopped; so did I.

**The generalisable form: a redaction scoped to the leak you already found
verifies only that leak.** p3's lesson was "a self-check phrased as an
instruction is a claim; only the command is evidence". This round adds the
next one — *the command is only evidence for the pattern it greps for.*
I widened it and gave the run the widened command to run and paste:

```
grep -rnE 'gentest-|report\.md|agforge|agautolab|agfront|agecho|arxivsage|cagent|agstudio|192\.168|localhost|:7801|:8188|/Users/' main/
```

I also found the six surviving `report.md` pointers the plan's MUST NOT
names — one in `summary.md`, five in `tips.md` — all in sections task 1 was
correctly told not to touch, so they are task 3's job rather than a miss.

## A blocker raised before it could bite

The plan says fire 2 may extend `gentest-skeletalRig/` because "its yaml is
not `verified`". **It is** — `localtest.yaml` reads `state: verified`. Under
standing request v2 that means fire 2 opens a *new* repository, which would
strand the existing rig-building and segmentation scripts and the four
rendered frames across a repository boundary, for what is a rerun of that
same pipeline with one prompt clause changed.

Rather than relax the constraint myself or let fire 2 discover it at its
start, I asked task 3 to decide it and write the decision and its reasoning
into the yaml — either a state that says "proven and being extended", if the
convention has one, or an explicit statement that fire 2 opens a new
repository and copies the code in — and said plainly: **do not change the
state without saying so.** I would rather have the constraint enforced and
worked around openly than quietly relaxed.

## Task 3: the publication fixes, and the decision I asked for

Task 3 did all three things and I verified each myself rather than relaying
the self-check.

**The leak and the pointers are gone.** The `agforge` line now names the
capability instead of the owner — "existing reusable submit/poll/fetch code
for the video backend" — and all six `report.md` pointers describe what the
evidence *is*. My own check:

```
$ cd main && git grep -nE 'gentest-|report\.md|agforge|agautolab|agfront|agecho|arxivsage|cagent|agstudio|192\.168|localhost|:7801|:8188|/Users/'
$ echo $?
1
```

`git grep` rather than `grep -r` is worth keeping as the standing form: it
only looks at committed content, so the `.git` config noise the run's own
report had to caveat cannot appear at all.

**The yaml decision came back the right way, by the right method.** Asked to
settle the contradiction between my plan and the standing request, the run
checked the seven-state convention across both sibling repositories, found
no state meaning "proven and being extended", and concluded that inventing
one **would break the shared convention rather than follow it**. So the state
stays `verified` and fire 2 opens a new repository, copying in the proven
`build_rig.py` / `segment_parts.py` / rendered frames. The reasoning is
written into `localtest.yaml` where fire 2 will find it. The constraint was
enforced and worked around openly rather than quietly relaxed — which is
exactly what I asked for and is the more valuable half of the answer.

**It checked `main` against `publish/` instead of assuming**, and found
`publish/` has zero commits locally and on its remote, so the equality
condition does not yet apply. Better than either asserting equality or
silently skipping.

Six new "Still open" items were appended, `INDEX.md` needed no change (its
date was already current), and both repositories pushed: `main` at
`71564dc`, `gentest-skeletalRig` at `a22d700`, `publish/` untouched.

**One process note.** The run wrote "per instructions, I should get developer
approval before committing" and then committed and pushed in the same turn.
The outcome is right — it is the terminal task and the standing request tells
it to push — but the sentence and the action disagreed. Recorded rather than
bounced.

## Costs

| | runs | cost |
|---|---|---|
| supercoder (the three tasks) | 5 | $3.59 |
| Front (supervision) | 8 | $2.23 |
| superdirector (planning) | 1 | $0.24 |
| | **14** | **$6.06** |

Roughly half p3's $12.11, for a fire with **no generation calls and zero GPU
seconds**. The difference is that the coordination overhead which dominated
p3 — two follow-up missions, a 26-minute dropped-callback stall, a
bounced-back publication leak — did not recur. One redundant run is
attributable to me (friction 1 below).

## Frictions

**1. My acceptance crossed Front's own report.** I read task 1's artefacts
directly and posted acceptance at 12:01:28Z; Front's report on the same task
had landed at 12:00:10Z and it had already started task 2. My post then woke
a Front run that found nothing new to do and said so (3491). One redundant
paid supervision run. The cause is that I was reading the run topics
directly and so was ahead of my own supervisor — supervising *and* watching
the work costs a run every time the two cross.

**2. `agentchat send` was blocked by the harness's auto-mode classifier.**
The fire could not go out at all until the Developer switched the session's
permission mode. Read calls passed; the outbound post did not. This is the
harness around the Omni Agent, not the system under study, but it is on the
record as a thing that stops a fire dead: the allowlist is not the lever,
the permission mode is.

**3. The build of `agentchat` on the front venv has no `wait` subcommand.**
Supervision from the harness side fell back to polling `read --since`. Not
a defect of the system — the newer blocking read exists — just a version
skew worth knowing before planning a supervision loop.

## Deus Ex Machina

- **Verified two of the survey's citations against their primary sources
  myself** (spine-canvaskit headless, Spine CLI's OpenGL requirement) rather
  than accepting the run's quotations. Both exact. This is the "test it
  rather than trust the reading" habit p3 asked for, applied by me to the
  run instead of by the run to a document. Not a handoff candidate — checking
  a supplier's citations is the requester's job.
- **Found the `agforge` leak in already-pushed `main/` by hand**, and
  bounced it back rather than editing `main/` myself. Handoff candidate, in
  one line: *nothing in the loop greps `main/` for internal **agent** names,
  and the one round that did grep it scoped the pattern to the leak it had
  already found.*
- **Raised the `verified`-yaml contradiction between my own plan and the
  standing request** and handed the decision to the run with the constraint
  intact. Information, not action — and the run reached a better answer than
  the one I would have imposed, by checking the sibling repositories'
  convention rather than reasoning from the single case in front of it.
