# advance_mediagen_study p6 ex1 — report

Plan: `plan.md`. Step detail: `report1.md` (preflight and fire), `report2.md`
(the mission, task by task).

One fire, four tasks, all delivered. The exercise had two halves and both
answered.

**The headline is the flow, not the pictures.** *A ComfyUI job finishing is
enough to resume an agent that ended its run seven minutes earlier.* Measured
twice, from the backend's own clock and the listener's own log, with no human
and no supervisor post in between:

| | job finished | bot posted | second serving started |
|---|---|---|---|
| dataset A — jump | 12:53:31.15 | 12:53:32 | **12:53:32** |
| dataset B — idle | 13:11:08.51 | 13:11:10 | **13:11:10** |

**About one second, twice, unprompted.** That is the leg the notifier episode
never proved — its step 4 hit a race and its step 5 used a scratch topic. It
is proved now, on real `workrun-` topics inside a real mission.

**The second result cost more to learn and is about deliverables.** The two
datasets were finished, inspected and reported — and **never committed**. Every
frame, sheet and `meta.json` sat in the ignored `.local/out/` while the task
commits carried only prose. Nothing in the system noticed: not the run, not the
supervisor, not the run's own publication self-check, which was clean and was
right to be.

## What was asked, and what came back

| the plan asked for | state |
|---|---|
| Omni Agent preflight, four items | **done**, including the optional one |
| the 124-output question answered before firing | **done — and it was the bad case**; fixed and re-proved live |
| a new character and a new still, judged by eye | **done** — cat, 8 seeds, 7 rejected |
| dataset A, jump, through the callback | **done**, callback unprompted |
| dataset B, idle, through the callback again | **done**, callback unprompted |
| `cycle` / `whole` / `motion` extraction modes | **done**; `motion` won both, `whole` has not yet won a real dataset |
| datasets in the test repository | **done at the third time of asking** — see below |
| tips, INDEX, untracked-safe self-check, push | **done** — `main` `f0ca60d`, test repo `688d8bc`, `publish/` untouched |

**The exercise is closed.** Everything the plan asked for was delivered.

## The braindump's question, answered

**"Is the notifier's post enough to resume a serving?"** Yes, and quickly.

The p6 rule this exercise deliberately suspended — *"block in-run, never
background a clip"* — was correct when it was written and is correct for what
it was written about: **an agent cannot delegate its own continuation to a
process it spawns**, because ending the serving kills the child. It never
applied to the notifier, and the distinction is worth stating in one line:

> A **child process** dies with the serving. A **daemon** does not, and its
> post is a resume.

The suspension had to be spelled out twice in the fire, because the guide, the
previous topic and the standing text all still said "block". It survived two
relay hops verbatim and neither agent tried to correct it back.

**The measurement also retires the trade-off.** At ~1 s from job end to work
resumed, blocking in-run buys nothing but a held run slot. The whole latency
budget is the daemon's own 5-second poll.

**And Front's silence was the other half of the test.** Asked to *not act* for
seven minutes rather than to keep a run alive, it did, twice. p6's most
expensive event was a supervisor writing *"I'll keep watching"* and then ending
its run; this is the same moment with the opposite outcome, because nobody was
asked to watch.

## The preflight's finding, which was not the one the plan expected

The plan asked whether a 125-output record would be **truncated** or **refused**.
It is neither. **Zulip accepts it, truncates it silently, and every party
reports success.**

| | before | after |
|---|---|---|
| rendered message | 27 333 chars against a ~10 000 cap | **1 758 chars** |
| Zulip | accepted, `[message truncated]` appended | accepted whole |
| daemon log | `success posted` | `success posted` |
| what arrived | 46 of 125 entries, cut mid-token, **unparseable** | parses |
| `prompt_id` / `state` / `wall_s` / `vram_free` | **all lost** | present |

The record's keys serialise sorted, so `outputs` precedes exactly the four
fields a callback exists to deliver. **A refused post would have been safer** —
it leaves a `post_failed` line, keeps the ticket and retries. A truncated post
archives the ticket, writes `posted`, and resumes the receiving agent with a
healthy-looking record it cannot read.

Fixed in `comfynotify` (cap the list, carry `outputs_total`), re-proved live on
the same job, committed and pushed. **The preflight paid for itself here**: had
this fired unfixed, the first callback would have arrived corrupt and the
exercise would have measured the wrong failure.

Worth keeping for its own sake: option (b) as the plan phrased it — *"the
callback only needs the id"* — was **false at 125 outputs**. The full id never
reached the topic; only its 8-character headline prefix did.

## The datasets

`cat_jump_contact_4x.png`, `cat_idle_contact_4x.png` and a `preview.gif` for
each, beside this file. All opened before any number was read.

**One still, one character, two actions**, as the plan required so the
comparison stays clean. Cat, seed 88888, chosen from 8 candidates by eye —
7 rejected for four different reasons (three-quarter body, a grass patch
breaking the flat background, a sitting pose, a head at the top edge).

