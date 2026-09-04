# Plan — `agy` (Antigravity CLI) as a fifth harness

Ask (2026-09-05): add Google's Antigravity CLI (`agy` 1.1.26, at
`~/.local/bin/agy`, OAuth-logged-in) as a pyagag harness and a selectable
profile for autolab and front. Same shape as `more_harness/gemini`
(pj-agdev `devdocs/episodes/more_harness/gemini/`): one pyagag commit, then
the pins and profiles move together. No compatibility owed, no shim.
AI-drafted from the Omni Agent's probing; everything not marked *verified*
is a hint, and the implementer decides.

## Verified on agstudio (2026-09-05, all from a shell)

**Invocation.** `agy` uses Go flag parsing: `-p`/`--print` takes the prompt
as its *value*. `agy -p --output-format json` reads `--output-format` as the
prompt and errors; `-p=''` errors with "empty prompt". **stdin is not read as
the prompt.** Two working spellings:

- Prompt in argv: `agy -p='<prompt>' --output-format json --model <native>`.
  A 400 kB prompt passed argument parsing (macOS ARG_MAX is 1 MiB total).
- Prompt on stdin via `--input-format stream-json --output-format
  stream-json`: one NDJSON line per turn, shape
  `{"event":"user","message":{"role":"user","content":"<prompt>"}}`. Every
  other `event` value is ignored with a warning. Two lines ran two turns in
  one conversation (memory kept), and EOF ends the process with exit 0. The
  `-p` flag is not needed in this mode.

**`--output-format json`:** one document, exit 0 on success:
`{"conversation_id","status":"SUCCESS","response":"PONG\n","duration_seconds",
"num_turns","usage":{"input_tokens","output_tokens","thinking_tokens",
"cache_read_tokens","total_tokens"}}`. The response keeps its trailing newline.
No cost field.

**`--output-format stream-json`:** JSONL keyed by `event`, not `type`:
`{"event":"init","conversation_id","init":{"model","cwd","tools":[…],
"permission_mode":"request-review"}}`, then `{"event":"step_update",
"step_update":{"step_index","state":"ACTIVE|DONE|ERROR","step_type":
"user_input|agent_response|tool", "text_delta"?, "tool_name"?,
"tool_info":{"name","parameters":{…},"error"?}, "usage"?}}`, and last
`{"event":"result","result":{…the json document…}}`. The result payload is
**nested under `result`**, so pyagag's `_result_line` (which looks for
`type == "result"` at top level) does not find it as is.

**Errors.** Unknown model: exit 1, `status:"ERROR"`, `error` carries the
whole catalog. `--print-timeout` expiry (default **5m0s**): exit 1,
`error:"timeout waiting for response"`. Tool permission denial: **exit 0**,
`response:""`, plus `"denied_actions":[{"action":"command","display_name":
"RunCommand"}]` and a one-line stderr explanation.

**Permissions.** Headless mode auto-denies every tool that would prompt —
`run_command`, and also `read_file` (a `ListDir` was denied in default mode).
`--mode accept-edits` auto-approves file writes but not commands. `--mode
plan` answers with "I have prepared the plan in ~/.gemini/…/brain/<conv>/plan.md"
instead of doing the work, so it is not a read-only door the way gemini's
`plan` was. `--dangerously-skip-permissions` is the working bypass (not
probed by the Omni Agent — its own harness blocks that flag — but the CLI's
own denial message names it). The stderr hint also names a second route:
`permissions.allow` in `~/.gemini/antigravity-cli/settings.json` with grants
spelled `command(<target>)`, `read_file(<target>)` — the closest thing to
translating an `allowed_tools` grant; unexplored.

**Workspace — the trap.** cwd is *not* the workspace. Without `--add-dir`,
a `write_to_file` of "e.txt in the current directory" landed in
`~/.gemini/antigravity-cli/scratch/e.txt` (and once in `brain/<conv>/`),
even from a directory under the trusted `/Users/eiji/projects`. With
`--add-dir "$PWD"` the same prompt wrote `$PWD/f.txt`. Always pass
`--add-dir <cwd>`; `init.cwd` in the stream reports cwd but means nothing.
No trust prompt appeared in print mode from untrusted directories.

