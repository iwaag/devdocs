# Report 8 — pyagag: delete the opencode harness

Status: **done**. pyagag green (194 passed). All five consumers re-pinned and
green: agautolab 78, agdevworld 129, agforge 124, cagent 159, nctl 1315.

## Deleted

**`agent_config.py`**

- `"opencode"` out of `CANONICAL_HARNESSES` and `INTRINSIC_CAPABILITIES`.
- The `"opencode": "opencode"` default command.
- The ollama-`base_url`-required check. It refused an ollama profile with no
  declared endpoint because the harness of the day could not be pointed at one
  without it; agcode carries `http://localhost:11434` as its own default, so
  absence is now legal.

**`harness.py`**

- The `opencode` branch of `build_argv`. `build_argv` now raises
  `unsupported harness: 'opencode'` — pinned by a new test, because a closed
  vocabulary refusing an unknown member is the behaviour, not an accident.
- The `opencode_config` parameter and the `OPENCODE_CONFIG` environment write.
- `extract_event_text`.

**Docs**: `docs/agent-config-v1.md` (harness table row, the committed and
overlay examples, the model-spelling paragraph, the run-record example) and
`README.md`.

## Two things the plan expected to be simpler than they were

### 1. `extract_event_text` was not callerless

The plan's parenthetical — "it parses opencode's event stream and has no other
caller — check the tests before deleting" — was the right instinct, and report
4 flagged what the check found: `run_harness`'s `else` branch used it for the
**`fake`** harness too, and agforge re-exported it from
`src/agforge/agent_run.py` with four tests.

`fake` now gets an explicit extractor:

```python
else:
    # `fake`: whatever the stub printed is the output, minus the trailing
    # newline `print` adds, and no statistics of its own.
    output, reported = raw.rstrip("\n"), {}
```

The `rstrip` is not cosmetic. `extract_event_text` ended with
`"\n".join(texts)`, which dropped the trailing newline as a side effect of
splitting and rejoining. A bare passthrough kept it, and
`test_fake_argv_and_environment_injection` caught the difference immediately.
Making it explicit keeps every `fake`-harness consumer byte-identical.

agforge's four event-stream tests are replaced by one that asserts the new
contract: a stub's output reaches the caller unparsed, with no `cost_usd` and
no `num_turns` invented for it.

### 2. `project_name` became dead, and two consumers passed it

`resolve_role(..., project_name=...)` existed for exactly one diagnostic — the
`local.provider.ollama.base_url is required by {project_name} OpenCode`
message. With that check gone the parameter had no reader, so it went too.

agautolab and agforge both passed it positionally-by-keyword and broke with
`TypeError: resolve_role() got an unexpected keyword argument 'project_name'`.
Both fixed. (My first grep for callers missed them by searching the wrong
directories; the consumer test runs found them.)

## Consumer re-pinning

Every consumer resolves pyagag from GitHub, not as a path dependency, so each
needed `uv lock --upgrade-package pyagag`: `b705fd97` → `f9b7723b`. This is
the fourth step where that has been true, so `README.md`'s "Consumers in the
sibling workspace use editable uv path dependencies" was simply wrong; it now
says what actually happens and what a consumer has to run.

## One Step-9 item pulled forward

`devpolicy/contracts/agent/examples/invalid/{capability-unmet,unknown-model,
overlay-out-of-scope}.toml` all used `harness = "opencode"`. With opencode out
of `CANONICAL_HARNESSES` they started failing as `E_UNKNOWN_HARNESS` instead of
their declared codes, turning agforge's contract-conformance suite red. All
three now spell `agcode` and test the same failure, as the plan asked. The
`unknown-harness.toml` fixture is untouched — it uses `ollama`, and the point
it makes (a provider is not a harness) is unaffected.

`spec.md` and `agent_records.md` are still Step 9's.

## Verification

- `pytest` in pyagag: 194 passed.
- Each consumer re-locked, re-synced, and run: all green.
- `cagent-api` and the autolab gateway restarted on the new pin, then a live
  window turn:

  ```json
  {"backend": {"harness": "agcode", "provider": "ollama",
               "model": "ollama/qwen3.6:35b-a3b-coding-nvfp4",
               "role": "window", "profile": "local"}}
  ```

- `grep -rn -i opencode src/ tests/ docs/ README.md` in pyagag returns one
  line: the `test_an_unsupported_harness_is_rejected` case that asserts the
  name is now refused.