**The jump reads.** Stand, crouch, rear, airborne with all paws off the ground
and the shadow detached, land, settle — plus a dust puff the model invented.
**A one-shot non-periodic action came out of a single standing still and one
sentence.** The model was never shown a crouch or an airborne frame.

**The idle reads too, and is deliberately the opposite profile.** Eight frames
holding one stance while the tail curl, the ear and head angle and a few motion
flecks drift.

| | jump | idle |
|---|---|---|
| mean adjacent distance | **5.3296** | **1.9549** |
| total motion, 124 frames | 655.5 | 240.4 |
| full-clip closure ratio | 0.2598 | 0.8334 |
| palette entries | 32 | 23 |
| extraction mode | `motion`, chosen up front | `motion`, by fallback |

**Start = end held for a one-shot action**, which is the assumption the plan
flagged as never yet tested outside a gait. It held, and the run said the sheet
is what decided rather than the ratio.

### Extraction: the honest result is a negative one

`cycle` was **genuinely tried on the idle and rejected.** It found period 4 —
implausible for a breath-and-glance cycle — and its best window scored a
closure ratio of **1.6868**. Above 1 means the wrap distance exceeds the
window's own mean adjacent step: not a loop, just eight near-identical frames.
The run opened that sheet, saw the near-duplicates, and fell back.

**`motion` won both datasets for two different reasons** — by construction on
the jump, by fallback on the idle. **`whole` has not yet won a real dataset**,
and the run wrote that down rather than letting "three modes added" imply three
modes earned.

This is the third time in this episode family that a **score optimised without
a constraint it cannot express** has been the defect. p6 found it twice, in the
period picker and the stride sweep. Here the metric behaved — it returned 1.69
and the run believed it. The pattern to carry forward is not "closure ratio is
broken" but **"open the sheet first" is what makes the metric safe to use at
all**, and it was followed every time this exercise.

## The specification failure, and why it is the more useful finding

Tasks 2 and 3 both ended with *"write `meta.json`, commit"*, and both did
exactly that. Their commits contain `report.md` and `localtest.yaml`. **The
datasets themselves were in `.local/out/`, which `.gitignore` excludes.**

The instruction named **an act** and not **a path**, and the natural home for a
tool's output is the one directory git was told to ignore.

**The rule: when a deliverable is a file, name the tracked path it must exist
at.** "Commit" is not a location.

A second shape of the same failure followed. `cat_idle/meta.json` was missing
nine fields `cat_jump/meta.json` had, so the two datasets could not be compared
— which was the entire reason for one character and one still.

**And that gap had already been published, in writing, without anyone reading
it as a symptom.** The tips entry comparing the two motion profiles says the
idle's figure was quoted *"from this subject's internal run log since idle's own
`meta.json` only records the `motion`-mode fields."* **A published document
documented a workaround for a defect in the artefact it was describing.** That
sentence is a defect report; it was read as a citation. Nobody in the chain
treated it as a signal — not the run that wrote it, not the supervisor that
relayed it, not the self-check that passed it.

Both went back and both were fixed, in a third round.

## A standing fact that was false

The fire told autolab *"`comfynotify` is on your PATH"*, on the authority of
the notifier episode's report and the local environment notes. **It was not.**
`which comfynotify` returned nothing, and `AGFORGE_COMFYUI_URL` was unset —
so **the exact `comfynotify watch` line the fire supplied failed on first
use.** autolab found the binary at its absolute path and retried with the
variable set inline; the ticket was written 11 s later.

`agautolab.instance.extra_environment` does put that directory on `PATH`, the
directory exists and the binary is in it, so the grant is written and the value
is lost somewhere between there and the agent's shell. **Recording the fact and
not the mechanism** — p5's lesson was that a confident mechanism invented at
this point is what the next phase pays for.

The recovery took two attempts and eleven seconds, which is the system working.
But it recovered from an **Unexplained Chainsaw**: a tool it was told it had, at
a name that did not resolve. **The one open item that could still break this
flow for the next user.**

## The fire's own failure, and the rule it bought

p6's fire was one long payload and reached Front truncated mid-sentence. ex1
fixed that — four numbered posts under the length cap, each labelled "N of 4",
a sentinel on the last — and **introduced a new failure in the same place.**

Both listeners snapshot their chatlog when a run *starts*, so both were served
a partial payload:

- **Front** got post 1, found no sentinel, **refused to relay**, and asked for
  the tail.
- **autolab** got posts 1–3 and **held off writing `plan.md`** rather than
  guessing how tasks 2–4 divide.

**Both refusals were correct, and autolab's was unprompted** — it had been told
nothing about sentinels and reasoned it out from the posts being numbered.

> **A multi-post payload must be posted tail-first, or announced only once
> every part is up.**

Cost: one Front run and about six minutes. Cheap, and autolab's listener
recovered on its own (`reprocessing …: human posts arrived during the run`).

## What the agents did well

- **Both refused to act on a partial instruction**, one of them without being
  told to.
