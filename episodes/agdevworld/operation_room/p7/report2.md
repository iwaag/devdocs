# Step 2 — relay: a session is a topic

`routines.py` now discovers routines from `routine-<name>` and
`front-routine-<name>-<stamp>` (`RUN_TOPIC`, anchored on the trailing stamp,
so routine names may contain hyphens); the bare `front-routine-<name>` of the
old layout is no longer a routine topic and is read as any other
conversation. `run_topics_of()` lists a routine's run topics newest first by
stamp, `session_of()` turns one run topic into one session, and
`session_list()` filters ✔'d runs before taking the last three.

Session payload: `id` (the fire's message id, or the first held post's when
the topic was opened by hand), `topic`, `stamp`, `fire`, `origin`,
`origin_evidence`, `schedule_event`, `previous` (the topic the fire's
`Previous run:` names), `answer` (answered/acked/unanswered, as the routine
row has it), `resolution` (`resolved` or `open` from the topic's ✔ flag —
nothing else; `unknown`, `reopened`, the notice window, `Topic.notices` and
`is_resolve_notice` are deleted), `history` (`posts`, `post_limit`,
`bounded`, sentence), `chat` (the run topic whole, real posts oldest first)
and the tree (`nodes`, `truncation`, unchanged). `start_id`/`end_id` are gone;
the routine detail's top-level `chat_log` is gone with them, each session
carrying its own chat. `fire_of()` now returns the **oldest** fire line of a
topic: a second one pasted in later is a comment, not a new session. The
routine row replaces `fire_topic` with `latest_topic`, `runs` and
`open_runs`; the detail adds `latest_topic` beside `latest_fire`.

Sweep cost, the constraint: `_resync` decides per `#front` topic whether it
is *deep* — every `routine-` standing request, plus the newest `DEEP_RUNS`
(= `SESSION_LIMIT` = 3) run topics of each routine by stamp
(`newest_run_topics()`). Deep topics are read with `num_before=200` and read
even under ✔. Every other topic, a routine's older runs included, follows the
realm's rule: 50 posts while open, not read at all once resolved. So a
resolved run older than the newest three is in Zulip and not on the board,
and the session list's `history.note` says so. `keep_history` stays true for
every routine topic, so an older open run is a session with a 50-post window
and `bounded` set when full. A ✔ on any held run topic still arrives on the
event queue as `update_message` and flips its `resolved` in place (covered
by the ops test).

Measured on the live realm: `sweep_calls` 56 before the restart, 51 after —
the five ✔'d legacy `front-routine-*` topics are no longer read. With the
cap, the steady state is at most 3 deep run reads per routine per resync
regardless of how many runs accumulate.

`chat.py`: `allowed_topic()` accepts a known routine's standing topic or any
topic in the run-topic shape for a known routine (that is how `start`
opens one that does not exist yet); `start()` posts into
`run_topic(name, stamp)`, names the row's `latest_topic` as `Previous run:`,
returns `previous` instead of `first_fire`, and refuses a second start in
the same minute (it would land in the same topic). `inflight()` watches the
newest session's topic instead of a fixed fire topic. `server.py` needed no
change: `?resolved=hide` and `POST /routines/<name>/start` keep their shape.

Tests: 114 pass (test_routines fixtures moved to run topics; the p6
notice-window tests were replaced by one resolution test, one identity/
pointer test and the filter test; the ops detail test now flips a run with an
`update_message` event). The launchd relay was restarted on this code; the
board shows the eight routines with zero runs each, which is the expected
state until Step 4 fires the first run — the dashboard is not yet on the new
payload (Step 3).
