# progress_panel p1 — step 2: a shared progress read model

Date: 2026-09-27 (UTC 08:45–10:00).

## What was built

| layer | change | commit |
|---|---|---|
| pyagag `agag.trace` | a conversation its owner opens and serves itself (Front's `routinerun-`) reads `executing`/`awaiting_requester` from its own acks and ends — never `not_started` (G1); the `ag-routinerun` block (`ag.routinerun-finish.v1`) is the run's end record: `finished` (goal reached) or `ended`; `Node.records` carries `doc`, `acceptance`, `change`, `delivered`, `sagesync`, `state` and `finish` notes | pyagag `e7fa25d` |
| pyagag `agag.progress` (new) | the interpretation for people: one card per request, units with **work / execution / recovery kept apart**, a display state per unit, plan meters, run activity, stages | pyagag `e7fa25d` |
| archsage | `sage sync`/`attach` inside a serving writes `[selfnote][sagesync] <sage> <revision> project=… findings=…` into the conversation served (G3); outside a serving nothing is written | archsage `6929249` |
| relay | `GET /progress[?current=<desk id>][&fresh=1]`: discovery, bounds, caching, Observer's records, health checks | agdevworld `1009d8b`, pj-agdev `43fd232` |

Pins: pyagag `e7fa25d` in archsage and the relay. The other consumers
(Front, autolab, forge, Observer, cagent) move in step 5's deployment so
that `agentchat trace` and Observer read runs the same way the panel does.

## The read model (`agag.progress.v1`)

Each unit (a conversation in the request's tree) carries:

- `work`: the trace state, the record word (`note_state`) and its detail;
- `execution`: `serving` (`open`/`ended`/`unknown`, ack, end) and, only
  for probed owners, `health` — the verdict, why, `last_work`, the named
  wait, the observation time — with `evidence` = `confirmed` (checked
  within 120 s), `stale`, or `conversation` (no check exists);
- `recovery`: Observer's incident for that anchor (kind, state, open or
  reported-and-unrecovered), or a hold/retirement on the request;
- `display`: `{state, reason, next}` from `planning, queued, working,
  waiting, awaiting_you, answered, completed, cancelled, stopped, unknown`;
- `run`: activity for an open serving or a checked one — `active`,
  `waiting`, `stopped`, `ended`, `claimed` (conversation only, recent work)
  or `unknown`; `determinate` is always `null` (no record counts a run's
  units), plus the current action and its age;
- for a plan: `meter` = `{known, total, completed, working,
  awaiting_agreement, cancelled, revisions[{doc, total}], note, segments}`.

Rules that keep it honest:

- An open serving without a check is `working` only while the owner showed
  work within 30 min (`WORK_QUIET`), and its run band is `claimed`, not
  `active`. Past that it is `unknown`. o11711's task (dead for 19 h) and
  the unended routine run of o11522 now read `unknown` instead of
  "executing".
- A check about another ack is not applied (`health_note` says so); a
  check older than 120 s is `stale` and never animates.
- `answered` is its own state: a plain conversation has no end record.
  A card is `completed` only when every unit of work (plan, task, run,
  routine run) is finished by record and every stage is recorded.
- **Stages** stay pending until their record: `tasks_agreed` (x/y),
  `plan_accepted` (`[acceptance]`/`done`), `run_ended` (finish block),
  `report_delivered` (`[delivered]` in the origin), `knowledge_refreshed`
  (a `sagesync` note after the research's acceptance; study routines only,
  `routine-study-*`). A card whose units are all quiet but has a pending
  stage reads `waiting: not complete: …`.

## The relay's board

- **Scope**: `#front`'s `front-*` conversations looked at within 72 h, or
  kept because a board found them unfinished, or tracked/held by Observer.
  *active* = unfinished, held, or tracked and not completed; *recent* =
  finished within 24 h, at most 6; at most 16 cards; `current` (the desk
  conversation open in the view) always included and pinned.
- **Cost**: the trace reads the mirror (`MirrorReader`): a board costs no
  Zulip call — tested (`realm.calls` unchanged). Boards are reused for 5 s
  and while the mirror's revision is unchanged. On today's realm a board is
  16–20 traces in ≈1.5 s.
- **Health**: Observer's `health.toml` and the same command, only for open
  servings of the listed owners (autolab), cached per (owner, ack) 15 s,
  up to 8 in parallel within 5 s; a probe that fails or is late is
  `unknown` with the reason. No new configuration: Observer's directory is
  the parent of `AGENTROOM_MONITOR_HEALTH`, which the relay already reads
  (`AGENTROOM_OBSERVER_DIR` overrides).
- **Stale source**: a mirror that is not live marks every card `stale`
  and prefixes its reason `last known (…)`; the last board stays served.
- The route is a read: no write, no run, no Observer state is touched.

## Checked against the realm (mirror copy)

| request | card | why |
|---|---|---|
| o11522 study aisvgs | `unknown`; stages `run_ended`, `report_delivered`, `knowledge_refreshed` pending | the routine run's serving has been open 20 h with no finish record (G2); refresh only in prose (G3) |
| o11711 round 2 | `awaiting_you` — held by a person | Observer's hold; its task reads `unknown` (open serving, no work for 19 h) |
| o8512 mediagen p3 | `awaiting_you` — held | task 2's agreement is the Developer's |
| o11450 ctx trial | `awaiting_you` — #11472 asks you | a question to the Developer still pending (Observer retired it) |
| o13116 etc. (failsafe p4) | `completed` | every unit done by record, stages recorded |
| o13048, o12741 | `answered` | plain conversations |

Two readings were wrong in the first version and were fixed before
commit: a ✔'d conversation's old questions read `awaiting_you` (a ✔
closes them, as `agag.outstanding` says), and plain conversations whose
agent answered without naming the requester read "waiting on Front"
(they owe nothing; Observer's `satisfied` rule).

## Tests

| suite | result |
|---|---|
| pyagag | 981 passed before the new file; `tests/test_progress.py` 16 (recorded fixture `progress_p1.json`: o11522, o11711, o13116, text shortened, local paths removed) |
| archsage | 38 (new: a refresh inside a serving is recorded; none outside) |
| relay | 362 (new `tests/test_progress.py` 9: separate cards for two requests to one agent, probe once per TTL and within budget, failing probe → unknown, hold/incident on the right card, current pinned, no Zulip call, stale source, revision invalidates the cache, the route writes nothing) |

## Left for later steps

- G2 (a routine run can be left unended when its mission is accepted from
  another conversation) is shown, not fixed; step 5 will see whether the
  ordinary path ends runs.
- Only autolab is probed; Front's, archsage's and forge's servings stay
  conversation-only (labelled so).
