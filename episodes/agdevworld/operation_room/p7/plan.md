# Operation room p7 — one topic per run

Give every run of a routine its own Zulip topic, so that a session *is* a
topic: its ✔ is the session's resolution, its history is the session's chat,
and `unknown` disappears from the board.

This is a destructive phase in a private experimental environment. Backward
compatibility is unnecessary: the eight existing `front-routine-<name>` topics
are retired, not migrated. Refactor APIs, components and data structures as
useful; the steps describe outcomes, not mandatory implementation recipes.
UI text is English. Main implementation: `pj-agdev/agdevworld` (relay and
dashboard) and `pj-agdev/devenv/routine` (dispatcher and trigger).

## Why

`scheduled_routine` p1 chose "one topic per routine, runs appended" and
noted the alternative, `front-routine-<name>-<date>`, as "the simpler choice
if history gets long" (p1 `plan.md` l.87). p2 later recorded that
new-topic-per-run "proved cleaner in both phases" for autolab. p6 then spent
a step reconstructing sessions from fire spans and resolve notices inside one
topic, and the result is honest but unusable: any run that was not ✔'d
before the next fire is `unknown` forever (12 of the 17 sessions the relay
shows today). A topic per run makes the Zulip ✔ mean exactly one session.

## Decisions already made

- **Resolution stays human.** A run topic is resolved only by a person's ✔.
  Nothing auto-resolves the previous run when the next one fires; an
  unresolved run staying visible on the board is the point.
- **Front gets the previous run by pointer, not by shared history.** The fire
  text names the previous run's topic; Front reads it with `agentchat` if it
  wants to. Developer comments on a run go into that run's topic, never into
  `routine-<name>` (its newest post is the standing request).
- **Naming:** `front-routine-<name>-<stamp>`, stamp = the fire line's own
  `YYYY-MM-DDTHH:MMZ`. Zulip topic names are capped at 60 characters; the
  longest current name (`front-routine-imgprompt-2026-09-07T04:30Z`) is 41.
  Two fires of one routine inside the same minute are not a case to support.

## Step 1: the naming contract, dispatcher and trigger

- Define the run topic name and the fire text in one place the relay and the
  trigger both follow. Today `trigger.sh` hardcodes the text and
  `routines.fire_line()`/`FIRE_LINE`/`TRIGGER_SENTENCE` re-recognise it; a
  contract test in `agentroom/tests/test_routines.py` substitutes the
  script's `text=` line. Keep that shape or replace it with something the
  script and relay share; either way, one test must fail when they drift.
- New fire text: keep `Routine \`<name>\`, run of <stamp>` as the first
  sentence (that is what the relay recognises), keep the standing-request
  pointer, drop "this topic holds the earlier runs", add
  `Previous run: #front › front-routine-<name>-<prev>` when one exists.
- `dispatch.py` owns the `schedule.json` rewrite and calls
  `trigger.sh <name>` (l.247). Record the last run topic per routine there
  (e.g. `last_topic` on the request, or a small `runs` map) and pass it to
  the trigger as an argument or env var. The relay reads `schedule.json` as
  a local file already, so it can show the same value.
- Update `devenv/routine/tests` for the dispatcher change; run `trigger.sh`
  once by hand into a throwaway routine or with a mocked `AGENTCHAT` and
  show the topic name and text in the step report.

Pointers: `devenv/routine/trigger.sh`, `devenv/routine/dispatch.py`,
`agentroom/src/agentroom/routines.py` (`FIRE_LINE`, `MANUAL_MARK`,
`TRIGGER_SENTENCE`, `fire_line()`, `fire_origin()`). The launchd job that runs
the dispatcher every five minutes needs no change if the script path stays.

## Step 2: relay — a session is a topic

- Routine discovery: `routine_name()` must strip the stamp; `is_routine_topic`
  must accept the stamped form and keep `routine-<name>` as the standing
  request. Decide whether a bare `front-routine-<name>` is still a routine
  topic at all — after Step 4 it is legacy, and ignoring it is fine.
- `session_list()`/`sessions_of()`: enumerate the routine's run topics,
  newest first by the fire's message id, take the last `SESSION_LIMIT` after
  filtering. `id` = the fire's message id still works; `start_id`/`end_id`
  become trivial (whole topic) — keep or drop them as the dashboard prefers.
- `resolution_of()` collapses to the topic's `resolved` flag: `resolved` or
  `open`. Delete the notice-window logic, `Topic.notices`, `reopened` and
  `unknown` unless something else uses them. A run topic with no recognised
  fire line (somebody opened it by hand) is still a session; say so in its
  origin.
