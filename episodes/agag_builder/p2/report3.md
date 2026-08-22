# agag_builder p2 — step 3: agfront on the skeleton

agfront `ef70baf`. `uv run pytest`: 20 passed (was 24; see "tests").

| file | before | after | what |
|---|---|---|---|
| `role_run.py` | 133 | — | deleted outright: Front adds nothing to `agag.agent.run_role`. `ROLE_ALLOWED_TOOLS["front"]` → `agents.toml` v2 `allowed_tools = "Read,Glob,Grep,Bash(agentchat:*)"` |
| `instance.py` | — | 26 | `SPEC = AgentSpec("front", ROOT, plan_prefix="front-")` |
| `intro.py` | — | 8 | `intro_main(SPEC)`; `params/intro.md` written, **not posted** (report1) |
| `listener.py` | — | 29 | `listener_main(SPEC, {"front-": handle_topic}, on_mention=handle_mention)` |
| `zulip_listener.py` | 310 | 242 | lost `main`, `observe_topic`, `topic_workspace`/`generation_dir`/`next_record_path`/`is_ack` copies and the ACK/EMPTY constants; keeps `serve`, `handle_topic`, `handle_mention`, `front_prompt`, `run_front`, the `write_agents_md` call |
| `agent/zulip_listen.sh`, `pyproject.toml` script | | | `agfront.listener` |

Five concerns: 443 → 305 lines, 242 of them Front's own serving.

Env var renames that fall out of the spec: `AGFRONT_ZULIP_LOG_ONLY` →
`FRONT_ZULIP_LOG_ONLY` (`AGAG_ZULIP_LOG_ONLY` works too); `FRONT_INSTANCE_NAME`
is new. Nothing in launchd/devenv referenced the old name.

## Tests

- `test_role_run.py` 10 → 2: the `tool_environment`/`resolve_agfront_role`
  tests were of the skeleton's `chat_environment`/`resolve_spec_role`
  (pyagag tests them); `test_every_role_carries_a_tool_grant` is the v2
  schema now. Kept: Front's grant shape, and the end-to-end stub run —
  re-pointed with `replace(SPEC, root=tmp_path)` instead of patching two
  config-path globals, so the "nothing falls back to the paid harness"
  property is the spec's root, not a pair of monkeypatches.
- `test_zulip_listener.py`: unchanged except one added test of what
  `listener.main` hands the skeleton (one route, `on_mention`, `front-` as
  the only swept prefix).

No skeleton difference surfaced here: Front's `main` was already the
skeleton's shape minus the own-channel sweep, and the own channel does not
exist.
