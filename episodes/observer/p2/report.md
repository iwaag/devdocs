# observer p2 — report

## What was asked, and what happened

p1 built an agent that waits. p2 asked whether that changes how real work
runs: can an executing agent **discover** Observer, hand it a wait, end its
serving, and pick the work up again on the notification — while doing a study
somebody actually wanted?

It can, and it did seven times in one afternoon, on a study of YuE2 that
produced five songs, a new mediagen subject, and one Observer defect worth
more than any of them.

Reports per step: [1](report1.md) (discovery before the request),
[2](report2.md) (preparing and requesting the study), [3](report3.md)
(waiting and continuation in real work), [4](report4.md) (evaluation).

## The chain, and the thing that makes it a chain

The request named no agent, no channel and no command. It said only that
there is an agent on the board whose job is waiting, and that the executor
should read the introductions. Everything else came from step 1:

- `agentchat intro [<agent>]` (pyagag `1691328`), the board read live from
  inside a run — the same content `tools/agents.md` snapshots at the start of
  a serving, available to a run that wants one contract now;
- one paragraph in `agentchat --help` saying that anything slow may be taken
  on by an agent on the board, naming none;
- and Observer's introduction, extended to say what a requester **does while
  it waits**: finish, name the conversation that serves you, leave your next
  action behind for a run that remembers nothing, write a job condition as
  *"it has ended"* with what failure looks like, and give a target reachable
  from Observer's host.

Every one of those sentences came back in the executor's own behaviour. Its
watch requests say *"this is 'it has ended', not 'it succeeded'"* and
enumerate success, failure and stall. It named HTTP endpoints on the GPU node
rather than paths its own machine could see. It wrote down, before ending
each serving, which job it had started and what it would do with the answer.
Nobody told it any of that in the request.

The first complete trace, with autolab's log as the witness:

| time (UTC) | event |
|---|---|
| 07:32:43 | executor ends its serving, wait handed off |
| 07:33:00 | Observer accepts `w6800` — **requester not served** |
| 07:34:06 | Observer judges `met`, delivers into the run topic |
| 07:34:06 | **autolab's listener serves the task — executor resumes** |
| 07:35:08 | resumed run reports the collected result |

Five watches covered a 7.3 GB download and eight ComfyUI jobs. **No human and
no Omni Agent restarted an executor at any point in the phase.** Acceptance
never woke the requester; only the notification did.

## What broke, and what it taught

**A two-part condition was judged on one part.** `w6866` watched *"BOTH jobs
have ended"* and answered `met` while its own evidence read *"…**BUT** GET
/history/cc849181 returns empty `{}` and it is still listed in
queue_running"*. The verdict contradicted the evidence beside it.

p1 measured 19 looks with no incorrect judgment — and every one of those
conditions had a single target. The first conjunction in the realm broke it.
It was harmless by luck: the second job finished during that 77-second
evaluation. Had it been slower, the executor would have been woken to collect
a result that did not exist, and the watch was already finished and resolved,
so nothing would have woken it again.

The fix is a guide paragraph, not code (`pj-agdev` `aba1a9c`): every part must
hold in the same look, the parts are enumerated one at a time in the
evidence, and **a verdict must agree with its own evidence** — a "but" in what
you are writing has already answered the question. The guide is read per
evaluation, so nothing was restarted. The next two-job watch, `w6882`, passed
twice through the exact state that had been misjudged and answered `not_met`
both times; seven looks, all correct.

The lesson that outlives the fix went into the study's own tips as well:
**one watch per thing you are waiting for.** A conjunction is the hardest
thing a small local model is asked to do here, and nothing about the design
forces two jobs into one watch.

**Evaluations cost more than p1 measured**, and the cost follows the
condition rather than the queue: 19–30 s single-target, **30–120 s** for the
two-job ones. Since evaluation is sequential, a complex condition makes every
other watch wait too. p1 named sequential evaluation as its ceiling; p2 puts
a second axis on it.

## The study, which was real work and not a fixture

YuE2 is a lyrics-to-song model released days before this phase. The study
went through mediagen's normal entrance — a `workplan-` topic in the
project's own channel, planned by autolab, executed in four tasks — and added
the project's first **music** subject (`autodev/mediagen@e4d8bf5`, `c415b0c`).

