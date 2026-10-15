# Set A — production-building rally points

Order: fresh first, then run. Both arms started from `7669fdb`. Evidence:

- `pj-agdev/.local/agent-session-p1-ex3/step3/A-fresh/` and `A-run/` (logs,
  transcripts, exec store, run record, branch bundle);
- `step4/A-fresh/` and `step4/A-run/` (probe, browser steps, screenshots,
  decisions).

The branches are `trial/agent-session-p1-ex3/a-fresh` (`a30735e`) and
`a-run` (`e029ec0`).

| | fresh | run |
| --- | --- | --- |
| Reported cost | **$6.57** | **$6.00** (0.91×) |
| Attempts / steps | 5 / 218 | 5 / 153 (one session `e8b777b9`) |
| Attempt wall time | 1,890 s | 1,249 s |
| Plan / implement / follow-up / repair / close-out | 0.51 / 1.65 / 2.30 / 1.71 / 0.39 | 0.58 / 1.19 / 1.69 / 1.85 / 0.69 |
| Context at attempt starts | 44 k each | 44 → 95 → 126 → 165 → 201 k |
| History reads after the plan | 8 (45 k chars) | 0 |
| Outcome | accepted | accepted |

**Designs.** The two designs are nearly identical:

- `cmdSetRally` / `cmdClearRally` in `sim.js`, refusing other owners' and
  non-production buildings;
- the rally applied at spawn;
- right-click with only buildings selected sets the rally;
- `Z` and a HUD button clear it;
- a dashed line and a flag mark it.

The differences:

- fresh sets the rally of the first selected building and moves the new
  unit with `cmdMove`;
- run sets every selected production building and gives the new unit a
  direct move order.

**Follow-up.** Both arms added the clear action in one attempt. A fresh's
attempt was the most expensive single attempt of the trial: 79 steps,
$2.30.

**Repair.** At the first evaluation both arms had the same defect. The new
help line explaining how to set and clear a rally ran past the canvas edge.
Both received the same repair text, and both fixed it by wrapping the help.

**Notes.** A fresh's implementation attempt launched a background subagent
to verify the feature visually; it had no browser. That attempt cost $1.65,
of which the subagent used 0.34 M cache read.

**Reading.** The retained arm was 9 % cheaper and 34 % shorter, on the same
design and the same repair. Fresh's extra cost is reconstruction:

- history and help reads;
- twice the source reads;
- 65 more steps.

The retained arm paid more per step from its larger prefix: 22.1 M cache
read against 18.9 M.
