# Step 3 — the scheduled investigation run

A one-shot event on the dispatcher's schedule fired `study-realworld` with no
payload; the run chose **investigation** from the standing default, reused
the recipes step 2 recorded without rediscovering any of them, produced two
new reports and one honest "nothing new", and pushed one commit. Two real
workflow defects surfaced, both recorded below rather than papered over.

## Getting it on the schedule

Asked Front, as the Developer, in `#front › front-schedule-studyrealworld-p1`
(message 5385) to add one request and one one-shot fire and **not** to fire
it itself. Front used its `rtschedule` tool and answered 29 s later
(message 5387, `front/run-0565`, 28.1 s, **$0.1428**):

- request **`r15`** — *"study-realworld p1: one scheduled investigation
  fire, to prove a scheduled run reaches real work."*, `until`
  2026-09-08T18:44:16Z
- event **`e32`** — `kind: fire`, routine `study-realworld`, `at`
  2026-09-08T16:47:00Z, chosen so the five-minute dispatcher tick could
  still take it

The dispatcher fired it at **16:47:02Z** — `fired_at` and the run topic
(`front-routine-study-realworld-2026-09-08T16:47Z`) are both written back
into `schedule.json`, and `runs["study-realworld"]` now names that topic so
the next fire can point at it. This is the first time `study-realworld`
appears in `runs` at all.

**A scheduled fire carries no mode**, exactly as the plan predicted: the
trigger line says only *"The standing request is the latest post in #front ›
`routine-study-realworld` … Do it."*

## The default flipped, on evidence

Front's first post in the workplan topic:

> `main/sources/INDEX.md` now lists three sources with verified retrieval
> (fed-fomc-minutes, pew-research-reports, reliefweb-updates), so per the
> routine spec this run is **investigation**, not discovery.

Same rule, same file, opposite answer to step 2's — decided by what the
catalog actually contained, not by anything in the fire. That is the whole
of what the "empty catalog starts with discovery" clause was for, and both
sides of it have now been exercised live.

Front also passed on the anti-duplication instruction unprompted: *"do not
just re-fetch the same document already reported on 2026-09-09-001 unless
there is something new to check."*

## What autolab planned and did

Scope it chose for itself, with no topic in the request:
**migration and displacement, across all three sources.** Its plan reply
said so before starting, including the prediction that one source would come
back empty:

> `fed-fomc-minutes` is re-checked but expected to honestly report "nothing
> new" (its most recent minutes are still the July 28–29 meeting; the next
> meeting, Sep 15–16, hasn't produced minutes yet — those lag ~3 weeks).

It started without a second round trip on the pre-approval, opened
`S4-3` / sub-work `S4-4` and `work-s4-3 › workrun-task1-s4-3`, and Front's
post there started it at 16:49:32Z.

**Result — `main` commit `c74c658`**, pushed, 4 files, 218 insertions:

| source | outcome |
|---|---|
| `fed-fomc-minutes` | **nothing new** — no minutes newer than July 28–29, 2026; no report written |
| `pew-research-reports` | `2026-09-09-002` — U.S. refugee admissions in FY2026 (short-read, 2026-08-17) |
| `reliefweb-updates` | `2026-09-09-002` — IOM Yemen Flow Monitoring Registry, migrant arrivals/departures, August 2026 |

**The acquisition routes were reused, not rediscovered.** Both new reports
were fetched through the recipes recorded on the first run — the Pew dated
report-page URL, the ReliefWeb RSS feed — and neither source file needed a
correction, so none was made. Nothing in the run re-derived a URL pattern
that was already written down.

**The reports carry what the routine asked for.** Each opens with
`Follows: [2026-09-09-001](../2026-09-09-001/report.md)` — the link to the
investigation it follows — and each ends with a *Citation metadata* block
(title, publisher, author, publication date, period covered, version,
retrieval time, relevant sections, URL) and an *Open questions* list. The
publisher/interpreter separation is done by sentence construction rather
than by a heading: the findings read *"Pew reports that refugee admissions
… have fallen sharply"* and *"Pew notes these claims 'have been disputed'"*,
and the interpretation is fenced into its own paragraph beginning *"This
suggests that…"*. That is the convention working as intended.