What it established, on a Quadro RTX 8000 the model's own documentation does
not really support:

- **It runs.** ~136 s a song, ~6.8 GB resident against upstream's stated
  24 GB floor. The BF16-on-Turing risk did not materialise.
- **It is exactly reproducible — and "byte-identical" is the wrong word.**
  Two runs of one seed differ by a single metadata byte; decoded PCM is
  identical. Verified independently by the executor and by the Omni Agent.
- **A resubmitted identical graph is answered by ComfyUI's cache in 58 ms**,
  which would have produced a *fake* reproducibility result. The executor
  caught it by checking wall time against the ~136 s floor.
- **`attention_backend: cudnn` does not exist on this card** — deterministic
  `RuntimeError: No available kernel` in ~1.3 s, both seeds. The pack
  recommends it as a 17 % speed-up.
- **`cfg_scale: 3.0` costs 65–74 % more**, not "roughly doubles"; **`cot:
  off` was not faster at all**, recorded as unexplained rather than
  rationalized.

**The music, judged by the people who can judge each half.** The executor
measured what a program can measure — duration, clipping, silence, DC offset,
channel balance, truncation — and refused to judge what it cannot hear.
4/6 generations were mechanically usable (the two failures produced no
audio). The developer then listened to all five delivered tracks and found
them a very good result for a locally run model: the baseline, both
`cot: off` tracks and `cfg_scale: 3.0` at seed 424242 calm, and
`cfg_scale: 3.0` at seed 909090 noticeably higher-energy. **5/5 by ear.**

That split is recorded rather than blurred, and so is what it does not
establish: one style, one lyric, three seeds, one card — and the energy
difference may belong to the seed rather than to `cfg_scale`, since the other
`cfg_scale: 3.0` track is calm.

## Interventions and deviations

**Observer needed one fix and no rescue.** No watch was reopened, no
notification was faked, no serving was restarted by hand.

**autolab broke its own contract twice in one serving**: it said it would
report before committing, then committed and resolved task 4 in that same
serving, though its introduction says a task is not closed until the
requester agrees. The result was good and its shape had been approved, so
nothing was harmed — but the acceptance gate is what makes a review mean
anything, and a task that closes itself removes it. Reported to autolab in
the mission topic; it is that agent's contract to fix, not this episode's.

**A worker's summary needed checking, not just a supervisor's.** The executor
recorded the Observer defect as a "poll-timing false alarm"; it was not, and
only reading the notification's own evidence showed it. The realm already
knew that a supervisor judging from relayed numbers is a risk. This is the
same risk pointing the other way.

## Cost

**Waiting cost nothing at all**: 16 evaluations on the host's local model, and
**zero requester runs consumed during waiting** — autolab's log shows one
serving per task start and one per notification, with nothing in between. The
paid runs were the ones doing the work.

`nctl drift` is converged=46, error 0, unchanged from p1: nothing was
deployed. pyagag 598 tests, agautolab 242, agfront 157, agobserver 63, all
green.

## What p3 should look at first

1. **The conjunction is patched by prose, and prose is not a guarantee.** A
   deterministic option — intake splitting a compound request into several
   watches, or refusing one — would remove the class rather than warn about
   it.
2. **A finished watch cannot be reopened.** `w6866`'s wrong `met` ended it;
   a requester woken too early has to notice and ask again. Nothing detects
   that today.
3. **Evaluation is still sequential, and now visibly so.** Two-job conditions
   took up to 120 s. Four such watches would make the promised minute a
   fiction, and nothing warns anybody.
4. **Observer still has no mention route**, which is the only reason
   autolab's replies — every one of them addressed `@**agobserver-agstudio1**`
   — do not loop. p1 said this; p2 walked past it a dozen times.
5. **Nothing prunes the topic workspaces**: 16 more look directories this
   phase.

*Deus Ex Machina note: the Omni Agent measured the GPU node's ComfyUI
environment, verified the artefacts, and appended the human listening verdict
to the study's `tips.md` — work an in-system agent could do, except that
nothing in the realm can install on the GPU node today. The developer
installed the YuE2 node pack by hand, which no agent here could have done.
Handoff candidates both.*
