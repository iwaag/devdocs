# observer p1 — step 2: waiting, as short evaluations

## The trigger, and why it is a thread

Nothing posts when a file finishes downloading. `sweep_serve` reacts to
posts, and `on_sweep` fires on startup and queue re-registration — neither is
a clock. A watch served only by the listener's triggers is therefore
evaluated once and then never again, and no launchd change fixes that: a
plist restarts a process, it does not make one wake up.

So `worker.py` is a daemon thread started beside the listener, with its own
Zulip client (the skeleton already does the same for its DM route). Interval
is `AGOBSERVER_INTERVAL_SECONDS`, default 60 s, held as a fixed cadence — the
wait is what is left of the interval, with a one-second floor so an
evaluation that overran the whole interval cannot produce back-to-back ticks. Evaluation is sequential: one watch at a time, each
bounded, which is enough for this phase and makes "why did nothing happen for
two minutes" answerable from one log.

## What one evaluation is given

Four things, and the list is exhaustive: the accepted request (condition and
target), the previous observation, whatever evidence the run gathers itself,
and the observe guide. **Not** the watch topic's chatlog, not the other
watches, not the realm, not any development guide. That is not economy —
a small local model's accuracy falls off sharply when the prompt carries
material that does not bear on the question, and all of the above is material
that does not bear on the question.

The tools are `read`, `list` and `run` (a shell command), plus `agentchat` on
PATH for reading a Zulip conversation. The model decides how to look; there
is no predicate language and no per-condition plugin. `home` is deliberately
**not** passed to the run, so an evaluation has no conversation and cannot
speak in one — the guide says so, but the reason it is true is that nothing
hands it one.

## Three answers, not two

```json
{"verdict": "met" | "not_met" | "unable", "evidence": "what was actually seen"}
```

`met` and `not_met` are judgments about the world. **`unable` is a statement
about the look** — the path was unreachable, the command failed, the
conversation could not be read — and keeping it separate is the one
distinction that matters here: a target that cannot be read reported as
`not_met` tells the requester "not yet" forever.

Every way an evaluation can go wrong comes back as `unable` with a reason:
a non-zero exit, the deadline, a run that wrote no readable verdict. It never
raises, so one bad look never removes a watch from the schedule.

Bounding is doubled: `run_role(timeout=180 s)` kills the subprocess, and
agcode is handed `--deadline-s 160` and `--max-turns 12` so it reports its
own deadline as a result document rather than being killed mid-turn with
nothing to show.

## Routine polling is silent

No post per look. The exception is a **streak**: at the third consecutive
`unable`, and only then, one line goes into the watch topic saying the target
cannot be read and that this is not an answer about the condition. One
transient failure is not news; a line per interval would make the topic
unreadable.

## Cancellation, checked twice

A resolved topic is a cancelled watch. Resolving *renames* the topic, so the
question is whether the bare name is gone and the `✔ ` one is there — and an
**unreadable channel answers "no"**, because a lookup failure read as a
cancellation would silently drop every watch in the realm on one Zulip
hiccup.

It is checked before evaluating and again immediately before notifying. The
second check closes this specific window: the evaluation took real seconds,
and a ✔ that landed inside them means nobody wants the notification any more.

## Recovery does not need a post

The schedule is the store, and the store is rebuilt from the channel — at
worker start and every tenth tick. Losing `.local/watches/` loses memory,
never work: an active watch nobody has spoken to since it was accepted is
found again by reading, and its first evaluation afterwards simply has no
previous observation to compare against.

A met watch that has not been delivered carries `pending_notification` rather
than a new state, because delivery is a separate retryable step (step 3): a
met watch whose notification failed is still met and must never be re-judged.

## Live verification

Two watches, both accepted through the real intake, both evaluated by
`qwen3.8:27b-mxfp8` through agcode.

| check | result |
|---|---|
| an unchanged watch is reevaluated without new messages | `w6676` reached **5 looks** with no post in its topic between them |
| independent watches make progress | one tick evaluated `w6676` and `w6691` in turn, 33.8 s total |
| a failed read is not reported as success | `w6691` (a permission-denied path) answered **`unable`** three times running — never `met`, never `not_met` |
| the failure is said once | at streak 3, one line in the watch topic; nothing at streaks 1, 2 or 4 |
| an empty queue causes no inference | tick over an empty store: **0 model calls, 0 Zulip calls** in the tick itself |
| a met watch is recorded and never re-judged | controlled observation: `pending_notification` set, two further ticks judged it **0** times |
| cancelling stops the schedule | ✔ on `watch-unreadable-target` → `w6691` `cancelled` on the next tick, with no evaluation spent |
| recovery needs no fresh post | `.local/watches/` deleted entirely; the next reconcile recovered `w6676` as `active` from the channel, and did **not** recover the ✔-cancelled `w6691` |

Evaluation cost and duration, real looks: **11–22 s each**, median about
14 s, zero paid tokens. `not_met` evidence was concrete every time — "`ls -l`
shows only `release.zip.part` (8 bytes, unchanged since last look);
`test -f …/release.zip` returned MISSING" — which is the sentence a
notification would have to stand on.

## Judgment quality, observed

The model honoured the part of the condition that says *how to tell*: the
`.part` file present was `not_met` on every look, never "a file is there so
it is probably done". It also declined to guess through a permission error
rather than inferring absence from it. Both are the failure modes this
three-valued answer exists for, and neither occurred.

One thing the previous-observation slot visibly earned: by look 5 the
evidence had become "unchanged since last look", which is the model spending
the look on what changed rather than re-deriving what it knew.

## Not yet done

Delivery. A met watch is currently recorded and held; step 3 gives the worker
a delivery route, re-checks the ✔ one last time, and finishes the watch
visibly.
