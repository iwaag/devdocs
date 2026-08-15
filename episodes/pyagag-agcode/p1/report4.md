# Report 4 — agforge

Status: **done**. agforge suite green (126 passed, was 129 — three
opencode-shaped config tests collapsed into two agcode ones and one was
dropped as obsolete).

## The decision: kept `[profiles.local]`, on agcode

The plan's primary instruction was to delete `[profiles.local]` because
nothing runs on it (both roles are `sonnet`), with "add `harness = "agcode"`"
offered as optional. I took the option.

Reasons:

- It costs nothing to keep. agcode needs no install, no grant file, and no
  `[local.harness.agcode]` overlay block — its default command is
  `sys.executable`, already the interpreter importing `agag`. Deleting the
  opencode profile removed a real maintenance burden; keeping an agcode one
  adds none back.
- The braindump's sentence is "local models run on agcode from now on".
  agforge would have been the one project where the local option disappeared
  instead.
- The plan's own reasoning for why `run` alone may be enough here is correct,
  and it is now checked rather than assumed.

**It is not dead config.** One live `generator` run on the `local` profile,
`profile_override="local"`, against ollama on `agstudio.home.arpa:11434`:

```
task    Run the shell command `ls scripts` and reply with the file names it
        prints, nothing else.
reply   __pycache__ / generate.py / generate.sh
record  harness=agcode  model=ollama/qwen3.6:35b-a3b-coding-nvfp4
        outcome=done  num_turns=2
```

The `run` tool alone served it, as predicted. agforge's work really is
image/audio tooling behind shell commands.

## Changes

- `agents.toml`: `[profiles.local] harness` → `"agcode"`.
- Deleted `opencode.json` (a 60-line bash allow-list). agcode has no
  permission engine; `run` is the whole shell surface, and the two safety
  devices agforge documents are unaffected — both guard irreversible harm
  (`generate.py` refusing the `nctl-outbox` bucket, no skip-permissions flag),
  not mistakes.
- `README_DEV.md`: the `opencode.json` file entry, and the safety-devices
  paragraph (agcode has no `opencode run --auto` equivalent to refuse).
- `service/GUIDE.md`: the profile list and the cost line. The cost line now
  says none is *reported* rather than a zero being invented — matching what
  agcode does.
- `.local/agents.local.toml` (ignored): dropped the opencode command, and the
  ollama base_url lost its `/v1` suffix.
- `uv.lock`: pyagag pin → `b705fd97`.

## Tests

- `tests/test_agent_config.py`: `BASE` and the overlay fixtures now spell
  `agcode`. Two opencode-specific tests were rewritten rather than deleted,
  because both were testing a *difference* worth keeping in the suite:
  - `test_agcode_takes_the_native_model_and_keeps_the_canonical_one` — the
    full argv is asserted, showing `--model local-model` on the wire while
    `resolved.model` stays `ollama/local-model` for the record, and
    `resolved.command == sys.executable` with no overlay block.
  - `test_agcode_needs_no_declared_provider_endpoint` — opencode's resolution
    refused an ollama profile with no `base_url`; agcode resolves, and
    `--base-url` is simply absent from the argv. That is the
    opencode-specific check Step 8 removes, pinned from the other side.
- `tests/test_service.py`: the `# --- opencode event-stream extraction ---`
  header is now `# --- event-stream extraction (the fake harness passthrough) ---`.
  The tests themselves stay (see the warning below).

## One Step-9 item pulled forward

`tests/test_agent_config.py` loads the shared fixtures in
`devpolicy/contracts/agent/examples/valid/`, and asserts every resolved
harness is in a known set. Leaving those fixtures on `opencode` would have
left agforge's suite red, so `valid/{agforge,agautolab,agdevworld}/agents.toml`
and `valid/agforge/agents.local.toml` now spell `agcode`
(devpolicy `c82a2e4`). The `invalid/` fixtures, `spec.md`, and
`agent_records.md` are untouched and remain Step 9's work — the `invalid/`
ones still resolve because `opencode` is still a canonical harness until Step 8.

## Warning for Step 8

`extract_event_text` is **not** callerless. `run_harness`'s `else` branch uses
it for the `fake` harness as well as opencode, and agforge re-exports it from
`src/agforge/agent_run.py` with four tests covering it
(`tests/test_service.py`). Deleting it in Step 8 means giving `fake` an
explicit passthrough extractor first, and updating those tests — not just
removing a dead function. The plan's parenthetical "check the tests before
deleting" is the right instinct; this is what the check found.
