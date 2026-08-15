# P1 — Retire opencode, run local models on agcode

Braindump: `braindump.md`. Prior research: `devdocs/research/agent_harness.md`.

Goal: `opencode` disappears from every project — code, config, Ansible,
clusterintent desired/actual state, and the running Macs/nodes. Local models
run on `agcode` (`agag.agcode`, inside pyagag). Destructive phase: no
backward compatibility, no deprecation window, no migration shims.

Environment is a private experiment. Keep irreversible-harm guards (don't
destroy cluster guests, don't leak secrets into tracked files); drop
wrongness-prevention guards. Implementer chooses tactics freely.

## What agcode is and is not

`pyagag/src/agag/agcode.py`, 661 lines, stdlib only, single file.

- Non-streaming `POST {base_url}/v1/messages` (Anthropic Messages API).
  ollama serves it; `x-api-key` is a dummy constant, so no account, no key.
- Exactly one base directory (`--working-dir`, default cwd). Every tool
  resolves paths itself (`(base / path).resolve()`). This is the whole point:
  0/12 wrong-base failures vs opencode's 6/10 on the same model.
- Tools: `read`, `write`, `list`, `run` (shell, unrestricted).
- One shot, stateless. No sessions, no MCP, no permission engine, no
  instructions file, no streaming, no cost reporting.
- `meta` already speaks `ag.agent-run.v1`; `_extract_agcode` in `harness.py`
  is one parse plus a key copy.

Three of those absences block a direct swap (agdevworld's MCP tools, cagent's
sessions + per-door permissions + instructions file). Step 1 closes them with
one small in-process seam. Everything else is deletion.

## Inventory (verified 2026-08-15)

Live on `agstudio`: `opencode` on `127.0.0.1:4096` (launchd
`com.clusterintent.opencode.agent`, node_agent profile), `:4097`
(`com.clusterintent.cagent-opencode`), `:4098`
(`com.clusterintent.cagent-window-opencode`).

Desired state: service `node-agent` + 3 `node_agent` placements (aghub, agpc,
agstudio) + 3 `llm_provider` bindings; service `cagent-opencode` + its
`cagent-opencode-agstudio` manual_toolchain placement.

Code owners of the dependency:

| Area | Coupling |
|---|---|
| pyagag | `CANONICAL_HARNESSES`, `build_argv`, `extract_event_text`, `INTRINSIC_CAPABILITIES`, `opencode_config` param, README, `docs/agent-config-v1.md` |
| agautolab | `profiles.local`, `role_run._opencode_config`, `agent/opencode-{front,mediator,coding,readonly}.json`, `opencode.json` |
| agdevworld | `profiles.local` (**the live `front`**), `opencode.json` MCP block, `assistant/Dockerfile` (`opencode-ai@1.18.10`), `overlay.py` |
| agforge | `profiles.local` (declared, unused — both roles are `sonnet`), `opencode.json` |
| cagent | `opencode_client.py`, `worker.py`, `server.py`, `main.py`, `opencode/`, `window/*.template`, both `start.sh`, 2 launchd plists |
| ansible_agdev | `roles/opencode_agent/`, `playbooks/agent/setup_opencode.yml`, `roles/autolab_node` (command + `agents.local.toml.j2`), `vars/deployment_profiles.yml` |
| nctl | `agent.py`, `agent_api.py`, `agent_render.py`, `cli/main.py` agent tree, `AgentConfig`, `nintent_opencode_ollama_url`, `PROFILE_BINDING_VARIABLES` |
| nintent | `PROFILE_BINDING_NAMES["node_agent"]` and its test suite |
| devpolicy | `contracts/agent/spec.md`, `agent_records.md`, contract examples |

## Step order

pyagag is an editable path dependency of every consumer, so removing
`opencode` from it breaks the tree instantly. Add first (Step 1), migrate
consumers (2–6), then delete (7–9).

---

### Step 1 — pyagag: give agcode a tool seam (additive)

`src/agag/agcode.py` stays single-file and stdlib-only.

- Make the tool table a parameter, not a module constant. `run()` gains
  `tools=` (default the current four) carrying both the JSON spec and the
  callable; `dispatch_tool` takes the mapping instead of reading `_TOOL_FUNCS`.
- Export a read-only preset (`read` + `list`) so a caller can hand out a
  smaller set without a permission engine. Permission *is* the tool set here
   — that is the Tool Giving shape, and it is simpler than opencode's bash
  glob lists.
- `run()` gains `system_suffix=` appended to `SYSTEM_PROMPT`. This is how
  per-role instructions (today's `AGENTS.md` files) arrive. Keep the working
  directory sentence first and unconditional.
- `run()` gains an optional `stop` callable checked between turns, so a
  caller can cancel a long run. Cancellation ends the run as `aborted`.
- The CLI keeps today's defaults exactly (four tools, no suffix). Add a
  prompt-audit test asserting the base system prompt still names exactly one
  directory and contains no operator-specific data.

Do **not** add MCP, streaming, or session resume. Out of scope, still.

Report: `report1.md`.

### Step 2 — agautolab → agcode

- `agents.toml`: `[profiles.local]` → `harness = "agcode"`. Model stays
  `ollama/qwen3.6:35b-a3b-coding-nvfp4` (agcode strips the `provider/` prefix
  itself via `native_model`).
- Delete `agent/opencode-*.json` (4 files) and `opencode.json`; delete
  `_opencode_config()` and the `opencode_config=` argument in `role_run.py`.
- `ROLE_ALLOWED_TOOLS` stays — it is claude_code-only and the `sonnet` roles
  still need it.
- `director`/`summarizer` lose their read-only enforcement. Either pass the
  read-only tool preset from Step 1 for those two roles, or accept the loss
  and say so in the report. Implementer's call.
- Tests: `tests/test_role_run.py`.

Live check: one `front` run through the gateway (`:8791`) on agstudio, and
one on agautolab1 after Step 6 redeploys it.

Report: `report2.md`.

### Step 3 — agdevworld → agcode with native tools

`front` is the live local-model role and its only tools are the MCP service
(`tool_service.py`: `fetch`, `wait`, `switch_view`, `show_image`); all
built-ins are disabled in `opencode.json`.

- Register those four as agcode tools in-process using the Step 1 seam. The
  logic is already stdlib and side-effect-simple (`fetch` is urllib,
  `switch_view`/`show_image` append to `AGDEVWORLD_ACTIONS_FILE`); lift the
  bodies, keep `tool_service.py` as the MCP entry point for the `sonnet`
  profile, which still goes through `--mcp-config`.
- This means `run_front` calls `agcode.run(...)` directly for `harness ==
  "agcode"` rather than `run_harness`, or `run_harness` grows a way to pass
  tools. Prefer whichever keeps `chat.py` legible; record the choice.
- `agents.toml`: `[profiles.local]` → `harness = "agcode"`.
- `_launch_conditions`: the agcode branch needs no project-root cwd (that was
  only opencode's relative MCP command).
- Delete `opencode.json`; strip `opencode` from `overlay.py` and from
  `assistant/Dockerfile` (drop the `opencode-ai@1.18.10` npm install — the
  image shrinks).
- Tests: `assistant/tests_py/test_chat.py`.

Live check: rebuild the assistant container, ask the front agent something
that forces a `switch_view` and a `fetch`, confirm the browser reacts.

Report: `report3.md`.

### Step 4 — agforge

Nothing runs on `local` here (both roles are `sonnet`). Delete
`[profiles.local]` and `opencode.json`, fix `tests/test_agent_config.py` and
`tests/test_service.py`, update `service/GUIDE.md` and `README_DEV.md`.

If a local-model profile is wanted, add `harness = "agcode"` — but agforge's
work is image/audio tooling behind shell commands, so `run` alone may be
enough. Optional.

Report: `report4.md`.

### Step 5 — cagent → agcode, in-process, under the agent contract

The largest step. cagent today drives a long-lived `opencode serve` over its
session API, on `openai/gpt-5.6-luna`, with two instances differing only in
permission set and instructions.

**Bring cagent into `ag.agent-config.v1`.** It has no `agents.toml` today.

```toml
schema = "ag.agent-config.v1"
project = "cagent"

[models."ollama/qwen3.6:35b-a3b-coding-nvfp4"]
[models."anthropic/claude-sonnet-5"]

[profiles.local]   harness = "agcode",      model = "ollama/qwen3.6:35b-a3b-coding-nvfp4"
[profiles.sonnet]  harness = "claude_code", model = "anthropic/claude-sonnet-5"

[roles.node]    profile = "local"
[roles.human]   profile = "local"
[roles.window]  profile = "local"
```

(Spell it as real TOML tables; the above is shorthand.) Per-door backend
choice then lives in `.local/agents.local.toml`, and every answer's run
record names what actually served it — Agent ≠ Model, satisfied by
construction.

- Replace `opencode_client.py` with a runner that calls `agag.agcode.run()`
  in-process from `Worker`. Delete `OpenCodeClient`, `OpenCodeError`,
  `AssistantMessage`, the message-count polling, and the `finish ==
  "tool-calls"` step logic — all of it existed to read opencode's session API.
- **Session continuity**: `Store` already holds every request's message and
  response per session (`list_session_requests`). Rebuild the conversation by
  prepending prior turns to the task string. agcode is stateless; cagent
  becomes the memory. Cap the replay length and say what the cap is.
- **Instructions**: `opencode/AGENTS.md` and `window/AGENTS.md` become the
  per-role `system_suffix`, read from disk per request. This kills a real
  wart: opencode fixed instructions at process start, so editing AGENTS.md
  required a restart and a stale file once hung a turn forever
  (`vision/file_output/report4.md`).
- **Window permissions**: the window is unauthenticated plain HTTP and
  opencode's allow-list was its only guard. Do not rebuild that allow-list —
  give the window role a dedicated `nctl` tool that execs
  `uv run --project nctl nctl <subcommand>` with the read-only subcommands
  (`status`, `drift`, `relations`, `actual`, `ops list`, `ops show`) selected
  in Python, plus `read`/`list` and the incident recorder. No shell tool at
  all. That is strictly tighter than the glob list it replaces and needs no
  permission engine.
- **Cancellation** uses the Step 1 `stop` callable. **Timeout** uses agcode's
  `deadline_s`; keep `CAGENT_TURN_TIMEOUT_SECONDS` as its source.
- **cost_usd** becomes `None` for agcode runs. agcode reports no cost and
  inventing one is worse than omitting it. Keep the field for `claude_code`
  runs.
- Delete: `opencode/` (AGENTS.md moves to a neutral path), `opencode/start.sh`,
  `window/start.sh`, both `*.json.template`, `CAGENT_OPENCODE_URL` /
  `CAGENT_WINDOW_OPENCODE_URL` / `CAGENT_OPENCODE_MODEL` env handling, the
  OpenAI key-file plumbing, `pyproject.toml` mentions.
- launchd: remove `com.clusterintent.cagent-opencode` and
  `com.clusterintent.cagent-window-opencode` (bootout + delete the plist
  templates in `devenv/launchd/` and their README rows). `cagent-api` is now
  the only process.
- Tests: `tests/fakes.py`, `test_server.py`, `test_worker.py`,
  `test_window_server.py`, `test_zulip_window.py`, `test_incident.py`.

Live check: one DM to the Cagent bot in Zulip, one `POST /window` asking for
`nctl drift`, one authenticated human request. Keep the wire transcripts.

Report: `report5.md`.

### Step 6 — Ansible

- Delete `roles/opencode_agent/` (all 11 files), `playbooks/agent/setup_opencode.yml`,
  `vars/opencode_agent.yml.example`, and the `vars/opencode_agent.yml` line in
  `.gitignore`.
- `playbooks/agent/setup_autolab_node.yml`: drop the `opencode_agent` role
  (it was there with `opencode_agent_serve: false` purely to install the
  binary).
- `roles/autolab_node`: delete `autolab_node_opencode_command` and the
  `[local.harness.opencode]` block in `templates/agents.local.toml.j2`. Add
  **no** `[local.harness.agcode]` block — agcode's default command is
  `sys.executable`, which is already the interpreter importing `agag`.
- Deleting the role does not uninstall anything already on a node. Remove
  `~/.local/bin/opencode` and `~/.config/opencode` / `~/.local/share/opencode`
  on aghub, agpc, agstudio, agautolab1 — either a one-shot playbook or by
  hand, then verify with `which opencode`.

Redeploy per README_DEV: push agautolab to **GitHub** (never the agstudio
Gitea mirror — it goes stale silently), re-render the production inventory,
then `--limit agautolab1` and `--limit agstudio`.

Report: `report6.md`.

### Step 7 — nctl and clusterintent state

- Delete the whole `nctl agent` subsystem: `agent.py`, `agent_api.py`,
  `agent_render.py`, the `agent_app` typer tree in `cli/main.py`,
  `AgentConfig` in `config.py`, the `[agent]` section in `example.nctl.toml`,
  and `tests/test_agent.py` / `test_agent_api.py`. It exists only to tunnel
  to `opencode serve` and to run `opencode attach`.
- `vars/deployment_profiles.yml`: remove the `node_agent` profile and its
  `deployment_profile_reconciliation` entry (the one whose action is
  `setup_opencode.yml`).
- `production/contract.py`: drop `nintent_opencode_ollama_url` from
  `_BASE_HOST_VARIABLES`. `production/service_dependencies.py`: drop the
  `("node_agent", "llm_provider")` entry from `PROFILE_BINDING_VARIABLES`.
  That leaves the map empty — keep the resolver and its tests alive with a
  test-local mapping, or leave one neutral declared entry. Either is fine;
  don't delete the binding machinery, it is the only service-dependency path
  in the system.
- `reconcile/classify.py:99` mentions `setup_opencode.yml` in a comment.
- Desired state (`.local/desired-state.yaml`): remove the `node-agent`
  service, its 3 placements, its 3 `llm_provider` bindings, and the
  `cagent-opencode` service + placement. Preview with
  `uv run --project nctl nctl desired apply -f .local/desired-state.yaml`,
  then `--yes`. Follow with `nctl prune` for the orphaned rows.
- `deployment_profile_digest` is computed from the file, not pinned — it just
  changes. Re-render: `uv run --project nctl nctl render production --out
  inventories/generated`.
- Confirm `nctl drift` no longer reports node-agent anywhere and `nctl status`
  stays ok.

Report: `report7.md`.

### Step 8 — pyagag: delete the opencode harness

Now nothing calls it.

- `agent_config.py`: drop `"opencode"` from `CANONICAL_HARNESSES` and
  `INTRINSIC_CAPABILITIES`, drop the `opencode` default command, drop the
  ollama-`base_url`-required check that is opencode-specific.
- `harness.py`: drop the `opencode` branch of `build_argv`, the
  `opencode_config` parameter, and `extract_event_text` (it parses opencode's
  event stream and has no other caller — check the tests before deleting).
- `docs/agent-config-v1.md` and `README.md`: remove opencode from the harness
  table, the command-resolution examples, and the capability paragraph.
- Tests: `tests/test_harness.py`, `tests/test_agent_config.py`.

Report: `report8.md`.

### Step 9 — devpolicy, and optional nintent

- `devpolicy/contracts/agent/spec.md`: harness table row, the
  `opencode -m` example, `[local.harness.opencode]`, the capability sentence.
- `devpolicy/agent_records.md`: the `opencode + ollama/<model>` example.
- `devpolicy/contracts/agent/examples/`: `invalid/*.toml` and `valid/*` all
  reference opencode; regenerate them against the new harness set. The
  `unknown-model` / `capability-unmet` / `overlay-out-of-scope` cases should
  keep testing the same failure, just spelled with `agcode`.
- `README_DEV.md` files (agdevworld, agforge, agautolab, pj-clusterintent)
  and `pj-clusterintent/devenv/launchd/README.md`.
- Historical `devdocs/episodes/**` and `devdocs/vision/**` are the record of
  what happened. **Leave them alone.**
- **nintent (optional, do last if at all)**: `PROFILE_BINDING_NAMES` and
  `REFUSED_PROFILE_CONFIG_KEYS` still name `node_agent`, and its test suite
  uses that profile heavily. Nothing breaks by leaving it — nintent only
  *rejects undeclared* binding names, it never requires the profile to exist.
  Changing it costs commit → ask the developer to push → `docker compose
  build` → restart (no mount, no hot reload), and would gut the binding
  tests. Recommended: leave it, note it as dead config in the report.

Report: `report9.md`.

---

## Done when

1. `grep -ril opencode` across all projects returns only `devdocs/**` history
   (and this plan).
2. Every test suite green: pyagag, agautolab, agdevworld, agforge, cagent,
   nctl, ansible_agdev syntax check.
3. No `opencode` process on any node; ports 4096/4097/4098 free; the two
   cagent launchd labels gone; `which opencode` empty on aghub, agpc,
   agstudio, agautolab1.
4. `nctl status` ok, `nctl drift` clean of node-agent, rendered inventory
   regenerated.
5. One live wire transcript per migrated entrance: agautolab `front`,
   agdevworld `front`, cagent window, cagent human. Verified at the wire, not
   by asking the agent — every decisive fact in the prior research came from
   captured payloads, and the model's self-reports were true but useless.

## Hints

- `agcode` needs no `[local.harness.agcode]` overlay: `_resolve_command`
  returns `sys.executable`, and `build_argv` runs `python -m agag.agcode`.
  The only requirement is that `agag` is importable from that interpreter —
  true for every `uv run` consumer already.
- Profile models keep the canonical `provider/name` spelling. `native_model`
  strips the prefix for agcode and claude_code; the canonical form is what
  travels in records.
- `run_harness` sets `cwd` **and** `PWD`. agcode resolves against
  `--working-dir` (default cwd), so there is exactly one base — which is the
  entire reason the wrong-base failures went 6/10 → 0/12.
- ollama has served the Anthropic Messages API since January 2026. No key, no
  account, no traffic to Anthropic. `total_cost_usd`-style figures from any
  harness are fictitious against a local backend.
- Isolate `HOME` for any claude_code role that stays: Claude Code loads the
  invoking user's global memory and instruction files into its system prompt,
  and that leak has already been observed here (`CLAUDE.md` reaching in-system
  agents).
- String-copy corruption is a model property, not a harness one. agcode
  reduces exposure; it cannot remove it. Where an agent only needs *content*,
  hand it the content (`--task-input` / `task_input=`) instead of a path.
- Expect the read-only doors to feel different: opencode denied a tool call
  and told the model so; agcode simply doesn't offer the tool. The second is
  usually better for weak models — there is no forbidden option to attempt.
- If the Omni Agent does work that belongs to an in-system agent (cagent,
  agautolab), leave the one-line note: "did X for agent Y — handoff
  candidate". That note is the whole obligation.
