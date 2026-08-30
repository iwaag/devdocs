# Step 2 — The routine: one fire = one cycle

The standing request for the `mediagen` routine is posted as v1 in
`#front › routine-mediagen`, message 3160, 2026-08-30. The runs and my
comments on them go in `front-routine-mediagen`, per the ordinary routine
convention — `trigger.sh` posts there, Front is served because the last
real poster in a `front-` topic is not Front, and the standing text lives
in a topic without the `front-` prefix so Front never serves it as a
request.

## The standing text

Reproduced below **with the backend block redacted**, because that block is
the one part of it that carries host facts and `devdocs` is a public
repository. Its shape is described instead of its contents; the literals are
in the Zulip post and nowhere else in version control.

---

> Standing request for the `mediagen` routine, **v1**.
>
> One fire is **one cycle**: gather → invent → verify → improve. In
> `#pj-mediagen`, ask autolab for the mission (or mission chain) that carries
> out one whole cycle. Approve and follow through an appropriate plan without
> asking me for a second confirmation.
>
> ### The cycle
>
> **0. Pick the question.** Read `main/INDEX.md` and `main/QUESTIONS.md` in
> the `mediagen` workspace. Pick **one** `open` entry — or, if my fire names a
> subject, that subject — and say in the workplan which one and why that one.
> If every entry is `blocked`, say so and stop: that is a complete fire, not a
> failed one. Do not invent a question so as to have something to do.
>
> **1. Gather.** Web research on that question. Paraphrase in your own words,
> cite the URLs you read, and quote nothing at length — `main/` is written
> publish-ready from day one and the publication conditions apply to this text
> before anyone reviews it. Say what you could **not** find out; a survey with
> a hole named in it is worth more than one that reads complete.
>
> **2. Invent.** Propose the method and the matrix that would answer the
> question. **State the asset requirement you will judge against before you
> generate anything**, not after. Where the question extends existing
> knowledge, reuse the existing requirement verbatim rather than writing a new
> one.
>
> **3. Verify.** Run it in that subject's own `gentest-<subject>/` repository
> — `autolab project init-repo gentest-<subject>` plus a hand-written
> resumable yaml, recorded in `README_PROJECT.md`. A new subject gets a
> **new** repository; do not extend a `gentest-` repository whose yaml is
> `verified`. Reuse code rather than rewriting it: the existing checkpoint
> test's `runner.py` already does backend call → nearest-neighbour downscale →
> ≤32-colour quantise → contact sheet → resumable `results.csv` upsert. Bound
> the work to one task's budget, fix the seed, and sweep one axis at a time.
>
> **4. Improve.** Append to `main/<subject>/tips.md` — one evidence line and
> the date per tip, append-only, and a condition that changed nothing is a tip
> too. Update the subject's row in `main/INDEX.md`. In `main/QUESTIONS.md`,
> **close** the entry you answered (marking it, not deleting it) and **raise**
> the questions your own work created. Commit and push `main` and the gentest
> repository. Never push `publish/`.
>
> ### When the method needs something installed on the host
>
> Downloading a checkpoint, adding a custom node, or standing up a tool on the
> GPU node is a host change outside your workspace, and there is already a
> proven path for it — do not invent one. Finish the fire in
> `waiting_external` with a handoff record stating: the operation requested
> and why the method needs it; expected disk/VRAM/network/runtime impact; the
> processes, ports and files it expects to create; a **read-only check that
> proves the operation is done**; and cleanup or rollback advice. Then do two
> more things, because nothing watches a handoff record on its own:
>
> - post the request in the `workplan-` topic, naming me; and
> - raise a `blocked` entry in `main/QUESTIONS.md` naming the human action it
>   waits on.
>
> A later fire reads the persisted state, observes the new reality with
> ordinary read tools, and continues without me restating anything. That is a
> successful cycle.
>
> ### Things earlier rounds paid for
>
> - **Literals do not survive paraphrase.** Planning paraphrases the mission,
>   and a paraphrase drops endpoints, model filenames and credential paths.
>   Anything a task cannot re-derive goes into the workplan **verbatim**, and
>   the workplan says out loud that it is doing so. One earlier task burned
>   most of its budget rediscovering an endpoint that had been dropped between
>   the mission and the task.
> - **A metric is a secondary signal beside the image, never a sort key.** The
>   background-flatness instrument ranked the one outright failure as the
>   flattest cell of twenty. Any new instrument you build gets tested against a
>   case you already know is bad, and you report where it lies.
> - **Video is not image.** `WORK_TIMEOUT_SECONDS` is 1200; about 20 SDXL
>   images fit in one task and a video matrix does not. Give video work its own
>   task, and measure **one** generation before committing to a grid.
> - **A mission cannot quote its own cost.** Read it from the role's run JSON.
> - **A task is not closed until I agree it is done**, and that acceptance
>   gates the next task. Budget for it: a two-task mission is about five paid
>   runs, not three.
>
> ### The backend, verbatim — copy these into the workplan as they stand
>
> *(redacted here — see below)*
>
> **Nothing here goes into `main/`, `publish/`, or any public repository** — no
> hostnames, IPs, ports, paths, internal repository names or credentials. They
> live in the workplan and in the test repository's ignored `.local/` only.
>
> ### Report back here
>
> In this topic, when the cycle ends: the workplan topic; which question you
> picked and why; the method chosen and the reasoning that ranked it above the
> alternatives; the repository and its state; the evidence produced; which
> backend you used; what you appended to `tips.md` and what you closed and
> raised in `QUESTIONS.md`; the cost from the run JSONs; and any request
> waiting on me.