**Models.** `agy models`: `gemini-3.8-flash-{high,medium,low}`, 3.7, 3.6,
`gemini-3.1-pro-{high,low}`, `claude-sonnet-4-6`, `claude-opus-4-6-thinking`,
`gpt-oss-120b-medium`. Effort is baked into the name; `--model
gemini-3.8-flash --effort medium` is also accepted. The catalog is the
Antigravity account's, not Google's API — Claude is served through it.

**Auth and PATH.** Token is a plain 0600 file,
`~/.gemini/antigravity-cli/antigravity-oauth-token`. `env -i HOME=$HOME
PATH=/usr/bin:/bin ~/.local/bin/agy -p=… ` answered, so a launchd run should
too — no key file needed, unlike gemini. Putting `GEMINI_API_KEY` in the
environment changed nothing observable (same catalog, same run), but the
changelog says it changes how the CLI connects. `~/.local/bin` is **not** on
the listeners' PATH (`/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin` in
all three plists), so the overlay must name the command or the plists must
grow.

**Latency and tokens.** Trivial runs: 5–10 s wall, ~13.7k input tokens per
turn (the system prompt), `cache_read_tokens` appears on later runs. A
flash-low file write took 8 s.

**Side effects.** Every run becomes a conversation in
`~/.gemini/antigravity-cli/` (sqlite + `conversations/`, `brain/<conv>/`),
and the CLI log is `~/.gemini/antigravity-cli/cli.log` (`--log-file` can
redirect it, e.g. beside the transcript). Print mode expands slash
commands and skills; `--disable-slash-commands` keeps a prompt that starts
with `/` literal. The Omni Agent's probes left `c.txt`, `e.txt` in the CLI's
`scratch/` and a dozen conversations; harmless.

## Suggested design

- **Harness `agy`, provider `antigravity`**, model IDs `antigravity/<native>`
  (`gemini-3.8-flash-medium`, `claude-sonnet-4-6`, …). Not `google`: the
  catalog is Antigravity's, and `PROVIDER_SECRETS["google"]` would push
  `GEMINI_API_KEY` from the overlay into every run for no reason. Joins
  `CANONICAL_HARNESSES`, `INTRINSIC_CAPABILITIES` (`agentic_tools`,
  `workspace_fs`), `NATIVE_MODEL_HARNESSES`, `HARNESS_PROVIDER`; default
  command `agy`.
- **Prompt delivery.** Recommended: the stdin route (`--input-format
  stream-json --output-format stream-json`, one `{"event":"user",…}` line).
  It keeps run_harness's stdin handoff and `_run_streaming` untouched, has
  no ARG_MAX or `ps` exposure, and every run leaves a real transcript (the
  p10 lesson). The cost is that `stream=False` and `stream=True` produce
  the same argv — fine. Fallback: give `build_argv` a `prompt` argument for
  this one harness and pass `-p=<prompt>`; the JSON string must then be
  written to stdin by run_harness for agy, or stdin left empty.
- **argv:** `agy --input-format stream-json --output-format stream-json
  --model <native> --add-dir <cwd> [--add-dir d]… --print-timeout <N>s
  [--dangerously-skip-permissions] <extra_args>`. `build_argv` does not see
  cwd today; add a `cwd` keyword (run_harness has it) rather than relying
  on the caller's `add_dirs`. `--print-timeout` must be ≥ the run timeout
  or agy ends a 20-minute autolab run at 5 minutes; `int(timeout)` with an
  `s` suffix (Go duration) is enough, and slightly under it lets agy report
  its own timeout as a document. Permissions mirror gemini_cli: bypass when
  `skip_permissions` or when `extra_args` names no `--mode`; then agfront's
  listener needs no change, and autolab's `role_run` needs nothing beyond
  not giving `summarizer` a `plan` mode (see above — reads are denied
  without the bypass, so read-only roles get the bypass too; the
  `allowed_tools` grant stays as documentation). `--disable-slash-commands`
  is worth adding by default.
