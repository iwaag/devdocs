# Report 1 — pyagag: `codex` harness (2026-09-05)

Step 1 of `plan.md`. Landed as **pyagag `9f6797e`**, pushed to `main`.
445 tests pass (437 before, 8 new).

## What was probed first

Four real `codex exec --json` runs from a shell on agstudio (`codex-cli
0.153.2`, ChatGPT account, `gpt-5.4-mini` at `low`), kept in the session
scratchpad and used to check the extractor before the tests were written:

- `PONG` on `-s read-only`: the shape the plan describes, exit 0, one
  `agent_message`, `turn.completed.usage` with `cached_input_tokens` most
  of the input. With `-` as the only prompt argument stderr is **empty** —
  the "Reading additional input from stdin..." line the plan saw appears
  only when a prompt argument is also given.
- An unknown model: exit 1, the `item.completed`/`error` warning, then the
  top-level `error` and `turn.failed`, message = the API's 400 JSON as a
  string.
- A command plus a write on `read-only`: exit 0; the command ran
  (`item.started` then `item.completed` for `command_execution`), the
  write produced **no item at all** — the model reported the denial in its
  last message ("the workspace is read-only (`operation not permitted`)").
  Three `agent_message` items in that run, which is why the answer is the
  *last* one.
- A write on `workspace-write`: `file_change` arrives as `item.started`
  then `item.completed`, `changes: [{path, kind: "add"}]`, absolute path.
  `~/.codex/sessions/` had the same file count before and after —
  `--ephemeral` holds.

## What landed

- **`agent_config.py`**: `codex` in `CANONICAL_HARNESSES`,
  `INTRINSIC_CAPABILITIES` (`agentic_tools`, `workspace_fs`),
  `NATIVE_MODEL_HARNESSES`; `HARNESS_PROVIDER["codex"] = "openai"`; default
  command `codex`. Nothing in `PROVIDER_SECRETS`: the CLI owns its auth.
- **`harness.py`**:
  - argv: `codex exec --json --skip-git-repo-check --ephemeral --color
    never -C <cwd> -m <native> [-c model_reasoning_effort="<effort>"] -s
    <sandbox> [--add-dir d]… <extra_args> -`. `cwd` reuses the keyword agy
    added. The sandbox is `danger-full-access` for `skip_permissions` and
    when the caller names none; an `extra_args` carrying `-s`/`--sandbox`
    keeps it. The effort is `model_options["effort"]`, absent when the
    model declares none. Streamed and unwatched runs are the same argv.
  - The prompt goes through run_harness's existing stdin handoff untouched.
  - `_extract_codex`: last `agent_message` → output; `turn.completed.usage`
    → `usage`; `turn.failed` (or a top-level `error`) → `is_error`,
    `subtype: turn_failed`, and the message as the output when there was
    no message, so `codex exited 1: …` quotes the 400. `num_turns` = the
    number of `command_execution` + `file_change` items completed. A
    capture with no `turn.completed` invents nothing.
  - `_codex_events`: every raw event first, then claude-shaped —
    `agent_message` → `text`, `command_execution` → `tool_use` `shell`
    with `command` (on `item.started`, once per item id), `file_change` →
    `tool_use` `apply_patch` with `path` from the first change and the
    whole `changes` list; `reasoning` skipped.
- **Docs**: table row and a paragraph in `docs/agent-config-v1.md` ("five
  times" now), a word in the README.
- **Tests**: argv shape incl. the three sandbox rules and the effort flag;
  extraction of the captured success with the real usage numbers and a spy
  on `-C <cwd>`; `turn.failed` exit 1 → `failed` with the 400 quoted;
  read-only denial → `done` with the explanation as output; stream
  translation; non-JSON passthrough and a killed stream; config resolution
  with `effort` in `model_options`, provider `openai`, no secret, `~`
  expansion of an overlay command.

## Proven through `run_harness` with the real CLI

From this shell, `-s read-only` in `extra_args` (this harness's own
classifier is not in the way of `-s danger-full-access`, but a read-only
probe needs nothing more):

| run | exit | outcome | note |
|---|---|---|---|
| `openai/gpt-5.4-mini`, "Reply with exactly PONG." | 0 | `done` | output `PONG`, 3.6 s, usage input 12814 / cached 4352 / output 18 / reasoning 10, `num_turns 0`, no cost in the record |
| `openai/no-such-model-xyz` | 1 | `failed` | `failure: codex exited 1: {"type":"error","status":400,…"The 'no-such-model-xyz' model is not supported when using Codex with a ChatGPT account."}`, `subtype turn_failed` |

## Decisions the plan left open

- **`openai` only**, bound in `HARNESS_PROVIDER`. The Ollama route
  (`--oss --local-provider ollama`) is not wired; an `ollama/*` profile on
  `codex` is `E_INCOMPATIBLE` (tested), not a runtime surprise.
- `num_turns` is the tool-item count, and the docs say so next to agy's
  user-message count and claude_code's API-turn count.
- `-C <cwd>` is always passed when run_harness knows the cwd (it always
  does), as the second working-directory defense.
- `--add-dir` is passed for `add_dirs` — in Codex it means an extra
  *writable* directory, which is what a caller adding a directory wants.

## Not done here

- No `danger-full-access` run was made from this shell; the first one is
  step 6's, through a listener, as with agy.
