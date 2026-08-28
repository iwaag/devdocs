# Step 2 — `localtest` standing routine and production fire

The standing request is now the latest post in `#front` › `routine-localtest`
(message 2525). It tells Front to create one ordinary Autolab mission for a
study project: resume `waiting_external` / `adoption_pending` work first,
otherwise select an eligible paper with a manual, create its repository-backed
local test, record evidence and cleanup, and turn an upper-actor dependency
into a persisted handoff rather than an indefinitely running task. It leaves
the installer, runtime, and exact experiment shape to the paper and live
environment.

Front processed the schedule request in `#front` › `front-schedule`:

| item | value |
|---|---|
| request | `r10` |
| event | `e40` |
| routine | `localtest` |
| scheduled time | 2026-08-28T06:15:00Z |
| request expiry | 2026-08-28T07:30:00Z |

The production dispatcher, rather than a manual `--now` invocation, marked
and dispatched `e40` at `2026-08-28T06:16:41Z`. It posted message 2529 into
`#front` › `front-routine-localtest`; Front acknowledged it as message 2530.

No schema or dispatcher change was needed. The first-run routing surface is
therefore `#front` › `front-routine-localtest`; the next step records the
Front-to-Autolab mission and experiment outcome.