- `chat_of()` and `/chat`'s topic check: chat is per session topic. The
  dashboard's ordinary continuation posts into the selected session's topic,
  not "the routine's fire topic". Decide what continuation means when the
  selected session is resolved (posting into a ✔ topic is allowed by Zulip
  and Front's sweep skips ✔ topics — say that in the UI or refuse).
- `chat.start()`: create the new topic `front-routine-<name>-<stamp>` and
  return its name; `first_fire` is no longer interesting. The manual mark and
  the previous-run pointer are the same as the dispatcher's.
- `/inflight` and host evidence attach to the newest run topic.
- **Sweep cost.** `ops.py` l.624 reads *resolved* routine topics too, deep
  (`ROUTINE_HISTORY` = 200) — the exception that was fine for sixteen topics.
  With a topic per run this grows by one deep call per run at every resync
  (the "183 calls" comment). Bound it: read only the newest N run topics per
  routine deep, and older/resolved ones shallow (`TOPIC_LOOKBACK`) or not at
  all; the event queue still delivers a ✔ on an old topic as `update_message`.
  Report the chosen N and the sweep's call count before/after.
- The child walk (`children_of`, `[served]`/`[rootchat]` notes) is keyed per
  topic and needs no change; a `[rootchat]` written by a remote now names the
  run topic, which is exactly the session.

Pointers: `routines.py` (`routine_rows`, `session_list`, `resolution_of`,
`chat_of`), `ops.py` (`_resync` l.611–650, `routine()`, `is_resolve_notice`),
`chat.py` (`start`, `check`), `server.py` (`_start`). Bare-topic keying
(`room.bare_topic`) already follows the ✔ rename, so a run topic keeps one key
across resolve/unresolve.

## Step 3: dashboard — session = topic

- Session rows: title from the fire stamp (and origin mark), state is
  open/resolved plus the run's answer state from the row. Remove `unknown`
  from the session vocabulary and the legend; keep `unknown` for relay health.
- Chat panel: shows the selected session's topic whole; the amber bounds
  line and `history.bounded` become the rare "this run has over 200 posts"
  case. Show the topic name so the user can find it in Zulip.
- Show resolved: unchanged in meaning, now backed by the topic flag. The
  empty state when every recent run is resolved stays.
- New session: after `chat.start()` returns, select the new topic when the
  event queue delivers it (as p6 did with the fire id). The instruction field
  and pending-state behaviour stay.
- Show the previous-run pointer when a session's fire carries one (a link
  that selects that session if it is in the list).

Pointers: `src/routineState.ts`, `src/operationDashboard.ts`,
`src/chatPanel.ts`, `src/sessionGraph.ts`, `src/operationParts.css`.

## Step 4: cut over, verify, deploy, report

- Retire the eight legacy topics: ✔ each `front-routine-<name>` in Zulip (the
  Developer account, `.local/zulip/developer.env`). Their history stays in
  Zulip; the board no longer reads them. `mediagen` is fired by hand and its
  legacy topic is the busiest — future manual runs start from the board.
- Restart the relay; confirm `/routines` shows the eight routines with zero
  or one session each, and that the sweep call count is what Step 2 reported.
- Fire one real run through the dispatcher (a cheap routine, `localtest`
  if it is un-retired for this, otherwise `ghtrends`) and watch: the topic
  appears with the expected name, Front acks and answers in it, `[rootchat]`
  from a remote (if any) names the run topic, the board shows it as the
  newest open session, ✔ in Zulip turns it resolved on the board without a
  relay restart, un-✔ turns it open again. One paid Front run is authorised
  for this; more only if the first fails.
- Start one session from the board (mocked send in tests, one real send
  allowed if the dispatcher run above did not exercise the previous-run
  pointer).
- Relay tests and `npm run build`; CDP pass over the assembled UI as in p6:
  session selection, chat continuation into the run topic, Show resolved,
  compact/detailed graph, outage/recovery, 390-px viewport. Rebuild the web
  image (`docker compose up --build -d web`, `:8090`) and restart the launchd
  relay. `nctl status`/`nctl drift` for service state; the known stale
  `agfront: service_missing` pattern from p6 is not a blocker.
- Update `agentroom/README.md` (topic layout, session fields, start route)
  and `routines.py`'s module docstring. Commit/push. The step report records
  the sweep cost numbers, the topic name of the first real run, and anything
  from `scheduled_routine` that now reads wrong.

## Minimal constraints

- Use the Developer write path for fires and chat; keep the observer
  credential for observation. Do not commit credentials or machine-specific
  paths.
- No auto-resolve of run topics by any service. No repeated Zulip sweeps for
  UI refresh; the event queue is the refresh.
- The relay must not read every ✔ run topic deep at every resync.

Everything else — module boundaries, API shape, what happens to the deleted
notice logic, visual design — is the implementer's choice. Prefer the
smallest coherent solution; most of this phase is deletion.
