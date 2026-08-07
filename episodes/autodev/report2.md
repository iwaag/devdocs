# autodev — Step 2 report: Claude Code headless adapter, proven with a toy job

Status: **complete** (adapter implemented, stub-tested, and proven with a real
model on a local toy job that converged; token/cost fields logged in evidence).

## What was built

`claude_code` adapter in `agautolab/src/agautolab/adapters/claude_code.py`,
registered next to `fake`:

- Runs `claude -p --output-format json` one-shot with `cwd=target/`, prompt
  fed on stdin. Chose the CLI over the Agent SDK: the CLI is one subprocess
  behind the existing tiny adapter interface, needs no extra dependency, and
  tool restriction is available via `--allowedTools`.
- Captures the full stdout JSON as `claude_output.json` in the iteration's
  evidence, and copies token/cost fields (`total_cost_usd`, `usage`,
  `modelUsage`, `num_turns`, `duration_ms`, `is_error`, `session_id`,
  `permission_denials`, …) into `adapter_result.json`.
- `adapter_config`: `command` (binary path, default `claude`), `args` (extra
  CLI args, e.g. `--model`, `--allowedTools`), `skip_permissions` (adds
  `--dangerously-skip-permissions`, default off).
- Error handling: subprocess timeout → exit −1 with partial output kept as
  evidence; unlaunchable binary → error result; non-JSON stdout → error with
  raw output; `is_error: true` in otherwise-exit-0 JSON → exit 1.

Small interface extension to support this cleanly: `AdapterResult` gained
optional `meta` (merged into `adapter_result.json`) and `artifacts`
(filename → content, written into `evidence/iter-NNNN/`). `run_once`'s outer
wall-clock guard now fires `timeout + 30s` so a well-behaved adapter's own
subprocess timeout wins.

## Policy note (constraint 2)

This Mac holds real credentials, so the local proof did **not** use
`--dangerously-skip-permissions`. The toy job instead restricted tools via
`--allowedTools "Write,Edit,Read,Glob,Grep,Bash(uv run:*),Bash(python3:*),
Bash(pytest:*),Bash(ls:*)"`. `skip_permissions` stays a config option for the
experimental dev node/VM in later steps. `permission_denials` in the evidence
was empty, so the allowlist was sufficient for this job.

## Verification

- Stub-CLI tests (no tokens): 8 new tests in `tests/test_claude_code_adapter.py`
  cover JSON/meta parsing, `is_error` mapping, non-JSON output, timeout,
  missing binary, `--dangerously-skip-permissions` passthrough on/off, bad
  config, and a full `run-once` integration where a stub `claude` writes a
  file, the gate passes, and cost/usage land in evidence. Full suite:
  `uv run pytest -q` → **18 passed**.
- Real-model toy job (the Step 2 acceptance): job at
  `agautolab/.local/jobs/fizzbuzz` (ignored dir), goal "implement fizzbuzz +
  pytest tests in this empty repo", gate `uv run --with pytest -- pytest -q`,
  model `claude-sonnet-5` via the VSCode-extension-bundled CLI binary
  (2.1.223; no standalone `claude` is installed on this Mac — noted below).
  `autolab run-once` → **converged on iteration 1**, exit 0. Evidence:
  `total_cost_usd 0.1319`, `num_turns 5`, `duration_ms 13278`, output tokens
  813 (cache read ~185k), `permission_denials: []`. Re-running the gate by
  hand: 4 tests pass. `target/` git log shows the initial-state commit plus
  one iteration commit. A rerun of `run-once` on the converged job exits 0
  immediately (terminal short-circuit).

## Notes / follow-ups

- No standalone `claude` CLI is on PATH on this Mac; the job config points at
  the VSCode extension's bundled native binary. Absolute path lives only in
  the ignored `.local/` job.yaml (devpolicy-compliant). Dev-node setup
  (Step 5) should install the real CLI.
- Wart observed: the gate run generates `__pycache__/` before the iteration
  commit, so it gets committed into `target/`. Harmless here; consider
  seeding a default `.gitignore` into auto-initialized targets, or committing
  before running gates. Left as a small follow-up.
- One-iteration convergence means the multi-iteration NOTES/gate-failure
  feedback path was exercised with a real model only implicitly (it is
  covered by the fake-adapter tests). Step 6's Othello job will exercise it
  for real.

Also reported in `pj-agdev/devdocs/episodes/agautolab/begin/report.md`.
No pj-clusterintent work was needed.
