# Step 4 report — the monitor's progress, watched from outside it

Plan: [plan.md](plan.md) step 4. 2026-09-24 (JST), by the Omni Agent. Code, fixtures and launchd
templates; installed and exercised live in step 5.

## What was wrong with "is Observer alive"

- nctl's liveness reads `<workspace>/.local/agag-status.json`. Observer's workspace is the
  superproject; its listener wrote the file in `agobserver/.local/`. Hence `unobserved`, in p1 and
  today (report1). The Observer template now sets `AGAG_STATUS_PATH` to the workspace's file.
- Even found, that file says only that the listener's intake polled. The monitor is a thread
  beside it; it could stop, or sit inside a local-model judgment (29–128 s measured, run inside
  the look loop in p1), and every liveness signal in the realm stayed green.

## The record

Every cycle writes `agobserver/.local/monitor-health.json` (`agobserver.monitor-health.v1`) twice
— at the start (`in_progress`) and at the end — so a cycle that never ends is visible as one:

| Field | Says |
|---|---|
| `cycle` | count, started, completed, duration, in progress |
| `source` | the mirror's state, why, stale since, last event |
| `requests` | tracked, looked at last cycle, **oldest unchecked** (seconds since the least recently looked-at tracked request was looked at), open incidents |
| `judgment` | the one running (incident, since), how many wait, the last one (seconds, verdict), the timeout |
| `latest_failure`, `spent` | the last exception a cycle raised; posts and judgments made |
| `enabled` | `false` is written when the monitor is switched off, so *off* is a state, not an absence |

## Slow judgment cannot hold the other requests

Judgments now run on a worker thread of their own (`observer-judge`). The look queues a judged
candidate with a snapshot of what it is to read and moves on; the verdict is picked up by a
later look. A held judgment holds itself: in the fixture a judgment blocked indefinitely while the
same look opened the other request's incident, and the next look completed on schedule. The
judgment itself stays bounded by the triage run's own 150 s timeout.

## The watchdog is another process

The agdevworld relay (always on, launchd, already the operator's board) evaluates the record
every 30 s, deterministically, from the record's own timestamps (same host) and Observer's
listener status file:

| State | Condition (interval *i* = 120 s) | Alert |
|---|---|---|
| `ok` / `idle` | cycles completing; `idle` when nothing is tracked | — |
| `disabled` | the monitor is switched off | — |
| `missing` | no record | yes |
| `stopped` | no cycle in progress, none completed for 3*i* + 60 s = 7 min; says whether Observer's listener is still polling ("the process is alive and the monitor alone is not moving") or not ("the process is down") | yes |
| `stalled` | one cycle in progress for 7 min | yes |
| `judgment_stalled` | a judgment running for 2 × its timeout = 5 min | yes |
| `unable_to_observe` | the monitor's mirror stale for 5 min | yes |
| `degraded` | a tracked request not looked at for 7 min | yes |

It shows on `/ops` (`watchers.observer_monitor`), on the ops board's header line — ahead of the
rows, because quiet rows mean nothing when what flags them has stopped — and on `/healthz`. On a
change into a failing state, and on the way back, it sends **one direct message to the realm's
owners** from the relay's Opsroom bot (`AGENTROOM_MONITOR_ALERT=dm`). That extends the bot's
posting contract (until now `#ops-testbed` only): `#ops-testbed` has no human owner subscribed, so
a post there would reach nobody. The DM goes to humans only and names no agent, so it buys no
run.

Operator fault hooks for step 5 (one file each in `agobserver/.local/faults/`, created only by a
person): `monitor-stop` ends the monitor thread with the process and listener up; `triage-stall`
holds the next judgment until the file is removed; `mirror-stale` makes the monitor treat its
source as stale while the file exists (a stand-in for a real stale feed; labeled as injected).

## Recovery of monitoring

A restarted monitor resumes from its incident store and the tracked-request index: requests
already made are counted, retry intervals are measured from the stored request times, and a lost
store is adopted from the incident notes (steps 2 and 3). Nothing is replayed blindly.

## Fixtures

| Suite | Result |
|---|---|
| agobserver | 104 passed; `test_monitor_health.py`: every cycle leaves a record; a stuck judgment holds only itself (the other request's incident opened in the same look, the next look ran, the verdict picked up after release); the stop fault ends the thread and the record stops moving; *off* says so |
| agentroom | 330 passed; `test_watchdog.py`: ok/idle, missing/disabled, stopped with the process alive vs down, stalled, judgment stalled, stale source vs a brief one, degraded, and exactly one alert in and one out |
| agdevworld frontend | `tsc --noEmit` clean |

## Targets carried into step 5

Measured: a look takes 0.18 s over the live realm without a judgment; judgments 29–128 s. From
these, and the watchdog's 30 s poll: a stopped or stalled monitor becomes visible within
**7.5 min**; a held judgment within **5.5 min**; a stale source within **5.5 min**. p1's six-minute
missing-start target is unchanged — the monitor's own cycle is unaffected by judgments now, so a
`unstarted` candidate (180 s grace) is found by the first look after it, ≤ 5 min.
