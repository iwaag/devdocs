# Step 1 — the naming contract, dispatcher and trigger

One run, one topic: `front-routine-<name>-<stamp>`, the stamp being the fire
line's own UTC minute (`2026-09-07T05:00Z`). The name is built in two places
from the same two pieces — `run_topic()` in `devenv/routine/dispatch.py` and
`run_topic()` in `agentroom/routines.py` — and read back by one regex,
`RUN_TOPIC`, which anchors on the trailing stamp so routine names may contain
hyphens.

`trigger.sh <name> [stamp] [previous_topic]`: the stamp defaults to now, the
dispatcher passes its own so the topic it records and the topic it posts to
are the same string. The fire text keeps its first sentence (`Routine
\`<name>\`, run of <stamp>`, what `FIRE_LINE` recognises) and the standing
request pointer (`TRIGGER_SENTENCE`, what `fire_origin()` reads as
scheduled), drops "this topic holds the earlier runs and my comments on
them", says instead "This topic is this run alone; resolve it (✔) when the
run is finished", and, when a previous run exists, adds `Previous run: #front
› \`front-routine-<name>-<prev>\`.` — the sentence `previous_of()` reads. Dry
run with a mocked `agentchat`:

    2026-09-07T05:00Z trigger papers -> #front/front-routine-papers-2026-09-07T05:00Z
    Routine `papers`, run of 2026-09-07T05:00Z. The standing request is the
    latest post in #front › `routine-papers`. This topic is this run alone;
    resolve it (✔) when the run is finished. Previous run: #front ›
    `front-routine-papers-2026-09-06T05:00Z`. Do it.

The dispatcher records the run in `schedule.json` under a new top-level
`runs` object, `runs[<routine>] = {topic, at, event}`, and copies `topic`
onto the fired event. The record is written **before** the action, in the
same marker commit as `fired_at`, so a crash between the commit and the post
leaves a record of a topic that may not exist rather than a post nobody
recorded; the previous record is what the fire callback receives and passes
to the trigger as its third argument. `load_schedule` tolerates a schedule
without `runs` (the live file has none yet) and validates the shape when it
is there. Nothing in the routine GUI on `:8093` reads the new key.

Relay side, contract only (Step 2 does the reading): `fire_line()` writes
the same new text with an optional `previous`, `PREVIOUS_MARK`,
`RUN_TOPIC`, `run_topic()`, `parse_run_topic()` and `previous_of()` were
added to `routines.py`. The contract test now also substitutes the script's
`topic=` line and its `Previous run:` line, so renaming the topic or the
pointer in the shell fails a test rather than the board.

Tests: dispatcher 7 pass (one new: a fire gets its own topic, names the
previous one, and the record lands with the marker); relay 115 pass. Nothing
was posted to Zulip and no service was restarted. Note that the dispatcher
launchd job reads `trigger.sh` from disk, so the next scheduled fire will
already land in a per-run topic; Front serves any `front-` topic, and the
relay reads that topic as a plain conversation until Step 2 restarts it.
