# agag_builder p1 — skeleton map (Step 1)

Three columns per module: **lift** (move into pyagag), **template** (stays in
the generated project, but as a few lines that call pyagag), **own** (that
agent's specific code, untouched by this phase). Base line is agforge;
agautolab and agfront are diffed against it.

## agforge `src/agforge/`

| module (lines) | lift to pyagag | template | own |
|---|---|---|---|
| `instance.py` (40) | all of it → `AgentSpec.instance_name()` = `agag.instance.instance_name(root/.local/instance.toml, fallback=agent, env_var=<AGENT>_INSTANCE_NAME)` | — | nothing (docstring only differs from autolab) |
| `intro.py` (27) | `agag.agent.intro_main(spec)`: `post_intro(client, instance, root/params/intro.md, root)` | `python -m <agent>.intro` = 3 lines calling it | nothing |
| `zulip_listener.py` (119) | `topic_filter` (own channel + prefixes), `main` (LOG_ONLY switch, DM thread, `sweep_serve`), `observe_topic`/`handle_message` (log-only handlers) → `agag.agent.listener_main(spec, dispatch)` | `listener.py`: build `AgentSpec`, a `dispatch` dict `{prefix: handler}`, call `listener_main` | `assetplan-`/`assetrun-` handlers, the DM `react` route (charter run) |
| `role_run.py` (205) | config pair loading, `chat_environment` (agentchat on PATH + `AGENTCHAT_ZULIP_ENV` + `AGENTCHAT_HOME`), `run_role` (resolve → `run_harness` → run record); **`ROLE_ALLOWED_TOOLS` → `agents.toml [roles.X] allowed_tools`** read by `agag.agent_config` (schema v2) | `agents.toml` carries the grants | `tool_environment` (ACE_STUDIO_CLI, `.local/bin`, `scripts/` on PATH) — passed to the skeleton as `spec.extra_environment` |
| `entrance_topic.py` (130) | whole serving: workspace under `.local/topics`, chatlog, front run with transcript, `handle_entrance` → `agag.entrance` | — | guide text (`agent/guides/entrance_front/guide.md`, 8 lines): only the `assetplan-/assetrun-` vocabulary is forge's; pyagag ships a default guide with `{plan_prefix}/{run_prefix}` substitution, the agent's file wins when present |
| `zulip_chat.py` (163) | `SWEEP_ACK`, `ACK_PREFIX`, `is_ack` → `agag.agent` constants | — | the DM charter run (`react`, `format_transcript`, `DESIRE_TEMPLATE`) |
| `anchor.py` (76) | — (`[selfnote][rootchat]` already lives in `agag.selfnote`) | — | `[selfnote][work]` tag, `assetrun_topic_name` |
| `plane.py` (252) | only the credential lookup: `AGAG_PLANE_ENV` env var or `<root>/.local/plane-credentials.env` (today `ROOT.parent/.local/…`, which breaks outside `pj-agdev/`) → `agag.plane.credentials_path(root)` | — | label/project choice, Work registration |
| `assetplan_topic.py`, `assetrun_topic.py`, `agent_run.py`, `generate.py`, `comfy_*.py`, `toolsets.py`, `transform.py`, `works.py`, `request_service.py`, `cli.py` | — | — | forge's work |
| `service/listen.sh` (4) | — | copied verbatim into the template (`exec uv run python -m <agent>.listener`) | — |
| `service/zulip_listener.py` | — | — | compatibility launcher, keep |
| `params/intro.md` | — | template skeleton with `{instance}`, prefixes and a TODO | forge's text |
| `instance.example.toml` | — | template verbatim | — |
| `agents.toml` | — | template: `ag.agent-config.v2` with `allowed_tools` per role | forge's `generator` role |

## agautolab — diff against forge

| module | verdict |
|---|---|
| `instance.py` (45) | identical modulo docstring (`diff <(sed s/agforge/X/g …)`) → lift |
| `intro.py` (34) | identical modulo docstring → lift |
| `agent_settings.py` (25) | = forge's `resolve_agforge_role` without tool env → lift (same `run_role` core) |
| `role_run.py` (196) | `tool_environment` = forge's `chat_environment` (same three things). Own: per-project profile override (`load_project_roles`), `ROLE_WORKSPACES`, agcode `extra_args` (`--max-turns`, deadline, max tokens), `skip_permissions=True` under claude_code. The skeleton takes `skip_permissions` and `extra_args` as arguments so autolab can be moved later without losing them. **Out of scope this phase** (plan). |
| `zulip_listener.py` (1254) | mission-specific; only `topic_filter`/`main` shape is shared. Not read for the skeleton. |
| `guides/entrance_front/guide.md` | same skeleton as forge's, different vocabulary (`pj-`/`work-`/`workplan-`). Confirms: the guide is per-agent content, the serving is shared. |

## agfront — diff against forge

| module | verdict |
|---|---|
| `role_run.py` (133) | = forge's `chat_environment` + `run_role` with no tool env. Grant `front = Read,Glob,Grep,Bash(agentchat:*)`. → the skeleton's `run_role` covers it 1:1. |
| `zulip_listener.py` (310) | `front-` prefix owner + `on_mention` route + `note_served`; `write_agents_md` into the run's `tools/`. Own to front this phase. The skeleton's `listener_main` accepts `on_mention` so front can move later. |
| no `instance.py` / `intro.py` | front has no instance name and posts no intro (it is the developer's agent) |

## `ROLE_ALLOWED_TOOLS` as seen in the three agents

| agent | role | grant |
|---|---|---|
| agforge | front | `Read,Write,Edit,Glob,Grep,Bash(agforge:*),Bash(agentchat:*)` |
| agforge | generator | 50-entry list (media tools) |
| agfront | front | `Read,Glob,Grep,Bash(agentchat:*)` |
| agautolab | front, director, mediator, coding, superdirector, supercoder | `WORKING_ALLOWED_TOOLS` (≈60 entries) |
| agautolab | summarizer | `Read,Glob,Grep` |

All three repeat the same comment: *a role missing from the table gets no
`--allowedTools` and claude_code waits for a permission answer until the
timeout*. Moving the grant into `agents.toml` beside the role makes the table
and the role list one thing, so the "every role belongs here" test
(`test_every_role_carries_a_tool_grant`) becomes a schema rule.

## Path conventions the skeleton fixes

| thing | today | skeleton |
|---|---|---|
| bot credentials | `<root>/.local/zulip.env` | same (`AgentSpec.zulip_env`) |
| instance name | `<root>/.local/instance.toml`, `<AGENT>_INSTANCE_NAME` | same |
| agents overlay | `<root>/.local/agents.local.toml` | same |
| Plane credentials | forge: `ROOT.parent/.local/plane-credentials.env` | `AGAG_PLANE_ENV` or `<root>/.local/plane-credentials.env` (no parent-relative lookup) |
| topic workspaces | `<root>/.local/topics/<ch>/<topic>/<N>/<role>/` | same |
| run records | `<root>/.local/agent/<kind>/run-NNNN.json` | same |
| log-only switch | `AGFORGE_ZULIP_LOG_ONLY`, `AUTOLAB_ZULIP_LOG_ONLY`, `AGFRONT_ZULIP_LOG_ONLY` | `<AGENT>_ZULIP_LOG_ONLY` derived from the spec, plus `AGAG_ZULIP_LOG_ONLY` for all |

## Proposed pyagag module layout (Step 2)

```
agag/agent.py      AgentSpec, constants (SWEEP_ACK…), run_role, chat_environment,
                   listener_main, topic_filter, intro_main
agag/entrance.py   handle_entrance(spec, client, channel, topic), default guide text
agag/agent_config.py  v2: [roles.X] allowed_tools (string), v1 still accepted
agag/templates/    agag init's files (Step 3)
```
