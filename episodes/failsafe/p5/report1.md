# failsafe p5 — step 1: the gaps, reproduced from the records

Date: 2026-09-27 (UTC). Nothing was changed in the running system in this
step: the gaps below are reproduced from progress_panel p1's recorded
trials, by replaying their messages through the current code, and by
tests that pin each gap as it stands. Those tests become the regressions of
the steps that fix them.

## Deployment checked first

- `nctl status`: Nautobot, worker, dumps and submodules ok. `nctl drift`:
  converged=46, 0 diffs.
- launchd: every listener (autolab, Front, archsage, forge, Observer,
  cagent), the autolab gateway, forge's service, cagent-api, the agentroom
  relay and comfynotify are running.
- Pins: pyagag `03f8fc9` in agfront, agautolab, agforge, agobserver,
  archsage and cagent; the relay on `29f3253` (the difference is
  `agag.progress` and docs only). Every repository is clean and level with
  its remote. The code is progress_panel p1's final state, so every
  observation below is about the deployed code.

## The evidence

The records are the realm itself (message ids below, read from Observer's
mirror). The reproductions are tests over a **synthetic** realm built in
code (`pyagag/tests/study_realm.py`: a Front Desk request, the routine run
Front opens from it, autolab's workplan, mission and task, archsage's
refresh topic — the post shapes the readers depend on, none of the real
text), so no realm content is published with the tests. A first attempt
exported the trial messages from the mirror into a fixture; publishing that
was refused as realm data leaving the host, and the export stays local
evidence only. `pyagag/tests/test_failsafe_p5.py` holds four
reproductions; two more live beside the code they are about
(`test_acceptance.py`, agfront's `test_runfinish.py`). All six pass against
the deployed code: every gap still exists.

## 1. A healthy serial queue is read as a stall (#13702 → #13721)

- 10:40:19 Front asked autolab to start m13679's task (#13702, trial B
  worldtrend). autolab's one executor was serving growbox's task
  (10:40:03–10:46:24).
- 10:45:54 Observer opened `incident-unacknowledged-13661`: "#13702 by Front
  not acknowledged for 5m", and asked Front for recovery (#13723).
