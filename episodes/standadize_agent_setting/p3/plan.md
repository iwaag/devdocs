# Phase 3 plan — agautolab role and runner conversion

Convert all five agautolab roles (`front`, `director`, `mediator`, `coding`,
`summarizer`) to `ag.agent-config.v1` profile selection, delete the direct
Ollama window path, and build one process-runner seam for both harnesses.
This is a destructive phase: no backward compatibility with
`AUTOLAB_WINDOW_BACKEND`, direct Ollama code, or old adapter configuration.

Read first: `devpolicy/contracts/agent/spec.md` (the contract),
`../p1/legacy-names.md` (§agautolab — the deletion list),
`../p2/report.md` (agforge already solved most of the hard parts),
`agforge/service/agent_config.py` and `agent_run.py` (working reference code).

Hard constraints (the roadmap's minimum set — everything else is your call):

- no credentials or generated private payloads committed;
- no silent harness/model fallback — fail with the contract's error codes;
- keep raw transcripts/stdout that diagnose a failed run;
- do not add `--dangerously-skip-permissions`, `opencode run --auto`, or an
  equivalent unrestricted mode. (The existing `skip_permissions` adapter
  option predates this phase; leave it as-is, default off.)

Environment facts you will need:

- Everything runs locally on this Mac (agstudio). The gateway is started by
  hand: `cd agautolab && nohup python3 agent/gateway.py > .local/agent/gateway/serve.log 2>&1 &`,
  port 8791. Node deployment (agautolab1) is Phase 6 — ignore Ansible here.
- Ollama serves `http://127.0.0.1:11434`; the proven local model is
  `ollama/qwen3.6:35b-a3b-coding-nvfp4` (P2 ran it through OpenCode in ~73 s).
- The Claude binary lives in a version-numbered VS Code extension dir; the
  overlay's `command_glob` (newest match wins) is the P2-proven answer. The
  current pointer value is in `agautolab/.local/agent/claude_bin` — migrate
  it into the overlay, then delete the pointer file.
- The gateway runs on bare `python3`, not the uv venv. `tomllib` needs
  Python ≥ 3.11 — check `python3 --version` before choosing how the gateway
  imports the loader (options: `sys.path.insert` to `src/`, run the gateway
  under uv, or vendor the loader; your choice, just make the failure loud).

## Step 1 — config files and loader

Add committed `agautolab/agents.toml` and git-ignored
`.local/agents.local.toml`. Suggested shape (contract §2–§7):

- models: `ollama/qwen3.6:35b-a3b-coding-nvfp4`, `anthropic/claude-sonnet-5`;
- profiles: `local-coder` (opencode+ollama), `sonnet-coder`
  (claude_code+anthropic), `stub` (fake) — keep the same names as agforge;
  the Phase 7 matrix checks that a name means the same pair everywhere;
- roles: `front`, `director`, `mediator`, `coding`, `summarizer`. `front`
  should declare `requires = ["nested_harness"]` (it starts missions and
  launches directors) with `provides` covering it in the committed file;
- overlay: harness `command` / `command_glob`, `[local.provider.ollama].base_url`,
  and any per-node role→profile override.

Port agforge's `service/agent_config.py` (217 lines, stdlib-only) into
`src/agautolab/agent_config.py`. Copy-and-adapt, don't share a library —
deliberate duplication between workspaces is the established pattern (see
the comment above `WINDOW_DEFAULT_MODELS` in `gateway.py`). Keep the error
codes and `ResolvedAgent` shape; they are already conformance-tested.

Tests: loader conformance against `devpolicy/contracts/agent/examples/`
(valid + invalid, comparing error codes, not messages), plus profile
resolution for every one of the five roles including an overlay override.
agforge's test suite shows the pattern.

## Step 2 — the common process-runner seam

One module (e.g. `src/agautolab/harness.py`) that every role launches
through. It owns:

- command resolution — unify the three duplicate resolvers
  (`gateway.claude_bin()`, `session.sh resolve_bin()`,
  `binpath.resolve_command()`) into one, fed by the loader's resolved
  command/glob;
- argv construction per harness. P2-proven mapping: OpenCode gets the full
  canonical ID (`opencode run --format json -m ollama/qwen3.6:...`), Claude
  Code gets the stripped native name (`claude -p --output-format json
  --model claude-sonnet-5`). Role-specific args (allowedTools, --add-dir,
  extra opencode config) are passed in by the caller — authority stays with
  the role, the seam only launches;
- execution with timeout, `NO_COLOR=1`, provider endpoint injection via env
  (P2 precedent: inject the overlay's base_url and have the project
  `opencode.json` reference it — avoids touching global OpenCode config);
- output extraction: OpenCode JSONL — copy `extract_event_text()` from
  `agforge/service/agent_run.py:173` (text events + step_finish cost/token
  aggregation, degrades to identity on plain text); Claude JSON — the
  pattern already in `run_claude()` / `adapters/claude_code.py` (parse
  stdout, `result` is the text, carry `total_cost_usd`/`num_turns`/
  `duration_ms`/`is_error`; non-JSON stdout is a legitimate answer);
- normalized run metadata per contract §9: `role`, `profile`, `harness`,
  `provider`, `model`, `outcome`, `duration_ms`, `cost_usd`, `usage`,
  `num_turns`, `failure`. Don't invent a cross-harness usage conversion —
  P2 kept each harness's shape as reported.

Tests: deterministic stubs speaking each protocol — a fake "opencode" that
emits a JSONL event stream and a fake "claude" that emits the JSON envelope
— asserting extraction, error paths (launch failure, timeout, empty output,
`is_error`), and the normalized fields. No live model calls in tests.

## Step 3 — coding role (adapters + job.yaml)

- Add an `opencode` adapter to the registry in `src/agautolab/adapters/`,
  built on the Step 2 seam; keep `claude_code` and `fake` registered under
  those exact names.
- Add a `profile:` key to `job.yaml` (`Job.load` in `src/agautolab/job.py`)
  — the roadmap-sanctioned per-run override, which must name a profile
  declared in `agents.toml` (`E_UNKNOWN_PROFILE` otherwise). Default: the
  `coding` role's profile. Then delete model smuggling via
  `adapter_config.args: ["--model", ...]` — model choice comes only from
  the profile. `adapter_config` stays for genuinely adapter-local knobs
  (`add_dirs`, tool grants).
- Rename the evidence artifact `claude_output.json` to a harness-neutral
  name (e.g. `agent_output.json`). Update writers and both readers:
  `iter_cost()` in `gateway.py:327` and any docs. Old evidence dirs keep
  the old filename; readers may simply stop finding it there — those runs
  are archives, not contracts.
- Merge the normalized §9 fields into `adapter_result.json` (run_once
  already merges adapter meta; add the identity fields).
- Note: `.local/jobs/*/job.yaml` are local experiment artifacts. Update the
  ones you want to reuse for evidence; deleting stale ones is fine.

## Step 4 — gateway roles (front window + summarizer)

- Delete `run_ollama()`, `WINDOW_BACKENDS`, `WINDOW_DEFAULT_MODELS`,
  `window_backend()`, `window_model()`, and the env vars
  `AUTOLAB_WINDOW_BACKEND`, `AUTOLAB_WINDOW_MODEL`, `AUTOLAB_OLLAMA_URL`
  (including the live `.local/.env` line). The window resolves the `front`
  role through the loader and launches through the seam.
- The window's answer path must work with both harnesses. Its current
  Claude-only privileges (`WINDOW_ALLOWED_TOOLS`, PATH surgery so a nested
  `claude` works, 300 s budget for a nested director run) are `front`-role
  authority — express the OpenCode equivalent with an agautolab-owned
  opencode config/permission set, not by widening the seam. Expect real
  permission-vocabulary differences here; they are phase-report findings,
  not things to normalize away.
- Summarizer: resolve the `summarizer` role instead of
  `AUTOLAB_SUMMARY_MODEL`; keep its read-only tool grant and the
  one-paid-call-per-iteration cache. It currently shells out through a bash
  wrapper — either route it through the seam or keep the wrapper and feed
  it the resolved command+model; your call, but the run/cost record gains
  the §9 identity fields either way.
- Window records (`window/run-NNNN.json`): replace `backend` /
  `backend_model` with the normalized fields. Old records on disk keep the
  old spelling; readers of archives must tolerate both (contract §9).
- Delete `AUTOLAB_CLAUDE_BIN` and the `claude_bin` pointer once the overlay
  carries the command glob.

## Step 5 — mediator (session.sh)

Make the mediator profile-selectable. `session.sh` is bash and the loader is
Python; two workable shapes — pick one:

- a small resolver CLI (e.g. `python3 -m agautolab.resolve mediator`)
  printing the resolved argv/env for the shell to exec, or
- rewrite the session launch in Python and shrink `session.sh` to a thin
  wrapper (drive.sh only cares that it runs one session and exits).

Keep mediator-owned authority where it is: the wide-but-explicit allowlist
in `session.sh`, CHARTER.md on stdin, one session per file in
`.local/agent/sessions/`. Session records gain the §9 fields (a sidecar
JSON next to the raw output is fine — `/status` cost rollups read
`total_cost_usd` from session files, so either keep that key reachable or
update `session_summaries()`/`sessions_cost()` to the new shape).

An OpenCode mediator run will likely behave differently under the charter
(permission model, tool vocabulary). Record what you observe; smoke-level
evidence is enough — the mission evidence in Step 6 may run the mediator on
either harness.

## Step 6 — live evidence, cleanup, report

- Grep the repo for every §agautolab entry in `../p1/legacy-names.md`
  (env vars, `run_ollama`, `backend_model` writers, `claude_bin`,
  `claude_output.json`, stale doc paths like `.local/direction/`) — all
  gone except where a test proves absence or an archive reader tolerates
  old spellings. Update `agent/README.md`, `agent/GUIDE.md`, `AGENT_GUIDE.md`.
- `uv run pytest -q` green (existing suites `test_gateway_window.py`,
  `test_claude_code_adapter.py`, `test_run_once.py` etc. will need
  rewriting to the new vocabulary — rewrite, don't preserve).
- Live evidence (record IDs/paths in the report):
  - one mission proving front → mediator → coding with the §9 identity
    recorded at each boundary (window record, session record,
    adapter_result.json). A small job like the existing `smoke-e2e` /
    `fizzbuzz` shape is enough; `stub` profile is fine for rehearsal but
    the recorded mission should use real harnesses;
  - director smoke: ask the window to consult a project director and show
    the nested run happened;
  - summarizer smoke: summarize one iteration, cost recorded;
  - both harnesses exercised somewhere across the roles (e.g. window on
    `local-coder`, coding on `sonnet-coder` — any split is fine).
- Write `p3/report.md` in the shape of `../p2/report.md`: outcome,
  implementation, verification table, contract findings, and the
  permission/prompt differences found by trying both harnesses — reported
  as findings, not converted into new rules.
