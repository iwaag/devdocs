# agag_builder p2 plan — agautolab and agfront on the skeleton

Goal: the five modules every agent used to copy — instance, intro, role_run,
listener `main`/`topic_filter`, entrance — come from `agag.agent` /
`agag.entrance` in agautolab and agfront too, as they do in agforge since p1.
Nothing about what either agent *does* changes.

Success criteria:

1. agautolab and agfront no longer carry their own copies of those five
   concerns; each has a `listener.py`-shaped entry that builds an `AgentSpec`
   and calls `listener_main`, and `python -m <agent>.intro` is `intro_main`.
2. Their existing tests pass (or are deleted where they only tested the
   removed copies — say which).
3. Live check: one small `workplan-` in an existing `pj-` channel runs end to
   end through autolab; agfront is asked something that needs another agent
   (agecho is cheapest — its listener is still up from p1) and reports home.
4. All three agents pin the same pyagag revision; pj-agdev's submodule
   pointers move in one commit.

Decisions already made:

- **Scope is the five modules.** agautolab's `zulip_listener.py` (1254 lines)
  is mission logic with the listener shape at the bottom; only
  `topic_filter`, `main`, the log-only handlers and the entrance serving are
  replaced. `serve_run`, `prepare_run_surfaces`, `mirror_task_changes` etc.
  stay untouched. Same for agfront's `serve`/`handle_mention`.
- No backward compatibility. Wrappers kept in agforge "for old tests" in p1
  are the pattern to avoid here: delete tests of removed code rather than
  keep wrappers alive for them.
- agfront gets an instance name like everybody else (see Step 1).

Constraints: secrets in `.local/`; pyagag → push → `uv lock
--upgrade-package pyagag` in each consumer → push; cost is not a concern.

## Facts checked at planning time

- The skeleton already takes what these two agents need
  (`pyagag/src/agag/agent.py`):
  - `run_role(..., skip_permissions=, extra_args=)` — autolab's agcode
    `--max-turns`/deadline args and `skip_permissions=True` under
    claude_code.
  - `resolve_spec_role(profile_override=)` — autolab's per-project profile
    override (`load_project_roles`).
  - `run_role` takes `cwd` per call — autolab's `ROLE_WORKSPACES` is a lookup
    on the caller's side, not a skeleton concern.
  - `listener_main(spec, dispatch, entrance=, dm_handler=, on_mention=)` —
    agfront's mention route; forge's DM route proved `dm_handler` in p1.
  - `AgentSpec.extra_environment` — any PATH/env additions.
- `topic_filter` in autolab (`zulip_listener.py:1127`) is the same logic as
  the skeleton's: own channel entirely, `sweep_prefixes` elsewhere.
- **agfront has no `.local/instance.toml`** (autolab and forge do). The
  skeleton's `topic_filter` compares against `spec.instance_name()`, and
  `intro_main` posts under `intro-<instance>`. agfront already introduces
  itself in `#agents` (standardize p4) — check what name that post uses.
- `note_served` / `[selfnote][served]` lives in `agag.selfnote` and
  `agag.zulip`; agfront's `handle_mention` keeps calling it from its own
  handler. `write_agents_md` is called in agfront's `serve` before the run;
  that stays.
- Pins today: agforge lock → pyagag `7c29696`; pyagag main is `084c42a`
  (init-only commits since). autolab and agfront locks: check.
- Tests: `agautolab/tests/test_zulip_listener.py` is 1643 lines; agfront has
  a handful. A failure there after the swap most likely means a behavior
  difference in the skeleton, not in mission code — read the diff before
  "fixing" the test.
- Both agents have `.local/agents.local.toml` with the claude_code
  `command_glob` overlay, so p1's "claude not on PATH" failure will not
  recur.
- agecho listener: `nohup`, PID 65851 at planning time; restart with
  `agecho/service/listen.sh` if gone.

## Step 1 — agfront's instance name

Decide and do one of:

- (a) give agfront `.local/instance.toml` (`front-agstudio1` or whatever its
  `#agents` intro already says) and let the skeleton's own-channel sweep
  apply — its own channel may simply not exist, which is harmless; or
- (b) let `AgentSpec` express "no own channel" and sweep `front-` only.

