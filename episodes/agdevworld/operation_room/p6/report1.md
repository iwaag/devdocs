# Step 1 — session identity and resolution

A session is identified by its fire message ID, and the relay now returns
that identity explicitly: each session in `/routines/<name>` carries `id`,
`start_id` and `end_id` (the next fire's ID, or null for the latest span), so
chat highlighting no longer has to derive a span from its neighbour in the
visible list. The one manual-activity session of a never-fired routine has
`id: null` and a span from message 0.

Origin is read from the fire's own wording, because both writers post as the
Developer. `fire_line()` in `routines.py` is the manual formatter for step 2;
it keeps the trigger's sentence and adds the mark "started by hand from the
operation room". `fire_origin()` reads that mark as `manual`, the trigger's
sentence without it as `scheduled`, and any other fire-shaped post as
`unknown`. A contract test substitutes the `text=` line of
`devenv/routine/trigger.sh` and checks it is recognised as a scheduled fire,
so rewording the script fails a test rather than the board. When the schedule
holds a fired event within 180 s of the fire post, the session names it as
`schedule_event`; the evidence sentence says whether wording, schedule, or
both were matched.

Completion semantics: `done` on the row still means the fire got an answer.
A session is `resolved` only by a human's ✔, and the ✔ is attributed to one
fire through Zulip's own "has marked this topic as resolved/unresolved"
notice, which is a system-realm post with a message ID that falls inside
exactly one fire span. The engine keeps those notices beside the history
(`Topic.notices`), never in it, so they are not speech and never reach the
chat. The rules, in `resolution_of()`: the newest notice inside a span decides
`resolved` or `reopened`; the latest session is decided by the topic's current
flag, with any notice cited beside it; an older span with no notice is
`unknown`, because the topic's flag today does not describe it. Storage is the
realm itself: the notices are re-read by every sweep and arrive live on the
event queue, so no local ledger was added and nothing has to survive a relay
restart in this process's memory.

Known limits, recorded before the filter is wired in step 3: the fire topic is
read as a window of 200 messages, and every session payload now carries a
`history` block (`posts`, `post_limit`, `bounded`, `fires`, `hidden_resolved`
and a sentence) that says whether that window was full. A resolve notice older
than the window is gone with the fire it belonged to. Zulip removes a notice
when a topic is toggled back within a few minutes, so a resolve/unresolve pair
can leave one notice or none; the latest span falls back to the current flag,
older spans stay unknown. On this realm the five retired routines each carry
one resolve notice, all posted on 2026-09-06 after their last fire, so their
last session will read resolved and their earlier sessions unknown.

`session_list()` accepts `include_resolved` and filters before taking the
last three; `sessions_of()` remains the unfiltered reading and `/inflight`
still attaches host observation to the actual latest fire. The detail payload
adds `latest_fire`, `history` and `filter`. Types were added to
`routineState.ts`; the dashboard itself is unchanged this step. 104 relay
tests pass (nine new) and `npm run build` passes. The running relay was not
restarted; it will be after step 2, when the write route is in place.
