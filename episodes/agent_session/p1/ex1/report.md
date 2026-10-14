# Agent session p1 ex1 — final report

Plan: [plan.md](plan.md). Step reports: [report1](report1.md) (launch
ownership), [report2](report2.md) (session mismatch), [report3](report3.md)
(verification and p1 report correction).

## Fixes

Both review findings from p1 are closed in the code, not by narrowing what
the p1 report claims.

1. **A launched process stays owned until it is stopped.**
   - The executor spawns the agent behind a launch gate. The gate starts the
     harness only after `wfe exec started` has registered its pid.
   - If the executor dies before registration, the gate exits without
     doing any agent work. A `launching` row therefore really does mean "no
     writer".
   - If a launch fails after the spawn, the process group is terminated and
     confirmed gone before the attempt is ended.
   - If the group cannot be confirmed gone, the executor keeps the process
     and the slot (`wfe exec reconcile --holding`) and reports that it is
     doing so.
2. **A mismatched session is stopped and refused.**
   - If the harness reports a different session id, both ids are recorded.
     The attempt is stopped, the group is confirmed gone, and the attempt
     ends `interrupted` with the reason
     (`wfe exec finished --interrupted`).
   - Under `run`, resume, recover and claim all refuse that session until
     it is retired (`session-mismatch`). The refusal names the command.
   - Retiring lifts no hold, permits no second writer, and resets no
     recovery budget. Observer's recovery request names the session as
     UNVERIFIED.
   - A failed attachment record stays pending and is retried. The end
     report also carries it.
3. **The p1 report is corrected.**
   - The step 5 trial total is **$14.06**, not $11.06. It matches the
     retained measurements and the driver's running total.
   - The overall trial cost in report6 is now about $17.6.
   - Compaction and per-node session splitting are described as untested
     candidates, not as established savings.
   - The `fresh` default and the small-sample qualification are unchanged.

## Tests and results

All tests use stubs and injected failures; there were no model calls.
Each command was run once after the last change:

- agautolab, step 1 and 2 boundaries: `pytest -k "mismatch or failed_attachment or launch_failing or unconfirmed or dying_before"`.
  **7 passed** (33.8 s).
- agautolab, existing p1 session cases that cross the rewritten
  `attach()`: **6 passed** (38.3 s).
- WFE `node --test test/sessions.test.ts`: **7/7**. `tsc --noEmit`: clean.
- agobserver `pytest -k recorded_session`: **1 passed**.
- Release gate (`wfe-release.sh deploy`, run once): `npm run check`
  **308/308** (306 before, plus 2 new), type check and build passed.

The 88-second executor suite and the full project suites were not rerun.
The mapping of each required regression to its test is in report3.

## Deployed versions (2026-10-14, 18:14 JST)

- agdevworld `75cdae7`, the WFE release; previous `dab1e54`. The service
  and `bin/wfe` both report it, and the editor answers HTTP 200.
- agautolab `043c082`. The executor was restarted and is idle; its first
  reconcile was clean.
- pj-agdev `a9e71e2` (agobserver). Observer was restarted, and the watchdog
  reports `ok`.
- pyagag is unchanged (`4dde895`). There was no store migration. The stores
  were backed up first.

Before activation, Nautobot (`nctl status`: ok) and the WFE state were
checked: no open attempt, nothing queued. After activation, the person's
hold on jevlab `experiment/run-001` is intact: `stopped`, seq 11. No live
workflow trial was run.

## Remaining limits

- **Mismatch is detected by polling.** The executor checks the harness's
  init event on each tick (every 2 s). A mismatched conversation can
  therefore run for up to about one tick before it is stopped. Its effects
  are what the retirement step asks a person or a recovering agent to look
  at.
- **No dedicated alert for an unconfirmed stop.** It is logged, and the
  slot stays held (shown as executing). Observer has no specific signal for
  it.
- **Claim's own refusal is tested directly.** A job for a mismatched run
  cannot reach claim through the normal paths: the run is held, and resume
  and recover refuse. The refusal was therefore tested on `sessionFor`
  itself.
- **Gate in the process tree.** The gate is a short Python start
  (`python -I -c`) before the harness `exec`s in its place. An agent's argv
  starts with that interpreter until then.
