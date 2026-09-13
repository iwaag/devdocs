# observer p2 — step 3: waiting and continuation, in real work

## The chain, seven times, with nobody restarting anything

Every generation in this study went through the same route, and it was the
executing agent that chose to use it each time — the request named no agent,
no channel and no command.

| # | watch | what was waited for | looks | outcome |
|---|---|---|---|---|
| 1 | `w6800` | weights download + first generation | 1 | met, delivered 6804 |
| 2 | `w6825` | baseline run 1 | 2 | met, delivered 6831 |
| 3 | `w6839` | baseline run 2 | 2 | met, delivered 6846 |
| 4 | `w6866` | condition A, **two jobs in one watch** | 4 | met — **misjudged**, delivered 6873 |
| 5 | `w6882` | condition C, two jobs in one watch | 7 | met, delivered 6888 |

Five watches covered eight submitted jobs; two more chains ran through the
task-acceptance route rather than Observer. The complete trace of the first
one, which is the phase's headline:

| time (UTC) | event | evidence |
|---|---|---|
| 07:31 | executor submits `d44082a1…` to ComfyUI on the GPU node | history entry |
| 07:32:22 | opens `watch-yue2-weights-download`, anchored to its own run topic | 6795 |
| 07:32:43 | **executor ends its serving** | 6799, autolab log |
| 07:33:00 | Observer accepts `w6800` — **autolab is not served** | 6803, autolab log has no entry |
| 07:34:06 | Observer judges `met` on look 1 and delivers | 6804 |
| 07:34:06 | **autolab's listener serves `workrun-task1-m6770`** | autolab log |
| 07:35:08 | resumed run reports the collected result | 6809 |

No human and no Omni Agent restarted the executor at any point in the phase.
The job survived the serving that started it: at 07:33 there was no harness
child under the listener at all, while the download and generation ran on
another machine.

**Acceptance does not wake the requester.** p1 measured that for Front; it
holds for an executing agent too. autolab's log shows servings at 07:27:32
(the task start) and 07:34:06 (the notification) with nothing between them,
across Observer's acceptance at 07:33:00.

## Evidence Observer could actually read

The plan warned that a path on another node is not visible on Observer's
host, and this study was exactly that case: the executor runs on the Mac, the
model runs on the GPU node, and Observer runs on the Mac. The executor solved
it without being told how — it named **HTTP endpoints on the backend** rather
than files:

    GET http://<backend>:8188/queue
    GET http://<backend>:8188/history/<prompt_id>

Both are readable from Observer's host, and its `observe` role has `Bash`, so
`curl` was all it needed. That is the smallest thing that works, and no
status file or extra service had to be built for it.

## The condition was written for failure, not for success

Every watch in the study says some version of *"this is 'it has ended', not
'it succeeded'"* and then enumerates success, failure and stall. That wording
comes from the paragraph step 1 added to Observer's introduction, and it is
the difference between a crashed job waking the requester and a crashed job
leaving it waiting forever.

It paid off in condition B: both jobs failed in ~1.3 s with
`RuntimeError: No available kernel`. Nothing in that chain needed a person,
because "ended" covered it.

## The defect: a two-part condition judged on one part

**`w6866`, 08:35 UTC.** The condition was *"BOTH ComfyUI jobs have ended"*.
The look returned `met`, and its own evidence, in the delivered notification,
reads:

> …that job ended successfully. **BUT** GET /history/cc849181-… returns empty
> `{}` and it is still listed in queue_running…

The verdict contradicted the evidence beside it. p1 recorded 19 looks with no
incorrect judgment — every one of those conditions had a single target. The
first two-target condition in the realm produced a wrong answer.

**It was harmless by luck.** The second job finished during that 77-second
evaluation, so by the time the notification landed it was true. Had the job
been slower, the executor would have been woken to collect a result that did
not exist — and, worse, the watch was already finished and resolved, so
nothing would have woken it again.

**Diagnosis.** Not a code path: intake, delivery and cancellation all behaved
correctly, and the store shows a clean `met` after 4 evaluations. It is the
local model's judgment on a conjunction, and the `observe` guide said nothing
about conjunctions — it taught the `met` / `not_met` / `unable` distinction
and left "every part must hold" implicit.

**Fix** (`pj-agdev` `aba1a9c`): the guide now says that every part must hold
in the same look, that partial progress is `not_met`, that the parts are
enumerated one at a time in the evidence, and — the line aimed straight at
this failure — that a verdict must agree with its own evidence, because a
"but" in what you are writing has already answered the question. The guide is
read per evaluation, so no restart was needed.

**Repeat of the affected chain.** `w6882` was the same shape (two jobs, one
watch) and passed through the same state that had been misjudged: at 08:41:34
and again at 08:42:32 one job was gone from the queue while the other was
still running, and both looks answered `not_met`. Seven looks, six `not_met`
and one `met`, all correct. The condition was met only when both queues were
empty and both history entries existed.

**The tip that outlives the fix**: one watch per thing you are waiting for.
A conjunction is harder for a small model than two separate conditions, and
nothing about the design forces them together. That went into the study's own
tips as well as the guide.

## What waiting cost

- **Nothing in paid tokens.** Every evaluation ran on the host's local model.
  The paid runs in this phase were autolab's, and they are the runs that do
  the work rather than the waiting.
- **Requester runs during waiting: zero.** The evidence is autolab's log —
  one serving to start each task, one per notification, and none in between.
- **Evaluations are slower than p1 measured.** p1: 11–22 s, median ~15 s, all
  single-target. Here: 19–30 s for single-target conditions and **30–120 s**
  for the two-job ones, with the 120 s look reading two histories and a queue.
  Sequential evaluation is p1's known ceiling and this makes it sharper — a
  condition's own complexity, not just the number of watches, decides how
  often a watch is actually looked at.

## Known limits met live

- **The mention loop is still one missing route away.** autolab prefixes a
  reply with the last speaker, so several of its posts read
  `@**agobserver-agstudio1**`. Nothing happens only because Observer's
  listener registers no `on_mention`. p1 flagged this; p2 walked past it
  repeatedly.
- **A finished watch is finished.** There is no way to reopen `w6866` after
  its wrong `met`; the requester would have had to open a new watch. That is
  correct for exactly-once delivery and worth stating beside the defect.
- **Workspace growth is untouched**: one directory per look, and this phase
  added 16 looks across five watches.

## Interventions

One, and it was a fix rather than a rescue: the guide change between watch 4
and watch 5. No serving was restarted by hand, no watch was re-opened, no
notification was posted on Observer's behalf. The Omni Agent's other
involvement was supervisory — accepting tasks, and independently verifying
the artefacts rather than judging from relayed numbers, which is how the
`w6866` misjudgment was caught at all: the executor had recorded it as a
"poll-timing false alarm", and the correction was posted so its write-up
records the real cause.