- **autolab held the mission in planning** and did not write `start.flag`,
  because nothing had greenlit it — its own contract, kept unreminded.
- **It proved its copied helpers unmodified with three empty diffs** against
  `422eecc`, rather than asserting it.
- **It judged the still by eye and used the pixel measurement only to separate
  close calls**, and re-checked headroom *after* padding — the number that
  matters is the one in the clip's frame.
- **It reported what it had not tested.** `cycle` untested in this repository;
  `whole` never having won a real dataset; `suggested_fps` labelled a judgment
  call rather than dressed up as measured. Three separate chances to overclaim,
  three declined.
- **It bounced a start signal that arrived out of order** (*"Please complete
  previous work (M-47)"*) rather than running task 2 on unfinished tooling.
- **Front stayed silent for two seven-minute waits**, which was the harder half
  of its job.

## Costs

| | runs | cost |
|---|---|---|
| autolab supercoder | 23 | $15.58 |
| autolab superdirector | 10 | $2.02 |
| Front | 49 | $9.99 |
| **total** | **82** | **$27.59** |

Plus about **15 GPU minutes** across two clips (435 s and 435 s) and roughly
two minutes of stills (11 renders at ~9 s).

**Against p6's $54.80 over 132 runs for a comparable four-task mission, this
is half the cost for the same shape of work** — and it included a mechanism
under test, two clips instead of three, and three sign-off rounds on task 4.
Front's 49 runs against autolab's 33 is still the ratio to watch, but it is
much closer than p6's 87-against-45, and the reason is visible: **there were no
stalls to notice, diagnose and restart.** The callback removed exactly the
failure class that generated p6's Front traffic.

## Deus Ex Machina

- **The preflight, and the `comfynotify` fix it produced.** The plan asked for
  the preflight, so it is not a handoff candidate; the fix is the Omni Agent
  maintaining a host tool. But note what it means: **the defect was in the
  notifier the whole time and the notifier episode did not find it**, because
  no step in that episode ever ticketed a 124-frame video graph.
- **Read `plan.md` and `task2.md` off disk rather than trusting the relay's
  summary of them.** Not a handoff candidate — this is the disinterested-eyes
  role, and it is the only way a fire's fidelity can be checked at all.
- **Opened every picture** — three preflight stills, the chosen still, both
  contact sheets at 4×. Not a handoff candidate.
- **Took all six timings from the backend's clock and the listener logs**, not
  from anyone's report of them. The reported numbers happened to agree.
  Not a handoff candidate; the measurement was the deliverable.
- **Caught that the datasets were never committed, and that the two
  `meta.json` files had different shapes.** **Handoff candidate, and the
  clearest workflow gap this exercise found.** Neither is subtle — one is
  `git show --stat`, the other is comparing two key sets — and neither the run,
  the supervisor, nor the publication self-check caught either. A run that
  verified its own deliverable existed at a tracked path would have caught the
  first without any new judgement.
- **Verified publication with a filesystem walk that never consults git's
  index.** Agreed with the run's `git grep --untracked`. Partial handoff
  candidate: the run's own scan is now good enough that the walk has agreed
  twice running.

## Closed

- **The callback leg.** A bot post resumes a `workrun-` serving, unprompted, in
  about one second, measured twice on real mission topics.
- **The suspension was right.** A daemon is not a child process, and blocking
  in-run buys nothing at ~1 s latency.
- **The 124-output question.** Answered, and it was the silent-truncation case;
  fixed and re-proved.
- **Non-gait extraction.** `motion` is the working default for anything not a
  confirmed repeating gait; `cycle` fails loudly and legibly on an idle.
- **A one-shot action from one still.** Start = end held for a jump.
- **Publication.** Both repositories pushed, verified two independent ways.

## Still open

- **`comfynotify` is not on the PATH autolab's runs actually see**, and
  `AGFORGE_COMFYUI_URL` is unset there. The grant is written in
  `agautolab.instance.extra_environment` and the value is lost downstream.
  Mechanism unknown, deliberately. **This is the item most likely to break the
  flow for its next user**, and the standing documentation says the opposite.
- **Nothing verifies that a deliverable exists at a tracked path.** The
  self-check looks for leaks in what *is* committed and cannot see what is
  missing.
- **A published document described a workaround for a defect in its own source
  artefact**, and no party read that as a symptom.
- **`whole` has never won a real dataset.** Implemented, unit-tested against
  synthetic frames, unproven.
- **The `[selfnote][served]` / listener bookkeeping did not log a "marked
  served" line for two of the task-4 servings** while the work itself completed
  correctly. Noted, not investigated.
- **Carried over from p6, unchanged:** agforge's production still-image path is
  still broken by the SwarmUI backend switch; the publication convention has
  still never been applied to the corpus (27 phase labels in
  `skeletalRig/summary.md`, deliberately left); the lavender background, the
  shared shadow/outline palette entry and the background ghost structure all
  survive into these sheets too.
