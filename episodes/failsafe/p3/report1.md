# failsafe p3 — step 1: current gaps and cleanup targets

## Observed state (2026-09-27, ~12:40Z)

| source | observation | age |
|---|---|---|
| `nctl status` | Nautobot 3.1.3 authenticated, 1 Celery worker, 0 pending jobs, submodules clean | live |
| `nctl drift --host agstudio` | converged; all 17 placements `applied` (agautolab, agfront, agforge, agobserver, archsage, cagent-api, …) | nodeutils dump 4.0 h old |
| launchd | all listeners, the gateway, forge service, relay, cagent-api running; `comfy-notifier` and `observation-refresh` exit 1 (not on this path, unchanged since before p3) | live |
| repositories | pyagag `3e9729e`, pj-agdev `b2cac8b` (agautolab `eb2c032`, agfront `bcba734`), all clean and pushed | live |
| trial aids | `agautolab/.local/faults/` and `agobserver/.local/faults/` empty; no `timing.json` | live |
| Observer store | 11 tracked requests, 1 held (o11711), 8 health entries, 42 incident records (13 rescued, 6 reported, 18 withdrawn, 3 finished, 1 dismissed, 1 cancelled), 2 open reviews | live |

The p2 report is accurate about versions and residue. Nothing was found
running that p2 left behind (the one `vite` process belongs to the
developer's terminal and is not touched).

## Reproductions from saved transcripts

### R1 — nested-fence truncation (p2 C, D)

The raw Claude Code output of the run behind #12328 (task 1 of m12295):

`````text
```ag-reply intent=response_request ask=confirmation
… Test output (`python3 -m unittest test_wordcount`):
```
Ran 207 tests in 0.207s

OK
```
````
`````

The mark was opened with **three** backticks, and the code block inside it
with a **bare** fence (no info string). The splitter closes a mark at the
first bare fence of at least its length, so the reply ended at "Test
output". The journal agrees (`reply_marked=1`, `reply_blocks=1`). #12338,
#12410 and #12419 are the same form. The splitter's fence support works
only when the mark is opened with four backticks **or** every inner fence
has an info string; the models did neither.

This form is ambiguous in the fenced representation itself: a bare fence
after reply text may be the close followed by the run's own notes (which
must never be posted) or a code block opening. No rule over the same text
can tell them apart, so a better heuristic is not a fix. The
representation has to change.

### R2 — Front's missing reply (p2 D2, #12509)

The run behind #12509 (Front run-1080, serving 493) did write a reply:

````text
```ag-reply intent=response_request ask=confirmation to=Omni Agent
**[Front] Task 12474#1 has its result … Please confirm mission m12474.**
…
```
````

`to=Omni Agent` is a name with a space. The opener's attributes must all be
`key=value` words, so the whole line was not recognised as a mark, and the
split reported "the output contains no ag-reply block". The repair run was
told exactly that, and wrote the same opener again. The failure line was
then posted as `intent=report end=12505`:

- the serving is `delivered` and the conversation reads as answered;
- the confirmation request to the requester was lost;
- nothing owed it afterwards: the next serving happened only because
  autolab's result arrived.

So the defect has two halves:

- **the splitter** discards a reply because of a malformed attribute. Its
  docstring promises the opposite ("a misspelt attribute never costs the
  answer");
- **the failure path** marks an unrepaired reply as a normal end.

### R3 — misplaced task agreement (p2 D)

- Front relayed the requester's task agreement into
  `workplan-soak-p2-d` (#12435) instead of the task topic.
- autolab's planning serving answered "Task 1 of m12379 is closed on the
  result in #12428" (#12439). No close-out ran, and the planner has no
  tool that closes a task.
- `agentchat accept` then refused (#12446, "still has unfinished
  task(s)").
- Front re-posted the agreement in the task topic (#12447), and it closed.

Two separate faults:

- the plan has no way to route or return a task agreement;
- the planner's reply claimed a record it did not have.

### R4 — health wait semantics (p2 B)

- `agag.execution.LiveExecution.event` sets `last_event_at` on **every**
  harness event, including `system` events and partial-message
  `stream_event`s.
- Claude Code emits `system task_started`-style events while a long Bash
  call runs. So `agag.health.probe` returned `running` (event within the
  window) for a five-minute quiet command, and `waiting` never happened
  live.
- The same effect would certify a hung tool call as `running` for as long
  as the harness emits housekeeping events.
- A genuinely open tool call is `waiting` for as long as the run's own
  deadline allows (autolab: 20 min). A repeated `waiting` never escalates.

### R5 — probe scheduling (p2 D, code reading)

`Monitor.health_candidates` calls `HealthProbes.probe` inline in the tick,
one probed unit after another, each bounded only by its own timeout
(10 s). There is no per-cycle budget. With N slow probes a look takes
about N × 10 s, and a `stopped` unit behind them waits for the whole look.
This has not been reproduced yet; step 3 measures it before deciding.

### R6 — reviews (p2 step 4, code reading)

- A `reported` incident that later moves (`Monitor.cleared`) is noted in
  its incident topic only. Its review occurrence still says
  "not recovered". This happened in p2 C: occurrence 1 of
  `review-autolab-uncertain` is unrecovered in the review, but the
  incident read "Moving again" at 02:14:33.
- Hypotheses and improvement candidates are per-kind constants
  (`review.py`). All three `uncertain` occurrences carry the same text,
  including the two caused by the injected `probe-fail` fault.
- The occurrences carry no marker saying they came from trial injection.

## Disposition table

| # | issue | evidence | owner | planned fix / cleanup | verification | open decision |
|---|---|---|---|---|---|---|
| 1 | reply truncated after a bare nested fence | R1; #12328, #12338, #12410, #12419 | pyagag `agag.reply` | Replace the fenced mark with a tag block (`<ag-reply …>` … `</ag-reply>`). Inside it, code fences are ordinary Markdown, paired by CommonMark rules. Update the reply guide, the repair prompt, the consumers' fixtures and autolab's fault texts. The old fenced form is removed; a run that writes it gets the repair. | R1's captured output as a regression case; step 6 trial with nested fences and text after them | — |
| 2 | malformed opener attribute discards the reply | R2 | pyagag `agag.reply` | Any opener that names the mark is the mark. Unusable attributes become `meta_error` and the reply is posted unclassified. `to=<name>` is not an id and is reported as such. | R2's captured output as a regression case | — |
| 3 | an unrepaired reply reads as a delivered, finished serving | R2 | pyagag `agag.topics` / serving journal | The failure line carries no `end=` and no `intent=report`; the serving is journaled as a failed reply that is still owed. A bounded reply-only repair is retried from the journal (no rerun of the run's actions), then the owners are told. | unit tests; step 6 trial "Front ends without a usable reply" | — |
| 4 | a task agreement posted in the plan | R3 | agautolab (plan serving), guides | Give the plan a way to route a requester's agreement to its task's close-out with the post as evidence, or return it for correction. The planner must not report a closure it has no record of. | tests; step 6 trial "agreement reaches the plan" | — |
| 5 | harness housekeeping events certify progress | R4 | pyagag `agag.execution` / `agag.health` | Record the nature of each event (work vs. housekeeping) and the age of the last work evidence separately. Only work evidence makes `running`. | unit tests; live `waiting` in step 6 | — |
| 6 | an unchanged wait is healthy forever (up to the run's deadline) | R4 | agobserver monitor | A bounded reassessment policy: an unchanged wait is reassessed with growing intervals and a ceiling, then investigated; explained long waits (a live child doing work) continue. Intervals from trial evidence. | tests with injected clock; step 6 stuck-wait trial | — |
| 7 | slow probes serialise the look | R5 | agobserver monitor/health | Measure first (step 3). If a `stopped` unit misses its target behind timing-out probes, add a cycle budget or concurrent probing. | step 6 trial "several probes time out" | — |
| 8 | a later recovery is missing from the review | R6; p2 C | agobserver `review` | Append the later outcome to the existing occurrence once (idempotent across passes and restarts), keeping the original escalation text. | tests with restart; step 6 trial | — |
| 9 | per-kind fixed diagnosis | R6 | agobserver `review` | An evidence-based assessment per occurrence (observed failure, plausible cause, confidence, missing evidence, candidate). It runs asynchronously and never delays handoff. Injected faults are labelled as such. Status words: reviewed / fix planned / fixed / accepted limitation. | tests; the p2 review topics annotated | — |
| 10 | plain conversations tracked forever | `tracked.json`: o8286, o8721, o11450, o11522, o11538, o11539, o11771 (age 11–78 h, only `awaiting_requester`/`ended` units) | agobserver `retain` | Define when a satisfied exchange leaves active tracking; unfinished work, owed replies and held requests stay. | tests; restart trial in step 6 | — |
| 11 | unbounded caches | 42 incident records, 30 execution records (kept 200), 8 health entries (TTL), `incidents-p1/` | agobserver, agautolab | Bound closed incident records and health entries; delete `incidents-p1/`. | restart trial in step 6 | — |
| 12 | p2 review topics | `review-autolab-stopped` (2), `review-autolab-uncertain` (3), both open | agobserver / Developer | Record findings, trial provenance and fix references in each; leave the ✔ to the Developer, or resolve only the disposable trial follow-up with that stated. | step 5 | the Developer's ✔ |
| 13 | p2 trial records | missions m12078–m12539 `done`; `front-failsafe-p2-*` conversations | trial residue | Confirm terminal; resolve any open trial conversation; keep the evidence in the reports. | step 5 | — |
| 14 | p1 trial records | m11804, m11895, m11866, m12016 `done`; o11771 still tracked (plain workplan, no mission) | trial residue | Retire o11771 from tracking via the fix in #10; resolve open trial conversations. | step 5 | — |
| 15 | o11711 / m11741 | `held.json` since p1; mission `started`, task 1 `executing`/open; the copy under `.local/missions/aisvgs/` is 6.2 GB | Developer | Stays held; step 5 writes a decision-ready summary. | step 5 | resume, cancel or keep (Developer) |
| 16 | m8519 (mediagen) | tracked; mission `started`; task 2 awaiting requester | Developer | Explicit disposition in step 5 from current records. | step 5 | likely a Developer decision |
| 17 | m9349, m9697 (robustp1) | tracked; `started`; m9349's task 1 awaiting delivery | trial residue (robust_workflow p2) | Probably disposable trial missions: check and cancel or record as done with evidence, without impersonating an acceptance. | step 5 | — |
| 18 | m6113, m7601, m7732 | no longer tracked (outside the window); m6113 has no `[state]` note, m7601 and m7732 are `started` | Developer / autolab | Explicit disposition in step 5 (terminal, active, replaced or held). | step 5 | possibly the Developer's |
| 19 | dependency pins | pyagag `f1e5973` in autolab, Front and Observer; `f5c4359` elsewhere | each consumer | Update the pin in every consumer that runs a conversational role (the reply fix reaches them all): agautolab, agfront, agforge, archsage, cagent. The relay and comfynotify only if they use a changed module. | each consumer's suite | — |

## Notes for later steps

- The reply representation (#1) is a realm-wide contract. Every
  conversational role gets the guide from its own pinned pyagag, so a
  consumer stays consistent until its pin moves. All five consumers move
  together in step 2.
- #3 and #4 need a close look at how `agag.topics` and autolab's plan
  serving produce and journal a reply before changing anything.
- Plane is gone; there is no old tracker to reconcile. The "old tracked
  backlog" is Observer's `tracked.json` and the missions above.