**The empty check was not manufactured into a report.** No
`reports/fed-fomc-minutes/2026-09-09-002/` directory exists. See finding 1
for what it *did* cost.

## Cost and timing

| run | what | duration | cost |
|---|---|---|---|
| `front/run-0565` | schedule edit (before the fire) | 28.1 s | $0.1428 |
| `front/run-0566` | fire → workplan | 47.9 s | $0.2698 |
| `superdirector/run-0163` | planning | 68.9 s | $0.2241 |
| `front/run-0567` | start the task | 35.5 s | $0.1910 |
| `supercoder/run-0228` | the investigation itself | 16:49:32 → 16:53:34 | **$1.0925** |
| `front/run-0568` | read the result, report | 12.3 s | $0.1028 |
| `front/run-0569` | finish the resolves (finding 2) | 19.3 s | $0.1178 |

**$2.14 for the scheduled run**, or $2.00 without the schedule edit. Fire to
completion was 16:47:02 → 16:53:48, **6 min 46 s**; the resolves landed at
16:55:45 after a nudge.

It is cheaper than step 2's discovery run ($2.71) and the difference is
almost entirely the task itself — $1.09 against $1.50 — which is what
"the route is already written down" is worth in this workflow.

Notably the investigation needed **one** serving of its workrun topic where
discovery needed two: autolab did the work, wrote its report, marked S4-4
Done and resolved the topic inside `run-0228`, because Front's start message
told it to proceed straight through to completion.

## Findings

**1. An empty check leaves no trace in the repository.** The standing
request says to record a no-update result honestly, and the run did — in its
Zulip report and in the commit message (*"fed-fomc-minutes re-checked: no
minutes newer than Jul 28-29, 2026 — nothing to add"*). But
`reports/fed-fomc-minutes/INDEX.md` is **byte-identical to before the run**,
and no source file's *last verified* line moved. A later run reading that
index cannot tell that the source was checked on 2026-09-09 and found
unchanged from the case where it was never looked at — which is precisely
the distinction the routine needs to avoid re-checking the same thing.
Front's own completion report also overstated it, saying *"Both source
indexes and per-source report indexes were updated"* when only two of the
three were.

The routine's request says *what* to record but never says *where*. The fix
is one sentence in the standing request — a no-update check gets its own
`INDEX.md` row with the result "no change" and the source file's
last-verified date is bumped whether or not a document was found — and it is
deferred to whichever run next touches the request, because correcting the
standing text mid-phase would have muddled which version step 3 was actually
run against.

**2. Front announced two resolves and its run ended before doing them.**
Message 5409 said *"I'll resolve the run topic … now"*, and neither
`front-routine-study-realworld-2026-09-08T16:47Z` nor
`pj-studyrealworld › workplan-investigate-realworld` was resolved when the
run ended. One Developer post naming the gap (message 5411) brought Front
back and it resolved both within 20 s (`run-0569`, $0.1178). The work itself
was complete and pushed before this; what failed was only the bookkeeping —
but an unresolved run topic is what the operation room reads as "still
running", so the failure is not cosmetic in that view. This is the ordinary
"an ack is not an answer" shape from `operation_room` p3, one step further
along: *a stated intention is not an action either*.

**3. The stray-twin pathology reproduced.** `#front` now holds both
`✔ front-routine-study-realworld-2026-09-08T16:34Z` and a bare-name twin of
it, containing exactly one message: Front's own
`[selfnote][served] work-s4-1/workrun-task1-s4-1 5378`, written *after* the
resolve. A post under a resolved topic's bare name opens a new topic rather
than reopening the resolved one, and a selfnote is still a post. Known
behaviour, reproducing on a new routine; recorded here, not fixed here,
because the fix belongs in the selfnote write path rather than in this
project.

## Deus Ex Machina

- Asked Front to edit the schedule, and posted the nudge that made it finish
  its resolves, as the Developer. The nudge is the honest kind of
  intervention: it named the gap and asked the agent to close it rather than
  closing it from outside — *did the noticing for agent Front — handoff
  candidate* (an agent that says it will resolve a topic should verify it
  did before ending its run).
- Nothing in `main/` was written or corrected by the Omni Agent.
