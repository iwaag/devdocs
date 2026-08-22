# Step 1 report — the skeleton map

Deliverable: [skeleton_map.md](skeleton_map.md).

## What was done

- Read agforge's `instance.py`, `intro.py`, `zulip_listener.py`, `role_run.py`,
  `entrance_topic.py`, `zulip_chat.py`, `anchor.py`, and the consumers of
  `AGFORGE_ROOT` / `run_role` (`assetplan_topic`, `assetrun_topic`, `agent_run`,
  `plane`, `toolsets`).
- Read all three `role_run.py` (forge 205 / autolab 196 / front 133) and both
  `entrance_front/guide.md`.
- Confirmed with `diff <(sed …)` that autolab's `instance.py` and `intro.py`
  differ from forge's only in docstrings.
- Read the pyagag pieces the skeleton builds on: `agent_config` (v1 overlay
  rules), `topics.serve_topic`, `zulip.sweep_serve` (owner sweep + mention
  route), `intro.post_intro`, `harness.run_harness` signature.

## Findings that shape Step 2

1. The shared core of `role_run` in all three agents is exactly three things:
   config-pair loading, the `agentchat` handover (PATH + `AGENTCHAT_ZULIP_ENV`
   + `AGENTCHAT_HOME`), and `run_harness` → run record. Everything else is
   per-agent (forge's ACE/PATH env, autolab's agcode args and
   `skip_permissions`).
2. `ROLE_ALLOWED_TOOLS` is repeated per agent with the same warning comment;
   it moves to `agents.toml [roles.X] allowed_tools` (config schema v2).
3. The entrance serving is identical between forge and autolab; only the guide
   text is per-agent. pyagag will ship a default guide with
   `{plan_prefix}`/`{run_prefix}` filled in, overridden by
   `agent/guides/entrance_front/guide.md` when the agent has one.
4. forge's Plane credential lookup is parent-directory relative
   (`ROOT.parent/.local/plane-credentials.env`); the skeleton resolves
   `AGAG_PLANE_ENV` or `<root>/.local/plane-credentials.env` instead.
5. agfront's `on_mention` route and agautolab's extra harness args are not
   moved this phase, but `listener_main` / `run_role` take them as optional
   arguments so the later move is mechanical.

No code changed in this step.