(a) is recommended: it keeps one rule for every agent and costs one file.
Re-post the intro if the name changes.

## Step 2 — agautolab

- `instance.py`, `intro.py`, `agent_settings.py` → `AgentSpec` +
  `intro_main`. `agent_settings` is forge's `resolve_agforge_role` without
  the tool env; it maps onto `resolve_spec_role`.
- `role_run.py`: keep `ROLE_WORKSPACES`, `_agcode_args`, the project-profile
  lookup; the run itself becomes `run_role(spec, ..., cwd=ROLE_WORKSPACES.get(role, cwd),
  skip_permissions=..., extra_args=...)`. Move `ROLE_ALLOWED_TOOLS` into
  `agents.toml` `[roles.X] allowed_tools` (schema v2, as forge did).
- `zulip_listener.py`: replace `topic_filter`, `main`, `observe_*`,
  `entrance_prompt`/entrance serving with `listener_main(SPEC,
  dispatch={"workplan-": handle_topic, "workrun-": handle_workrun},
  entrance=None)` — `None` means the skeleton's `agag.entrance` with
  autolab's `agent/guides/entrance_front/guide.md` winning over the built-in
  default (that is how p1 resolved guides). Check that the `pj-`-channel
  sweep (topics in channels other than its own, matched by prefix) is what
  `sweep_prefixes` produces; autolab is the one agent whose plan topics live
  in somebody else's channel.
- `mission_done.py`, `project_init.py`, `project_archive.py` keep their own
  `ZULIP_ENV`/`PLANE_ENV` lookups or switch to `spec.zulip_env` /
  `agag.plane.credentials_path(root)` — either is fine; the latter removes
  the `ROOT.parent` assumption p1 found in forge.

Hint: do it in the order instance → intro → role_run → listener, running
`uv run pytest` after each; the listener swap is where tests will speak.

## Step 3 — agfront

- `role_run.py` (133) → `AgentSpec` + `run_role`; `ROLE_ALLOWED_TOOLS`
  (`"front": "Read,Glob,Grep,Bash(agentchat:*)"`) → `agents.toml`.
- `zulip_listener.py` (310): `main` → `listener_main(SPEC,
  dispatch={"front-": handle_topic}, on_mention=handle_mention)`. `serve`,
  `handle_mention`, `front_prompt`, `write_agents_md` call stay.
- Workspace helpers (`topic_workspace`, `generation_dir`, `guide`,
  `next_record_path`, `is_ack`) are the ones every agent copied; they exist
  in `agag.topics` / `agag.agent` already — use those and delete the copies.

## Step 4 — pins and pushes

pyagag (if touched) → push. Then in agforge, agautolab, agfront:
`uv lock --upgrade-package pyagag`, commit, push. Then one pj-agdev commit
moving all three submodule pointers. Restart the three launchd listeners
(`launchctl kickstart -k gui/$UID/com.agdev.<name>-zulip`).

## Step 5 — live check

- autolab: post a trivial `workplan-` (write one file) in an existing `pj-`
  channel; watch `.local/out/zulip-listener.log` through plan → run →
  report. Ask its own channel "list your plans" once to exercise the
  entrance through the skeleton.
- agfront: "ask agecho-agstudio1 to say hello and tell me what it said" —
  same exchange as p1 Step 4 but from the re-based Front.

Things that would show a skeleton difference rather than a mission bug:
a topic served twice (ack detection), a mention not bringing Front back
(`on_mention` wiring), the entrance answering with forge's vocabulary (guide
lookup path).

## Step 6 — record

`p2/report.md`: per-agent line counts before/after, test changes with
reasons, the live-check log excerpts, and what still differs between the
three `listener.py` files — if the answer is "only `dispatch` and the
guides", the skeleton is done and `agag init` output is the actual shape of
every agent.

## Out of scope

- autolab invoking `agag init` on request (next phase; needs only the
  `--yes` path and the printed checklist from p1).
- agecho under launchd; forge's unrun `F2-22` plan (delete, not resolve, if
  tidying — see standardize p10 TODO on revived Works).
- pruning standardize p10 TODOs that surface on the way; note them in the
  report instead.
