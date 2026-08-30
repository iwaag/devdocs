# Step 1 — Framework: workflow subjects and an open-question queue

Harness-side Developer work, done by the Omni Agent. Doc change only; no
service restart, and `autolab doc patterns` reads the file per invocation
(verified: the new `QUESTIONS.md` text comes back from the CLI without a
reload).

## What changed in `agent/project_pattern.md`

Two additions, both in the pattern file the study routines actually read.
Commit `b6fa5d0` on `agautolab`, pushed.

**1. A subject may be a workflow family.** The "Repository-backed generation
tests" section previously mentioned "a checkpoint, a LoRA set, or a workflow
family" only in passing, inside the sentence describing the resumable yaml's
`subject` field — which is not a place a run looks when it is deciding
whether it is allowed to start a workflow subject at all. It now says so
explicitly, as its own paragraph: a workflow family gets the same
`main/<subject>/{summary.md,tips.md}`, the same `gentest-<subject>/`
repository by the same route, and its own row in `INDEX.md` under the
workflow-families heading. The only thing that differs is what a tip is
*about* — "wiring these steps in this order produces that, and this step is
where it breaks" rather than "under these settings this model produces
that".

The paragraph ends with one prohibition that this phase needs and that the
pattern did not previously imply: **do not fold a workflow subject into an
existing checkpoint's `gentest-` repository.** That repository's yaml holds
one subject and one state, and `gentest-pixelArtDiffusionXL`'s state is
`verified`. Without that line a run reading the pattern could reasonably
have decided that extending the existing repository was the frugal choice.

**2. A new section, "The open-question queue: `main/QUESTIONS.md`".** It
defines the file as part of `main/` and therefore publish-ready, gives the
six fields of an entry (subject, question, why it matters, what evidence
would close it, date raised, status), and states the three behaviours that
make it a queue rather than a list:

- a run **closes** an entry by appending the answering tip to that subject's
  `tips.md` and marking the entry closed — closed entries stay in the file;
- a run **raises** the questions its own work created;
- **a fire that finds only `blocked` entries says so and stops**, and does
  not invent a question in order to have something to do.

It also states the relationship to the per-subject "Still open" sections
plainly, because they now overlap on purpose: `tips.md`'s "Still open" is
the local narrative for someone reading one subject end to end;
`QUESTIONS.md` is the cross-subject queue a routine reads first, and it is
the one that gets consumed. And it routes the `waiting_external` handoff
through the queue: a run blocked on a host-level install raises its request
here as `blocked`, because this file is the only thing a later fire is
guaranteed to read — the handoff record in a test repository is not.

The `"study"` pattern's own folder list gained a `main/QUESTIONS.md` line so
the file is visible to a run that reads only that section.

## `QUESTIONS.md` as landed

Committed to `autodev/mediagen` (`main/QUESTIONS.md`, commit `fc1b606`,
pushed) with eight seeded entries — all `open`, nothing `blocked`, nothing
`closed`:

| id | subject | question, in short |
|---|---|---|
| Q1 | `pixelArtDiffusionXL` | Does `pixel art` early in the prompt help or hurt? (the checkpoint page and a sibling LoRA card disagree) |
| Q2 | `pixelArtDiffusionXL` | Pixel-art LoRA on a stock checkpoint vs. the dedicated checkpoint — same quality, same shadow defect? |
| Q3 | `pixelArtDiffusionXL` | Do palette size, sampler, or step count move anything? All three were held fixed in every matrix so far |
| Q4 | `pixelArtDiffusionXL` | Does the ground-contact defect respond to anything other than negative-prompt wording? |
| Q5 | `pixelArtDiffusionXL` | Re-measure `cat_sentence_cfg9`'s wall time on an idle backend |
| Q6 | *(new workflow family)* | Skeletal vs. sprite-sheet animation on the AI-generation premise — the web survey and the ranking |
| Q7 | *(the family Q6 names)* | Can the chosen method produce a usable 64×64 looping walking cycle? |
| Q8 | *(the family Q6 names)* | How is frame-to-frame consistency measured, and where is that instrument blind? |

Q1–Q5 are a straight transcription of `main/pixelArtDiffusionXL/tips.md`'s
"Still open" list; Q6–Q8 are this phase's animation questions, written
before the routine exists so that step 3's fire has something real to pick.

`main/INDEX.md` changed in the same commit: it now points at `QUESTIONS.md`
in its opening, and its "Workflow families" section — which said "None
found", explaining that the backend's model-listing endpoint exposes no
saved workflows — now says what a workflow family *is* as a subject, keeps
the endpoint note as the reason such subjects are not auto-discoverable,
and carries an empty table ready for a row. One stale sentence ("no
`gentest-` repository exists yet") was corrected while there.

## Does the queue read as something a cold run could act on?

Mostly yes, with one honest weakness.

What works: every entry names the evidence that would close it in concrete
terms — "one axis, everything else fixed, `pixel art` first / late /
omitted" rather than "investigate prompt ordering". A run that has never
seen this plan can read Q1 and know what matrix to write. Q6→Q7→Q8 also
read in the right order without being numbered as a dependency chain: Q6's
evidence is a survey and a ranking, Q7's evidence needs Q6's answer to
exist, Q8's needs Q7's frames. A cold run picking Q7 first would discover
it has to do Q6 anyway.

The weakness: **Q6, Q7 and Q8 carry `-` as their subject** because the
workflow family has no name until the survey picks one, and the pattern
says a subject is the `main/<subject>/` folder. A cold run reading Q7 alone
has no folder to append a tip to, and has to infer that answering Q6 is
what names it. That is a real gap in the file's self-sufficiency; it is
written down here rather than papered over, and step 3's fire is the test
of whether it actually trips a run. If it does, the fix is a field for "the
subject this question will create" rather than an overloaded `-`.

A second, smaller one: the queue has no notion of cost or size. Q3 is three
axes and Q5 is a single re-generation, and nothing in the file tells a run
that Q5 is fifteen seconds of work. A routine that picks by "what looks
interesting" will never pick Q5. Not fixed this step — noted as a candidate
for the next.

## Deus Ex Machina

- Wrote `agent/project_pattern.md`'s two additions as the Developer, rather
  than asking autolab to edit its own pattern file — handoff candidate.
- Seeded `main/QUESTIONS.md` and edited `main/INDEX.md` by hand, as the plan
  permits — handoff candidate. Note that no agent has yet *consumed* this
  file; step 3 is the first time one will.
