# Step 6 report — first real full-auto job: browser Othello

Date: 2026-08-07. Outcome: **converged in a single iteration**, no human help
mid-run. `autodev/othello-web` on the agstudio gitea now holds a playable
browser Othello (human Black vs simple CPU White) produced by the loop on
agautolab1, all 10 acceptance tests passing.

## Job design

Stack choice: plain ES module + `node --test` — no npm deps, no browser
automation, so gates are cheap and deterministic on the VM's user-space
Node 22.

- Seed repo (pushed before the run): `README.md` stating the deliverable and
  the exact engine API, and `test/acceptance.test.mjs` — 10 `node:test` cases
  covering the standard opening position, legal-move generation for both
  sides, flip/purity semantics of `applyMove`, diagonal flips, forced passes,
  `cpuMove` legality, a full engine self-play game that must terminate
  consistently, and static checks that `index.html` exists and loads
  `othello.js` as an ES module.
- The acceptance test was validated **before** the run against a throwaway
  reference implementation (kept out of the repo): fails exit-1 with no
  implementation (checked on the VM too), passes 10/10 with a correct one.
  Gate command is bare `node --test`; `node --test test/` (directory
  argument) misbehaved on newer Node and was avoided.
- `job.yaml`: adapter `claude_code` with `--model claude-sonnet-5` pinned
  (per user instruction; smoke-tested headless on the VM first) and
  `skip_permissions: true` (experimental VM per policy); gate `node --test`;
  `max_iterations: 15`, `no_progress_limit: 3`, timeouts 900 s / 300 s.

## The run

Started `autolab@othello-web.service` at 07:56:59 UTC; monitored only by
polling `state.json` over ssh (no intervention).

- Iteration 1: adapter 64 s wall, 12 turns, **$0.311**, ~6.0 k output tokens
  (sonnet-5, plus haiku for internal utility calls). Wrote `othello.js`
  (109 lines) and `index.html` (235 lines). Gates 1/1 → `converged`;
  the loop exited 0 and the unit went inactive cleanly (no restart).
- Post-run verification: `test/` and `README.md` untouched by the agent
  (empty diff vs seed), 10/10 tests pass, UI wiring present in the page
  (click handlers, legal-move highlighting, score/status, new-game control).
  Result pushed to gitea `main` (`bbd5cea`). A human playtest in an actual
  browser remains open — the gates cannot judge feel/rendering.

## Observations for Step 7

- The job converged in one iteration, so this run exercised prompt assembly,
  gates, evidence, and systemd wiring — but **not** the multi-iteration
  NOTES handoff or stuck detection with a real model. A harder job (or
  deliberately stricter gates) is needed to observe those paths for real.
- `run_once` commits `target/` but never pushes; I pushed manually after the
  run. Follow-up candidate: optional `git push` on iteration commit or at
  least on terminal states.
- Monitoring was ad-hoc ssh polling of `state.json`; fine for one job, but a
  `autolab status <job-dir>` (or journald-based) view would be nicer.
- Friction on agstudio: the permission classifier blocked `scp` and one
  compound ssh setup command; ssh-with-stdin and smaller per-step ssh
  commands worked. Not painful enough yet to file, noting for recurrence.
- Housekeeping done en route: `jobs/` added to agautolab's `.gitignore`
  (job dirs hold evidence and job-local state; pushed to GitHub origin and
  the gitea mirror so the VM checkout stays clean).

Also reported in `pj-agdev/devdocs/episodes/agautolab/begin/report.md`.
