# Step 2 report — the skeleton in pyagag, agforge on it

## pyagag (commit `7c29696`, pushed)

New modules:

- `agag/agent.py` (383 lines): `AgentSpec` (short name, root, `plan_prefix`,
  `run_prefix`, optional `extra_environment`) fixing every path the copied
  modules used to spell (`.local/zulip.env`, `.local/instance.toml`,
  `agents.toml` + `.local/agents.local.toml`, `params/intro.md`,
  `agent/guides`, `.local/topics`, `.local/agent`) and the env var names
  (`<AGENT>_INSTANCE_NAME`, `<AGENT>_ZULIP_LOG_ONLY`, plus a global
  `AGAG_ZULIP_LOG_ONLY`). `chat_environment` / `resolve_spec_role` /
  `run_role` are the shared part of the three `role_run.py`; `topic_filter`
  and `listener_main(spec, {prefix: handler}, dm_handler=, on_mention=)` are
  forge's `zulip_listener` generalized, with anything matching no prefix sent
  to the entrance; `intro_main(spec)` is `intro.py`. `SWEEP_ACK` /
  `ACK_PREFIX` / `is_ack` moved here from forge's `zulip_chat`.
- `agag/entrance.py` (165 lines): forge's `entrance_topic` for any spec.
  `DEFAULT_GUIDE` is built in and gets `{plan_prefix}`/`{run_prefix}` filled
  in (or degrades to "lists your conversations." when an agent has no
  prefixes); `agent/guides/entrance_front/guide.md` wins when present.
- `agag/agent_config.py`: `ag.agent-config.v2` = v1 + required
  `[roles.X] allowed_tools` (string or array), exposed as
  `ResolvedAgent.allowed_tools`. v1 still loads (`allowed_tools` → `None`).
  `docs/agent-config-v1.md` has a v2 section; README has a skeleton section.
- `agag/plane.py`: `credentials_path(root)` = `AGAG_PLANE_ENV` or
  `<root>/.local/plane-credentials.env`.
- Tests: `tests/test_agent.py` (13) + 3 in `test_agent_config.py`; 395 pass.

## agforge (commit `cb37dc7`, pushed; pin bumped 1db9150 → 7c29696)

| module | before | after |
|---|---|---|
| `instance.py` | 40 | 47 (defines `SPEC`, the two prefixes, keeps `instance_name(path)` for callers) |
| `intro.py` | 27 | 29 |
| `role_run.py` | 205 | 128 (only `tool_environment` is forge's; grants gone) |
| `zulip_listener.py` | 119 | 57 (routes table + DM route; `dispatch`/`topic_filter` kept for tests) |
| `entrance_topic.py` | 130 | 36 |

`agents.toml` is `ag.agent-config.v2`; `front` and `generator` carry their
grants there. `agent_run.py` (the `:8092` charter path) reads
`agent.allowed_tools` instead of `CLAUDE_ALLOWED_TOOLS`. `plane.py` uses
`credentials_path`; the shared file is symlinked as
`agforge/.local/plane-credentials.env` (local only).

Tests: the ones that exercised lifted code (`test_entrance_topic` serving
tests, `test_role_run` wiring) were replaced by checks of forge's own
guide/grants/wiring; 197 pass.

## Checks

- `AGFORGE_ZULIP_LOG_ONLY=1 uv run python -m agforge.zulip_listener`:
  `agforge zulip listener starting (log only) (pull sweep: all topics in
  'agforge-agstudio1', prefixes ('assetrun-', 'assetplan-') elsewhere,
  routes ['assetplan-', 'assetrun-'] + DM thread)` → full sweep OK.
- `launchctl kickstart -k gui/$(id -u)/com.agdev.agforge-zulip` restarted
  the real listener on the new code.
- Real Zulip: posted `assetplan-skeleton-check-icon` in `#agforge-agstudio1`
  as the Omni Agent. forge acked (`Message received…`), ran the front, asked
  one question (my wording "register it" collided with its own
  "I register it as a Work" vocabulary — a wording artifact, not a skeleton
  fault), and on the answer registered the plan:
  `created F2-22 "Plan: Paper Plane Icon (PNG)" in FreeForge / posting in
  assetrun-skeleton-check-icon starts it`. Plane was reached through
  `credentials_path`. The run was not triggered (not needed for the check).

## Left in agforge for a later phase

- `zulip_chat.py` DM route (charter run) — forge-only, passed in as `dm_handler`.
- `tool_environment` (ACE Studio, `.local/bin`, `scripts/` on PATH) — attached
  as `SPEC.extra_environment`.
- `anchor.py`'s `[selfnote][work]`, `plane.py` label/project choice.
- `listener.topic_filter`/`dispatch` wrappers exist only so the old tests
  still read; they could go with those tests.
