# autodev — Step 3 report: loop mode

Status: **complete** (`autolab loop` implemented, tested, and smoke-run as a
plain foreground command; systemd unit template recorded for Step 5).

## What was built

`autolab loop <job-dir>` in `agautolab/src/agautolab/loop.py`, wired into the
CLI next to `run-once`:

- While `run_once` returns 10 (continue), sleep `--sleep SECONDS` (default 5)
  and run again. Any other code ends the loop and becomes the loop's own exit
  code, so the terminal verdict (0=converged, 20=stuck, 30=error) is visible
  to systemd and shell callers alike.
- Ctrl-C returns 130; the interrupted iteration's state is already persisted,
  so nothing is lost. Crash recovery needs no extra code per the plan: all
  state is on disk, the next `loop`/`run-once` reconstructs and continues.
- Inherited edge case, documented in the module: `run_once` exits 0 silently
  when another process holds the job lock, so a second concurrent `loop` on
  the same job ends immediately with 0, leaving the job to the lock holder.

Systemd template at `agautolab/devenv/systemd/autolab@.service` (one user-unit
instance per job, `Restart=on-failure`, `RestartSec=30`). Deliberate detail:
`RestartPreventExitStatus=20 30` — stuck/error are terminal verdicts, not
crashes; without it, `on-failure` would restart into a job that only
short-circuits with the same code until the start-limit trips. Installation
and path adaptation are Step 5 work; the template documents its assumptions.

## Verification

- 5 new tests in `tests/test_loop.py` (fake adapter, `time.sleep` patched):
  loop-until-converged with sleep only *between* iterations, stop on stuck,
  immediate exit on an already-terminal job without sleeping, 130 on
  KeyboardInterrupt with state persisted, and the CLI `loop` subcommand
  end-to-end. Full suite: `uv run pytest -q` → **23 passed**.
- Foreground smoke run (plan requirement "must also work as a plain
  foreground command"): a fake-adapter job with a 3-lines gate under
  `uv run autolab loop --sleep 1` converged in 3 iterations, exit 0,
  `state.json` terminal — no hand-driven re-invocation needed.

## Notes / follow-ups

- systemd unit is a *template*; `ExecStart` assumes `uv` at `~/.local/bin/uv`
  and jobs under `~/agautolab/jobs/`. Step 5 fixes real paths on the dev node.
- No sleep/backoff distinction between "gate still failing" and "adapter
  errored but run_once continued" — not needed yet; revisit if Step 6 shows
  token burn on rapid failing iterations (ceilings from job.yaml already cap
  the total).

Also reported in `pj-agdev/devdocs/episodes/agautolab/begin/report.md`.
No pj-clusterintent work was needed.
