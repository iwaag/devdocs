# Set B — Shift-queued movement waypoints

Order: run first, then fresh. Both arms started from `7669fdb`. Evidence:

- `pj-agdev/.local/agent-session-p1-ex3/step3/B-run/` and `B-fresh/`;
- `step4/B-run/` and `step4/B-fresh/`.

The branches are `trial/agent-session-p1-ex3/b-run` (`f2963e9`) and
`b-fresh` (`64b4a95`).

| | fresh | run |
| --- | --- | --- |
| Reported cost | **$3.71** | **$3.02** (0.81×) |
| Attempts / steps | 4 / 130 | 4 / 99 (one session `afafd9be`) |
| Attempt wall time | 1,015 s | 811 s |
| Plan / implement / follow-up / close-out | 0.60 / 1.68 / 1.01 / 0.43 | 0.54 / 1.10 / 0.98 / 0.40 |
| Context at attempt starts | 44 k each | 44 → 86 → 115 → 136 k |
| History reads after the plan | 3 (22 k chars) | 0 |
| Outcome | accepted, no repair | accepted, no repair |

**Designs.** The two designs are close:

- `cmdQueueMove` and `e.waypoints` in `sim.js`, using the same names
  independently;
- Shift-right-click appends a destination;
- a dashed route with a numbered point at each destination.

The differences:

- **How queues are cleared.** run clears the queue explicitly in every
  non-queued command. fresh clears it inside `setOrder`, so any order
  replaces it, and it advances waypoints without calling `setOrder`.
- **Help and counter.** fresh also added an in-game help line and, for the
  follow-up, a "残り N" counter above the unit. That help line is cut off
  at the canvas edge, but help was not part of B's criteria and the other
  arm added none, so it was not counted.

**Quality.** Both arms passed every B criterion and needed no repair:

- ordered visits;
- a plain move replaces the queue, and Stop clears it;
- a dead unit is safe;
- gather and attack still work;
- the follow-up numbering.

**Reading.** With equal work and no repair, retained was 19 % cheaper.
Its implementation attempt cost $1.10 against $1.68 and took 40 steps
against 51. That attempt resumed from 86 k with the plan already in
context, while fresh re-read the history, plan and source.
