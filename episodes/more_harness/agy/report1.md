# Report 1 — pyagag: `agy` harness (2026-09-05)

Step 1 of `plan.md`. Landed as **pyagag `955529e`**, pushed to `main`.
437 tests pass (429 before, 8 new).

## What was probed first

Three real runs from a shell on agstudio, kept in the session scratchpad and
used to check the extractor before the tests were written:

- `PING` on `gemini-3.8-flash-low` over stdin stream-json: the shape the plan
  describes, exit 0, `response: "PONG\n"`, ~13.8k input tokens.
- An unknown model: exit 1, one `result` line, `status: ERROR`, the catalog in
  `error`, `num_turns: 0`.
- A `run_command` without the bypass: exit 0, `response: ""`,
  `denied_actions: [{"action":"command","display_name":"RunCommand"}]`, and
  the tool step in the stream carries **`tool_info.parameters.CommandLine`**
  — the command-tool parameter key the plan wanted recorded. The step ends
  with `state: ERROR` and `tool_info.error.type: TOOL_ERROR`.

A bypass run (`--dangerously-skip-permissions`) could not be probed from this
harness — its own permission classifier blocks the flag, as the plan
anticipated — so the first bypass run will be step 6's, through a listener.

## What landed

- **`agent_config.py`**: `agy` in `CANONICAL_HARNESSES`, `INTRINSIC_CAPABILITIES`
  (`agentic_tools`, `workspace_fs`), `NATIVE_MODEL_HARNESSES`;
  `HARNESS_PROVIDER["agy"] = "antigravity"`; default command `agy`. Nothing
  in `PROVIDER_SECRETS`, so an overlay's `google_api_key_file` does not
  reach an agy run (tested).
- **`harness.py`**:
  - `build_argv` gained `cwd` and `timeout` keywords; run_harness passes
    both, the other harnesses ignore them. agy's argv is
    `agy --input-format stream-json --output-format stream-json --model
    <native> --disable-slash-commands --add-dir <cwd> [--add-dir d]…
    --print-timeout <timeout-10>s [--dangerously-skip-permissions]
    <extra_args>`. The bypass is on unless the caller's `extra_args` names a
    `--mode`; `skip_permissions` forces it. `stream=True` and `False` give
    the same argv.
  - The prompt is written to stdin as one `{"event":"user",…}` NDJSON line
    (`_agy_stdin`); `_run_streaming` and the `subprocess.run` path are
    untouched apart from that payload.
  - `_extract_agy` takes the `result` payload of the last `event == "result"`
    line (or a bare `-o json` document): `response` minus one trailing
    newline → output; `status == "ERROR"` → `is_error` with `subtype` the
    first line of `error` (capped at 120 chars) and the whole `error` as the
    output when the response is empty, so `agy exited 1: …` quotes the
    catalog; `duration_seconds*1000`, `num_turns`, `usage` as they are;
    `denied_actions` kept in meta when non-empty. `write_run_record` copies
    `denied_actions` into the record next to `empty_final`.
  - `_agy_events` wraps a progress consumer: every raw event first, then a
    claude-shaped `{"type":"assistant","message":{"content":[block]}}` — one
    `text` block per finished `agent_response` step (deltas gathered until
    `DONE`), one `tool_use` block the first time a `tool` step is seen, with
    the parameters as agy spells them plus snake_case doubles
    (`CommandLine`→`command`, `TargetFile`→`file_path`, …). autolab's
    `RunProgress` needs no change.
- **Docs**: table row and a paragraph in `docs/agent-config-v1.md`, a word in
  the README.
- **Tests** (mirroring the gemini set): argv shape and bypass rules;
  nested-payload extraction with the real usage numbers and a spy on the
  argv's `--add-dir <cwd>`; unknown model exit 1 → `failed` with the catalog
  in the failure; denial exit 0 → `done` + `empty_final` + `denied_actions`
  in meta and record; stream translation for a claude-shaped reader; non-JSON
  passthrough and a bare document; config resolution to `~/.local/bin/agy`
  via the overlay's `command`, provider `antigravity`, no secret.

## Decisions the plan left open

- Provider name `antigravity`, as suggested; the model ID in the tests is
  `antigravity/claude-sonnet-4-6` on the config side and
  `antigravity/gemini-3.8-flash-medium` on the harness side.
- `--print-timeout` margin is 10 s under the run timeout, floor 1 s. A
  caller that passes no timeout to `build_argv` gets no flag (CLI default).
- `--disable-slash-commands` is always on: a prompt that starts with `/` is
  a prompt.
- The translation emits both spellings rather than replacing agy's, so a
  consumer that learns agy's own keys later loses nothing.

## Not done here

- No bypass run was seen (blocked from this harness); the parameter keys
  other than `CommandLine`/`TargetFile` in `AGY_PARAMETER_KEYS` are guesses
  after Antigravity's PascalCase, and are labelled so in the code.