---

## What the redacted backend block contains

Nine bullets, all facts a task cannot re-derive and would otherwise
rediscover at its own expense:

1. The image backend's base URL as a literal, with the reason to use the
   address rather than the name (the name stalls ~5 s on an unanswered AAAA
   lookup — the trap recorded from an earlier episode), and its version.
2. Where the embedded second backend lives and where a standalone one
   answers, both versions, and the instruction to **say which was used**.
3. The whole API surface needed: session creation, the generate call with
   every parameter named, the shape of the response, and the fact that the
   returned image path is relative. Plus the two silent-default traps —
   everything but `model` falls back to a 512² JPEG, and JPEG ringing
   survives a nearest-neighbour downscale, so the size and format arguments
   are never optional.
4. The GPU: one card, its memory, that it is **shared with agforge's
   production asset runs**, and that a 14B video generation holds it for
   minutes.
5. Measured per-image timings, cold and warm.
6. The weights actually present, with the warning that a node existing does
   not mean its weights are on the box.
7. The weights **absent** — no ControlNet files at all, no pose estimator, no
   frame interpolation — and that downloading one is permitted but is a host
   change, and that the download cost is a legitimate reason to rank a method
   lower on present-day certainty.
8. Two pieces of reusable code beyond the matrix runner, named by path in the
   workspaces.
9. The prohibition: none of this reaches `main/`, `publish/`, or a public
   repository.

This block is the whole of p9's "literals do not survive paraphrase" lesson
turned into standing text. It is also the largest thing that will go stale:
every version number, timing and weights listing in it was checked on
2026-08-30 and is dated as such in the post.

## What it deliberately does not prescribe

The routine has to *choose*, so the text says nothing about:

- **Which method.** No mention of i2v frame extraction, keyframe editing,
  pose conditioning or cut-out rigging as candidates, and no hint of which
  is preferred. The plan's four candidate families were deliberately left
  out of the standing text; the run reads them, if at all, out of its own web
  survey. The one thing the text does supply is what would make ranking
  possible — which weights are present, which are absent, and what a download
  costs — and it names "present-day certainty" as the axis without saying
  what scores well on it.
- **Which question.** Step 0 says read the queue and pick one, and say why.
  The `#pj-mediagen` fire may name a subject; a bare fire must choose.
- **The instrument.** Step 4 in the plan wanted a frame-consistency measure;
  the standing text asks only that any instrument be tested against a case
  known to be bad and that its blind spot be reported. Not what to measure,
  not how.
- **The frame count, the subject animal, or the deliverable format.** Those
  are in `QUESTIONS.md` Q7 as a *suggested* requirement the run may adjust
  with a stated reason — deliberately in the queue rather than in the
  standing text, so a later fire on a different question is not carrying a
  walking-cycle spec it has no use for.
- **How many missions or tasks.** "The mission (or mission chain) that
  carries out one whole cycle." The plan's own uncertainty about whether a
  cycle fits one fire is left as something this fire will answer rather than
  something the text decides in advance.
- **Which backend.** Both are declared fine; the report must say which.

What it *does* prescribe is entirely process: read the queue first, state the
requirement before generating, one axis at a time, seed fixed, close and
raise queue entries, push `main` and the gentest repository, never push
`publish/`, and the `waiting_external` route with its two extra obligations
(post in the topic, raise a `blocked` entry). Those are the parts an earlier
round paid for; everything about *what to generate* is left open.

## One thing added beyond the plan's step 2

The plan lists the `waiting_external` handoff's two known gaps under
background. The standing text turns both into instructions rather than
observations, because "nothing watches the handoff" is not a fact a run can
act on: it now must post the request in the `workplan-` topic naming the
Developer, **and** raise the `blocked` queue entry. The queue entry is the
part that makes a later fire self-sufficient; the post is the part that makes
a human notice this week.

## Deus Ex Machina

- Posted the standing request as the Developer account, as every previous
  routine's standing text was — handoff candidate, though it is arguably the
  Developer's own job by definition: the standing request is the human's
  request.
