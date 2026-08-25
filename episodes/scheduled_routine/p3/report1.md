# scheduled_routine p3 — Step 1 report

Executed 2026-08-25 UTC.

## Dispatcher and repository

- Created the public local Gitea repository `autodev/rtschedule` and its
  ignored clone at `pj-agdev/.local/rtschedule/`.
- Initialized `schedule.json` as the planned `requests` / `events` object.
- Added `pj-agdev/devenv/routine/dispatch.py`. It pulls with rebase, validates
  the whole schedule, ignores future/already-fired/expired-request events,
  and prunes events more than seven days in the past.
- For every due event it writes `fired_at` atomically and makes a local commit
  **before** the Zulip action. It pushes that commit after the action. A crash
  after posting but before pushing therefore leaves a local commit which the
  next `git pull --rebase` preserves; the event is not posted twice.
- `fire` calls the existing `trigger.sh`. `decide` posts the event id, its
  `ask` verbatim, and `autodev/rtschedule, schedule.json` into
  `#front` / `front-schedule` as the Developer. The dispatcher contains no
  decision logic.
- Added `com.agdev.routine-dispatch.plist.in`, with `StartInterval = 300`,
  `RunAtLoad = true`, and the ignored log
  `agfront/.local/out/dispatch.log`. Installed and loaded it; its first
  service tick logged `2026-08-25T02:47:24Z no due events` and exited 0.

## Tests

`python3 -m pytest -q devenv/routine/tests/test_dispatch.py`:

```text
..                                                                       [100%]
2 passed in 0.06s
```

The fixture contains a due `fire`, due `decide`, future event, expired-request
event, and already-fired event. It asserts that only the first two actions run,
and that each action observes its persisted marker before it runs. A second
test pins the seven-day pruning boundary. `plutil -lint` also reported the new
plist `OK`.

## Manual due-fire tick

Added request `r-step1-manual` and due event `e-step1-manual` for `rtnotes`,
then ran one dispatcher tick at 02:46:38Z. Output:

```text
2026-08-25T02:46:38Z marked e-step1-manual before action
2026-08-25T02:46:38Z dispatched and pushed e-step1-manual
```

The schedule repository history is:

```text
72174a2 Mark schedule event e-step1-manual fired
38c7a38 Add Step 1 manual dispatcher proof
3056360 Initialize concrete routine schedule
```

The resulting event has `"fired_at": "2026-08-25T02:46:38Z"`. Zulip message
1998 appeared at the same second in `#front` / `front-routine-rtnotes`:

```text
Routine `rtnotes`, run of 2026-08-25T02:46Z. The standing request is the
latest post in #front › `routine-rtnotes`; this topic holds the earlier runs
and my comments on them. Do it.
```

Front acknowledged it in message 1999 one second later. The old rtnotes plist
remains loaded for now: Step 2 must first express its continuing cadence as
concrete events before that plist can be retired safely.
