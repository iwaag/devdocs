# agag_builder p1 plan — the thin skin `agag init` generates

Goal: bring up a new agag-compliant agent with **one command + a few
questions**, post its introduction to `#agents`, and have agfront talk to it
knowing nothing but that introduction — and get a reply.

Success criteria:

1. `agag init <agent>` asks interactively for instance name, topic prefixes
   and roles, and generates a working project.
2. The generated project is **thin**: agent-specific code is the instance
   name, role definitions, guides and `params/intro.md`. Listener / intro /
   selfnote / entrance / role_run live in pyagag.
3. A generated dummy agent (`agecho` or similar) is started with
   `listen.sh`, posts its intro to `#agents`; agfront is asked "say hello to
   `agecho-agstudio1`" and the reply lands in a `front-*` topic.
4. agforge is moved onto the new pyagag skeleton and keeps working (other
   existing agents are not touched this phase; no backward compatibility —
   if agforge breaks, fix it).

Decisions already made (braindump + discussion):

- **Library over template.** The value of `agag init` is measured by how
  small its output is. A convention change (like p8's selfnotes) must reach
  every agent with one pyagag push.
- The new command is `agag` (added to pyagag's `[project.scripts]`).
  `agentchat` stays as it is.
- First version deploys only as far as a local `listen.sh`. No launchd plist
  / ansible generation (`pj-agdev/devenv/launchd/*.plist.in` is a 5-minute
  copy by hand).
- Creating the Zulip bot account, the Plane account and editing the channel
  description are human work. `agag init` **prints them as a checklist at
  the end**, nothing more. An agent cannot edit its channel description
  (HTTP 400, standardize p10 TODO), so do not force automation.
- Plane registration follows the "plan topics register a Work" convention,
  but **the first dummy agent need not use Plane**. Plane is opt-in in the
  generated project (`agents.toml` or a role's `requires`).

Constraints: secrets in `.local/`. After changing pyagag: push →
`uv lock --upgrade-package pyagag` in consumers → push (localrule.md).
Cost is not a concern.

## Facts checked at planning time

- pyagag (`/Users/eiji/projects/pyagag`): the only CLI is `agentchat`
  (`agag.chat:main`; `send/read/topics/channels/resolve`). No scaffold.
  Already shared: `agag.zulip` (`ZulipClient`, `sweep_serve`, `serve`),
  `agag.topics.serve_topic` (ack → handler → always reply; p10's common
  seam), `agag.intro` (`post_intro`, `write_agents_md`), `agag.selfnote`,
  `agag.harness.run_harness`, `agag.agent_config` (`ag.agent-config.v1`),
  `agag.plane`, `agag.instance.instance_name`, `agag.status`.
- **Copy-pasted across agents** (candidates to lift into pyagag):
  - `instance.py` (forge 40 / autolab 45 lines; docstring is the only diff)
  - `intro.py` (27 / 34 lines; same)
  - `role_run.py` (forge 205 / autolab 196 / front 133): resolve
    `agents.toml` → put `AGENTCHAT_ZULIP_ENV` / `AGENTCHAT_HOME` / `agentchat`
    on PATH → `run_harness`. The `ROLE_ALLOWED_TOOLS` table (per-role
    `--allowedTools`) is here too. **A role without a grant makes claude_code
    sit on a permission prompt until timeout** (note in agfront/role_run.py).
    Moving that table into agents.toml makes the skeleton easier.
  - `zulip_listener.py` (forge 119): `topic_filter` (whole own channel +
    prefixes), `dispatch` (prefix → handler, otherwise → `entrance_topic`),
    `main` (`sweep_serve`). autolab's 1254-line version is mixed with mission
    logic; do not use it as the reference.
  - `entrance_topic.py` (130) + `agent/guides/entrance_front/guide.md`
    (8 lines): the same "front run reading its own board" for every agent.
    Only `assetplan-/assetrun-` in the guide is specific.
  - `anchor.py` (`[selfnote][work]`), label/project choice in `plane.py`.
- Path conventions: `<root>/.local/zulip.env` (bot credentials),
  `pj-agdev/.local/plane-credentials.env` (shared; forge looks it up as
  `ROOT.parent/.local/…`, which fails outside pj-agdev. The skeleton should
  resolve it via an `AGAG_PLANE_ENV` env var or
  `<root>/.local/plane-credentials.env`, and stop using parent-relative
  paths), `<root>/.local/instance.toml`, `<root>/.local/agents.local.toml`.
- pyagag is consumed as a git dependency:
  `pyagag = { git = "https://github.com/iwaag/pyagag.git", branch = "main" }`.
- `agautolab/init_project.py` / `project_init.py` are **autolab creating a
  work project**, not an agent scaffold. Still worth reading as an example of
  interactive prompts + Zulip channel creation + Plane project creation
  (`client.create_channel`, `agag.plane.create_project`).
- `devdocs/episodes/agent_standardize/p10/report4.md §8` lists open TODOs.
  Not for this phase, but "an agent cannot edit its own channel description"
  feeds straight into init's checklist.

## Step 1 — the lifting map (output: `p1/skeleton_map.md`)

Lay agforge / agautolab / agfront `src/` side by side and sort every
module/function into **lift to pyagag / keep in the template / agent's
own**. The facts above are the draft. Use forge as the base line; autolab is
only diffed against it.

Hint: `diff <(sed s/agforge/X/g …) <(sed s/agautolab/X/g …)` already showed
instance.py and intro.py differ only in docstrings. Read all three role_run.

## Step 2 — the skeleton in pyagag (`agag.skeleton`, `agag.agent`, naming is free)

Move the "lift" column. Target API, roughly:

- `AgentSpec` (or toml): `agent` (short name), `plan_prefix`, `run_prefix`,
  roles, `root: Path`. `instance_name()` just calls
  `agag.instance.instance_name(root/.local/instance.toml, fallback=…)`.
- `listener_main(spec, dispatch)`: generalizes forge's `zulip_listener.main`
  + `topic_filter`. Prefixes with no `dispatch` entry fall through to
  `entrance`.
- `run_role(spec, role, prompt, workspace, …)`: the common part of role_run.
  `ROLE_ALLOWED_TOOLS` moves to `agents.toml` `[roles.X] allowed_tools = "…"`
  read by `agag.agent_config` (update `docs/agent-config-v1.md`; schema may
  become v2).
- `entrance.handle(spec, client, channel, topic)`: forge's `entrance_topic`
  moved; guide body is a pyagag built-in default with
  `{plan_prefix}/{run_prefix}` substitution, and the agent's
  `agent/guides/entrance_front/guide.md` wins when present.
- `intro.main(spec)`: the body of `python -m <agent>.intro`.

Free choices: module split, names, dataclass vs toml. **No prohibitions.**
Existing tests (pyagag, agforge's 19) are fixed or deleted when they break.

Check: replace forge's `zulip_listener.py` / `role_run.py` /
`entrance_topic.py` / `intro.py` / `instance.py` with skeleton calls, run
`uv run pytest`, start with `AGFORGE_ZULIP_LOG_ONLY=1`, then one real
`assetplan-` on live Zulip. Commit → push (pyagag, agforge, lock update).

## Step 3 — `agag init`

`pyagag/src/agag/init.py`, `[project.scripts] agag = "agag.cli:main"`
(`agag init` alone is enough).

Questions, minimal (with defaults; `--yes` takes every default):

1. agent short name (argument)
2. instance name (default `<agent>-<hostname>1`, written to
   `.local/instance.toml`)
3. plan / run prefixes (default `<agent>plan-` / `<agent>run-`)
4. roles (default `front`), profile (default `sonnet`)
5. output directory (default `./<agent>` under the current directory;
   `git init` only, no remote)

Generated files (a guide; shrink it):

```
<agent>/
  pyproject.toml            # pyagag git dependency, scripts
  agents.toml               # ag.agent-config + allowed_tools
  instance.example.toml
  params/intro.md           # {instance} substitution, prefixes filled in
  agent/guides/<plan>_front/guide.md   # ~8-line stub; "what it does" left as TODO
  src/<agent>/__init__.py
  src/<agent>/listener.py   # 10–20 lines: define spec, call agag's listener_main
  service/listen.sh         # near copy of forge's
  .gitignore                # .local/
  .local/instance.toml      # from the answers
```

Checklist printed at the end (human work):

- Zulip: create the bot → `.local/zulip.env` (`ZULIP_URL/EMAIL/API_KEY`).
  Creating the `<instance>` channel may be automated with
  `ZulipClient.create_channel`; subscribing to `#agents` and the description
  are human.
- Plane: if used, `pj-agdev/.local/plane-credentials.env` already exists.
  Adding an account is human.
- `uv sync && uv run python -m <agent>.intro` → `service/listen.sh`.
- To keep it running, copy a plist.in from `pj-agdev/devenv/launchd/`.

Hint: no template engine needed; `string.Template` or f-strings. Ship the
file contents inside the pyagag package as `templates/`
(`importlib.resources`).

## Step 4 — run it through with a dummy agent

Location: workspace root `/Users/eiji/projects/agecho/` (no remote). Do not
put throwaways under `pj-agdev/`, where each agent is a submodule.

`agag init agecho` → do the checklist → start `listen.sh` →
`python -m agecho.intro` → ask agfront "say hello to agecho-agstudio1 and
tell me what it replied".

Expected: agfront, from `tools/agents.md` (the intro harvest) alone, posts
in a plain topic of the `agecho-agstudio1` channel; agecho's entrance (a
front run) answers; agfront reports in `front-*`. If a conversation happens
with zero agent-specific code, the phase is a success.

Fixing what fails is the point (Failure Farming). Likely snags:
- agfront's `tools/agents.md` is harvested at run start; call Front after
  the intro is posted.
- The bot must be subscribed to `#agents` to post the intro.
- The entrance should reply with the built-in default guide. If not,
  suspect a missing `allowed_tools` grant (permission prompt until timeout).
- `[selfnote][rootchat]` is written by agfront's `agentchat send`; agecho's
  side should be handled by `serve_topic`'s `reply_to` automatically.

## Step 5 — record

- `p1/report.md`: line counts of the generated project, Step 4 Zulip log
  excerpt, failures and fixes, and what could not be lifted and stayed in
  agforge (seeds for the next phase).
- One line in `devdocs/README_DEV.md` In-System Agents: new agents come from
  `agag init`.
- agecho may stay (it becomes the minimal fixture for later standardize
  experiments). If removed: archive the Zulip channel, leave Plane alone.

## Out of scope (later)

- autolab calling `agag init` on "make me a new agag agent" (Step 3's
  `--yes` and the printed checklist are the groundwork).
- Moving agautolab / agfront onto the skeleton.
- Generating launchd / ansible.
