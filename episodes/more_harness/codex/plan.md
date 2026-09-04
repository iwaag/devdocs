# Plan — `codex` (OpenAI Codex CLI) as a sixth harness

Ask (2026-09-05): add Codex CLI (`codex-cli 0.153.2` at `~/.local/bin/codex`,
logged in with a ChatGPT account) as a pyagag harness and a selectable
profile for autolab and front. Same shape as `more_harness/agy` (this
folder's sibling): one pyagag commit, then pins and profiles move together.
No compatibility owed, no shim. AI-drafted from the Omni Agent's probing;
everything not marked *verified* is a hint, and the implementer decides.
Codex is the easiest of the three so far: the prompt comes on stdin, the
JSONL is flat and typed, and the bypass is an ordinary flag.

## Verified on agstudio (2026-09-05, from a shell)

**Invocation.** `codex exec [OPTIONS] [PROMPT]`. `-` as the prompt reads
stdin — run_harness's existing stdin handoff works unchanged. A 400 kB stdin
prompt was accepted. When stdin is piped *and* a prompt argument is given,
stdin is appended as a `<stdin>` block; stderr always says "Reading
additional input from stdin..." — noise, not an error.

**Trust gate.** In a directory that is neither a git repository nor listed
under `[projects]` in `~/.codex/config.toml`, `exec` exits 1 immediately:
"Not inside a trusted directory and --skip-git-repo-check was not
specified." A fresh `git init` was enough without the flag. Autolab's
workspaces are clones, front's topic workspaces are not — pass
`--skip-git-repo-check` always.

**`--json` output:** JSONL, exit 0 on success:
```
{"type":"thread.started","thread_id":"…"}
{"type":"turn.started"}
{"type":"item.completed","item":{"id":"item_0","type":"agent_message","text":"PONG"}}
{"type":"turn.completed","usage":{"input_tokens":14368,"cached_input_tokens":11008,"cache_write_input_tokens":0,"output_tokens":6,"reasoning_output_tokens":0}}
```
Item types seen: `agent_message` (`text`), `command_execution`
(`command`, `aggregated_output`, `exit_code`, `status`; an `item.started`
precedes each), `file_change` (`changes:[{path,kind}]`), `reasoning`
(`text`, seen on the local model), `error` (`message`). No cost field.
`-o <file>` writes the last agent message to a file as well; `--color
never` is implied by non-tty stdout but costs nothing to pass.

**Errors.** Unknown model: exit 1, an `item.completed`/`error` warning,
then `{"type":"error","message":…}` and `{"type":"turn.failed","error":
{"message":…}}` — the message is the API's 400 JSON as a string ("The
'no-such-model-xyz' model is not supported when using Codex with a ChatGPT
account"). A sandbox denial is **not** an error: the run exits 0 and the
agent explains in its last message ("this workspace is read-only").

**Sandbox and approvals — the important part.** `exec` never prompts
(approval policy is effectively `never`), so what a run may do is entirely
the sandbox flag `-s|--sandbox {read-only,workspace-write,
danger-full-access}`:

- Default in `exec` was **read-only**, even with `--skip-git-repo-check`:
  a write was "rejected by user approval settings" and stderr logged
  `writing is blocked by read-only sandbox`.
- `workspace-write` wrote `b.txt` in cwd, but `git commit` failed on
  `.git/index.lock` ("operation not permitted" — `.git` is protected) and
  `curl` to `127.0.0.1:11434` exited 7: **no network**.
  `-c sandbox_workspace_write.network_access=true` restored network; the
  `.git` block stayed.
- `danger-full-access` committed with git normally and needed no
  "dangerously" flag — approvals were already off. This is the working
  bypass for every role that commits, pushes, or runs `agentchat`
  (Zulip over HTTPS). `--dangerously-bypass-approvals-and-sandbox` exists
  too but adds nothing here, and the Omni Agent's own harness blocks it.
- `read-only` is a real read-only door: reads and shell commands run,
  writes fail with "operation not permitted", the model reports why.
  Good for `summarizer`.

**Environment.** Shell commands see the run's environment unfiltered:
`FOO_TOKEN`, `FOO_KEY`, `AGENTCHAT_ZULIP_ENV` all reached `env` inside a
command (`shell_environment_policy` default). Bare `env -i HOME PATH
~/.local/bin/codex exec …` answered, so launchd runs need no key plumbing;
`auth_mode` is `chatgpt`, tokens in `~/.codex/auth.json` (0600 plain JSON).
`~/.local/bin` is **not** on the listeners' PATH — overlay `command`, as
for agy.

**Models.** `~/.codex/models_cache.json` lists (ChatGPT account):
`gpt-5.6-terra` (the config default), `gpt-5.6-sol`, `gpt-5.6-luna`,
`gpt-5.5`, `gpt-5.4`, `gpt-5.4-mini`; efforts `low|medium|high|xhigh|max`
(`ultra` on 5.6). `-m <slug>` selects; effort is a config value,
`-c model_reasoning_effort='"low"'` (the value is parsed as TOML, hence the
inner quotes). Trivial runs: 3–5 s, ~14k input tokens, mostly cached.

**Local models, for free.** `codex exec --oss --local-provider ollama -m
qwen3.8:27b-mlx-bf16` answered PONG through the Ollama on this Mac in 111 s
(one harmless "failed to refresh available models" stderr line — the
catalog endpoint mismatch, not the run). So one harness can serve both
`openai/*` and `ollama/*` models if the provider is read from the model ID.

**Side effects.** Every run persists a session under `~/.codex/sessions/`
(162 MB today) and the log db `~/.codex/logs_2.sqlite` is 208 MB;
`--ephemeral` skips the session file and worked with `--json`. Codex reads
`AGENTS.md` from cwd and its parents, and the binary carries an importer
for Claude Code settings/`CLAUDE.md`/`.cursorrules` ("external config
migration") — watch for a prompt-shaped surprise on first run in a
workspace that has them (cf. the CLAUDE.md-leak memory). No timeout flag
exists; run_harness's kill is the deadline, so a killed run has no
`turn.completed` line and falls through to the timeout branch as today.

## Suggested design

- **Harness `codex`**, default command `codex`, capabilities
  `agentic_tools`, `workspace_fs`, native model names. Provider: `openai`
  for the ChatGPT catalog. If the Ollama route is wanted, do not bind the
  harness to one provider in `HARNESS_PROVIDER`; branch on
  `agent.provider == "ollama"` → `--oss --local-provider ollama`
  (`provider_base_url` is already in the environment as
  `AGENT_PROVIDER_OLLAMA_BASE_URL`; codex reads its own
  `OLLAMA_HOST`-style config, so the overlay's URL is informational
  unless mapped through `-c`). Simplest first cut: `openai` only.
- **argv:** `codex exec --json --skip-git-repo-check --ephemeral --color
  never -m <native> [-c model_reasoning_effort='"<effort>"'] -s <sandbox>
  [--add-dir d]… <extra_args> -` with the prompt on stdin. Sandbox from
  what the caller already says: `skip_permissions` → `danger-full-access`;
  an `extra_args` that names `-s`/`--sandbox` keeps it; otherwise
  `danger-full-access` too, since `workspace-write` cannot commit or reach
  Zulip and a role that only needs reads says so through its mode. Take
  the effort from the model's declared options in `agents.toml`
  (`[models."openai/gpt-5.6-terra"] effort = "low"`) — `model_options`
  already reaches `build_argv`. `-C <cwd>` is a free second defense for
  the working directory.
- **Extractor.** Walk the JSONL: output = the **last** `agent_message`
  text (the model narrates before acting, so the first one is a preamble);
  `turn.completed.usage` → `usage`; `turn.failed`/top-level `error` →
  `is_error` + `subtype` `turn_failed`, message quoted; count
  `command_execution` + `file_change` items as `num_turns` if a number is
  wanted (agy's 1-per-run was called out as a different unit — say which
  this is in the docs). No result line exists, so `_result_line` is not
  used; a clean run without `turn.completed` is a killed one.
- **Live progress.** autolab's `RunProgress` reads claude-shaped
  `assistant` events with `text`/`tool_use` blocks and detail keys
  `command|file_path|path|pattern|url`. Translate as agy does:
  `agent_message` → `text`; `command_execution` (on `item.started`) →
  `tool_use` named `shell` with `command`; `file_change` → `tool_use`
  named `apply_patch` with `path` from the first change. Skip `reasoning`.
- **Records.** `usage` only, no `cost_usd`; keep `cached_input_tokens` —
  it is most of every run.

## Steps

1. **pyagag** — `agent_config.py` (vocabulary, capabilities, native-model
   set, default command; provider binding or not, see above),
   `harness.py` (argv branch, stdin passthrough, extractor, stream guard,
   event translation), tests in the gemini/agy style: argv shape incl.
   sandbox choice; extraction of a captured success; `turn.failed` exit 1
   → `failed`; a read-only denial run → `done` with the explanation as
   output; stream translation. Docs table row, README line. Push.
2. **agents.toml** in agautolab and agfront:
   ```toml
   [models."openai/gpt-5.6-terra"]
   effort = "medium"
   [models."openai/gpt-5.4-mini"]
   effort = "low"
   [profiles.codex]
   harness = "codex"
   model = "openai/gpt-5.6-terra"
   ```
   A `codex-mini` profile is the cheap one for `summarizer` trials.
3. **Overlays** (`.local/agents.local.toml`, both): `[local.harness.codex]
   command = "~/.local/bin/codex"`, beside the agy entry.
4. **agautolab `role_run.py`** — `harness_args`: `READONLY_ROLES` get
   `["-s", "read-only"]`, everyone else the bypass via `skip_permissions`;
   one test each, plus "the profile resolves". agfront needs nothing if
   the harness defaults to full access.
5. **Pin** — `uv lock --upgrade-package pyagag` in agautolab, agfront,
   agforge; `uv run pytest` in each; kickstart the listeners and gateway
   (one re-served mention is one paid run).
6. **Prove it** — front on `codex` in `#front > front-codex-trial`;
   autolab `window` through the gateway; one `workrun-` task on `codex`
   in `pj-runsmoke2` so a commit and push through `danger-full-access`,
   the progress lines, and the 20-minute timeout path are seen once.
   Read the records: `harness: codex`, `usage`, `outcome: done`; one bad
   model → `failed` with the 400 quoted; confirm nothing was written
   under `~/.codex/sessions/` if `--ephemeral` is in the argv. If the
   Ollama route is included, one `summarizer` run on `ollama/qwen3.8:…`
   and its wall clock (expect minutes).

## Out of scope

- Mapping `allowed_tools` onto codex `.rules` execpolicy files.
- Reusing a thread across posts (`codex exec resume <id>`), tempting for
  `workrun-` topics.
- MCP servers, `codex review`, cloud tasks, cost estimation.