- **Extractor.** Parse the last `event == "result"` line and take its
  `result` (or the single document if someone runs `-o json`). `response`
  → output (rstrip one newline); `status == "ERROR"` → `is_error`, `subtype`
  from the first line of `error`; `duration_seconds * 1000` → `duration_ms`;
  `num_turns`, `usage` as they are; **`denied_actions` non-empty → keep it in
  meta** (`denied_actions`), because that run exits 0 with an empty response
  and would otherwise be a silent `empty_final`. Assistant text in the
  stream is the `text_delta` of `agent_response` steps, but the result line
  already carries the full `response`, so no reassembly is needed.
- **Live progress.** autolab's `RunProgress` (`zulip_listener.py`) reads
  claude-shaped `{"type":"assistant","message":{"content":[…]}}` events with
  `text`/`tool_use` blocks and detail keys `command|file_path|path|pattern|
  url`. agy emits `step_update` with `tool_name` and
  `tool_info.parameters` (`TargetFile` seen; the command-tool key is
  unknown until a bypass run shows it). Either translate in pyagag's agy
  branch — wrap `on_event` so a `tool` step becomes a `tool_use` block and
  an `agent_response` `text_delta` a `text` block — or widen the consumer.
  Translating keeps the consumer harness-agnostic, as agcode did.
- **Records.** `usage` only, no `cost_usd`; `provider: antigravity`. The
  mission-cost reader shows tokens for these runs, as for gemini.

## Steps

1. **pyagag** — `agent_config.py` (vocabulary, provider binding, default
   command), `harness.py` (argv branch with cwd, stdin message, extractor,
   stream guard, optional event translation), tests in the style of the
   gemini ones: argv shape; result-line extraction with the nested
   payload; bad model exit 1 → `failed`; denial exit 0 → `done` +
   `empty_final` + `denied_actions`; non-JSON passthrough. Table row in
   `docs/agent-config-v1.md`, a line in the README. Push; note the hash.
2. **agents.toml** in agautolab and agfront:
   ```toml
   [models."antigravity/gemini-3.8-flash-medium"]
   [models."antigravity/claude-sonnet-4-6"]
   [profiles.agy]
   harness = "agy"
   model = "antigravity/gemini-3.8-flash-medium"
   ```
   Add a second profile for the Claude model if wanted — Agent ≠ Model, and
   the record says which ran. `GET /projects` lists profiles from the file,
   so `agy` shows up with no gateway change.
3. **Overlays** (`.local/agents.local.toml`, both): `[local.harness.agy]
   command = "~/.local/bin/agy"` — `_resolve_command` expands `~`. Or put
   `~/.local/bin` on the three plists' PATH and reload with bootout +
   bootstrap (kickstart does not re-read a plist; see devenv.md).
4. **agautolab `role_run.py`** — `harness_args`/`skip_permissions` for
   `agy`: bypass for every role; one test that the profile resolves, one
   that `summarizer` gets the bypass rather than a mode. agfront's listener
   needs nothing if the harness defaults to the bypass.
5. **Pin** — `uv lock --upgrade-package pyagag` in agautolab, agfront,
   agforge; `uv run pytest` in each; kickstart the listeners and the
   gateway. A kickstart re-serves one pending mention (one paid run).
6. **Prove it** — front on `agy`: uncomment `[roles.front] profile = "agy"`
   in `agfront/.local/agents.local.toml` (read per run, no restart), post
   once into `#front > front-agy-trial`, read the record under
   `agfront/.local/topics/front/<topic>/<N>/front/`. autolab: `summarizer`
   or a `window` run through the gateway, then one `workrun-` task on `agy`
   through the acceptance route so the progress display and the
   20-minute timeout are seen once. Read `run-NNNN.json`: `harness: agy`,
   `usage`, `num_turns`, `outcome: done`; one bad model name → `failed`;
   and confirm the run's files landed in the workspace, not in
   `~/.gemini/antigravity-cli/scratch/`. Record what the command tool's
   parameter key is called, for the progress translation.

## Out of scope

- Translating `allowed_tools` into `permissions.allow` grants.
- Keeping a conversation open across topic posts with `--conversation` or
  a long-lived stream-json stdin (tempting for `workrun-` topics; later).
- Cost estimation; `--sandbox`; MCP servers; agforge/arxivsage opting in
  (they inherit the harness with the pin).
