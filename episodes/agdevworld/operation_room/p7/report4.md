# Step 4 — cut over, verify, deploy, report

Cut-over: the three still-open legacy topics (`front-routine-ghtrends`,
`-papers`, `-publish`) were ✔'d as the Developer with `agentchat resolve`;
the other five already carried ✔. The relay was restarted: eight routines,
zero runs each, `sweep_calls` 48 (56 before this phase: every ✔'d legacy
topic is now skipped like any other resolved conversation).

First real run, through the dispatcher: a request `r14` and a due fire event
`e31` for `ghtrends` were committed to the local rtschedule clone; the
launchd dispatcher's 07:00:04Z tick pulled, marked, fired and pushed. The
schedule now carries `runs.ghtrends = {topic:
front-routine-ghtrends-2026-09-07T07:00Z, at, event: e31}` and the event
carries `topic`. The fire landed in that topic; the relay listed it from the
event queue without a restart (origin `scheduled`, `schedule_event` e31,
`open`, `acked` at first); Front's listener served the topic at 07:00:04Z,
acked, and answered at ~07:02 (it posted the mission to autolab in
`pj-ghtrends › workplan-trend4`). The run's tree from the notes is
`workplan-trend4` → `workrun-task1-g-9`, both keyed to the run topic — the
`[rootchat]` written by autolab names the run topic as its home, as
expected. One paid Front run, as authorised.

Resolution round trip: ✔ on the run topic (agentchat) flipped the session
to `resolved` on the relay within seconds via `update_message`; un-✔ (a
`PATCH messages/5044` back to the bare name) flipped it to `open` the same
way; no relay restart in between. After a restart, `sweep_calls` is 53
(the run topic and the conversations it opened), the run reads whole (5
posts, `answered`) and the tree is the same.

Dashboard against the real run, over CDP on `:5173`
(`.local/p7/step4/a.js`, screenshots `real-run.png`, `narrow.png`): the
card `Run 2026-09-07T07:00Z`, scheduled start, the resolution chip, the
chat header naming the run topic with the fire line as the first post, the
graph root as the run topic with the linked `workplan-trend4`; then a mocked
start (no real send: the pending card `Run 2099-01-01T00:00Z` and a result
naming the real run as the previous one), and a simulated relay outage —
health and every chip unknown, the session chip reading `? unknown (relay)`
rather than any verdict on the run, composer and start disabled — with the
run still selected and the chat draft retained after recovery. No real
start was posted from the board: the dispatcher run already exercised the
previous-run pointer path end to end through the same `fire_line` writer
(covered by the relay tests with a recording client).

Deployment: `docker compose up --build -d web` (HTTP 200 on `:8090`, the
served dashboard chunk carries "Run conversation"); the launchd relay is
running the Step 2 code. `nctl status` passes; `nctl drift` reports
`agfront: converged` with `liveness=polling` (the p6 `service_missing` is
gone) and an unrelated `agforge: agent_zulip_channel_unsubscribed` error,
not touched here. `agentroom/README.md` documents the run-topic layout, the
session fields, the deep-read rule and the start route.

What now reads wrong in older documents: `scheduled_routine` p1's "one topic
per routine, runs appended" and `trigger.sh`'s old "this topic holds the
earlier runs" sentence — both superseded by this phase; `agentroom/README.md`
still says "fire topic" in the link-notes section, which is now the run
topic and reads correctly with that substitution. Remaining limitation, by
design: a resolved run older than the newest three of a routine is not on
the board; it is in Zulip, and the history line says so.