- Front ran `agentchat recheck 13661 --after 13702`. It answered
  **STOPPED** for a post that was simply queued: recheck reads only the
  conversation. Front then "resumed" with another post (#13732) — a false
  intervention that happened to be harmless only because autolab's queue
  coalesces posts to one conversation.
- 10:46:55 autolab reached the post; Observer recorded the incident as
  **rescued**, and a developer review was opened for it
  (`review-autolab-unacknowledged`). Nothing had been rescued.

Reproduced (`test_baseline_a_healthy_serial_queue_is_read_as_unacknowledged`):
301 s after the start request `stall_candidates` yields `unacknowledged`
for exactly that post, while the other request's trace shows the same
owner's serving open. The rule has no queue input at all. The panel's `queue_behind`
(`agag.progress`) knows what the post waits behind, but only across the
panel's cards and only as conversation evidence. autolab's listener journal
(`listener.sqlite`: `pending` with `enqueued_at`, `servings`) holds the
confirmed fact — which entry is running and which wait — and
`agag.health` already reads that file read-only, but only for a serving
that has an ack.

## 2. Acceptance depends on the route, not on who decides

Both study guides say the runner accepts ("accept once the report and the
integrated `main` commit are in"; "The runner accepts once …").

- **B growbox**: Front agreed in the plan's conversation (#13798, "that
  completes the mission"). autolab's planning serving ran `agentchat accept
  13665 --evidence 13798` under its own credential: accepted, recorded as
  `[selfnote][acceptance] #13798 by 15 (Front)` (#13804). No human word.
- **B worldtrend**: Front's desk told the stand-in "I'm still waiting on
  your acceptance of mission m13679 … before I can record it" (#13796) and
  waited until #13801.
- **Code**: `accept_mission` refuses evidence written by the recorder
  ("#… is your own post"). Front as recorder of its own agreement is
  refused; autolab as recorder of the same post succeeds.
  `test_baseline_front_s_own_agreement_is_refused_to_front_but_accepted_from_autolab`
  pins it.
- **Guides**: Front's desk and front guides say "the mission's acceptance
  has to be the developer's own words", which contradicts the study guides'
  delegation. autolab's close-out tells the requester to record it with
  `agentchat accept`, which the requester (Front) cannot do with its own
  words.

The model has no notion of who holds the decision: "whose words" stands in
for it, and only the recorder is compared.

## 3. B worldtrend: a workplan beside its run, and a resume that served nothing

- 10:37:49 the desk serving (not the run) wrote Front's root note into
  `workplan-worldtrend-r1-onesided` naming the **desk** (#13661), posted the
  request, and only then opened the run (#13663), against the desk guide
  ("opening the run is the whole of your work").
- The run's first serving (#13687) found nothing it had delegated; it ended
  with "waiting". It held nothing, so Observer's `unheld` at 10:40:54
  (#13707) was **correct**
  (`test_baseline_a_workplan_opened_beside_the_run_leaves_the_run_unheld`:
  the plan is the desk's child, the run's holder is `none`, and a
  self-post "Resuming" leaves it so).
- Front's answer to it was a self-post into the run, "Resuming: …"
  (#13715), and 11 minutes later another (#13790). Front's own posts never
  serve its own conversation (the owner route skips a topic whose last
  speaker is Front), so neither started anything. The shared listener
  already has the mechanism that would: a `[selfnote][start]` note of the
  owner's own is enqueued and served (`agag.listen._intake`,
  `owed_start`), which autolab uses to start tasks. Front has no tool that
  writes one.
- Every later result came back to the desk (the workplan's root note), the
  desk recorded the acceptance and asked for the refresh, and nothing asked
  it to end the run until the stand-in's nudge (#13853 → `agrunfinish`
  #13855 at 11:08:51).

## 4. `agrunfinish` interrupted: the report is never delivered (confirmed)

`agrunfinish` writes the end record, then the report and `[delivered]` into
the origin, then resolves. A retry that finds the end record resolves the
run if needed and returns "already ended" — it never checks the delivery.
`test_baseline_an_end_record_without_its_delivery_is_never_delivered_on_retry`
drops the connection on the report's send: the retry exits 0, the run is ✔
and the desk never receives the report or the `[delivered]` note. The
hypothesis is confirmed by test; it has not happened live.

The listener's own path (`finish_run`) has the mirror-image gap: it
delivers the report into the origin during the serving and posts the end
record afterwards as the reply. A crash between them leaves a delivered
report with no end record, and the restart re-serves the run — a paid
rerun that may deliver a second report.

## 5. The fixed refresh topic returns every answer to the first request

- `study-growbox`'s guide: "ask archsage in the `study-growbox` topic of its
  channel". Front's root note there is the setup request's
  (#11548 → `front-desk-20260926-sagep2-growbox` #11539). `agentchat send`
  writes a root note once per topic and the rule keeps the earliest, so
  trial A's (#13503) and trial B's (#13808) refresh requests were both
  answered into the 2026-09-26 setup conversation (#13540, #13840 are Front
  serving them there), never into the run or desk that asked
  (`test_baseline_the_fixed_refresh_topic_returns_to_the_first_request`).
- worldtrend has the same shape one level down: trial A opened
  `study-worldtrend-refresh-r1`, trial B reused it, and B's answer was
  served in trial A's desk (#13841).
- `agentchat send` gives no sign that a topic already returns to another
  conversation.
- The record: `[selfnote][sagesync] <sage> <revision> project=<slug>
  findings=<n>` says neither which request/run it was for nor which result
  the revision had to include. The panel's `knowledge_refreshed` therefore
  matches by project and "after this run's acceptance".
  `test_baseline_the_refresh_stage_is_matched_by_project_and_time_only`
  shows a refresh of another revision, recorded for another run, completing
  a run's stage the moment its own mission is accepted.

## What the later steps start from

| gap | where it lives | step |
|---|---|---|
| queue read as stall; recheck calls a queued post STOPPED; "rescued" without a rescue | `agag.trace.stall_candidates`, `agag.trace.recheck`, `agobserver.monitor` | 2 |
| queue facts only in the panel, conversation-only | `agag.progress.queue_behind`; `agag.health` reads the journal only by ack | 2 |
| recorder compared instead of decision holder; guides forbid Front's delegated decision | `agag.acceptance`, agautolab close-out text, agfront guides | 3 |
| desk opens work beside its run; Front cannot resume its own run | agfront guides and tools; `agag.listen` start notes already exist | 4 |
| end record without delivery is never completed; listener delivers before recording | `agfront.runfinish`, `agfront.zulip_listener.finish_run` | 4 |
| one refresh topic per study; refresh record without run or revision relation | routine guides (archsage's), `agentchat send`, `archsage` `sagesync`, `agag.progress._stages` | 5 |

## Commits

- pyagag: `tests/study_realm.py`, `test_failsafe_p5.py`; the acceptance
  reproduction in `test_acceptance.py`.
- agfront: the interruption reproduction in `test_runfinish.py`.
