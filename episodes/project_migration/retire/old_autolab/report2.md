# Step 2 — delete the old implementation and its callers

## agautolab

Commits `688e404` (deletion), `8440d65` and `1e90c27` (README, counted in step 3).

- **Deleted** (59 files, −13.6k lines):
  - the listener and routing (`listener.py`, `zulip_listener.py`);
  - the mission/task records (`anchor.py`, `worklog.py`), mission working copies (`missionspace.py`) and the close-out and correction tools (`mission_done.py`, `correct_state.py`);
  - `argue.py`, `study_setup.py`, `project_init.py`, `project_archive.py`, `project_settings.py`, `role_run.py`, `trial.py`, `intro.py`, and the `autolab` CLI (`cli.py`, `[project.scripts]`);
  - `agent/gateway.py` and `agent/zulip_listen.sh`;
  - the old guides (`entrance_front`, `workplan_superdirector`, `workrun_supercoder`, `bmining_director`, `argue`) and `agent/project_pattern.md`;
  - the `agent/front` and `agent/mediator` workspace members, `init_project.py`, `params/intro.md` and `instance.example.toml`;
  - 21 test modules and their fixtures.
- **`instance.py`** now holds only what the executor and transports use: `AGAUTOLAB_ROOT`, `PROVISIONER_ENV`, and `SPEC = AgentSpec("autolab", root)`. It sweeps no prefixes, publishes no execution-option menu and adds no environment. No managed workflow uses comfynotify or the admin env.
- **`agents.toml`** keeps every model and profile; workflows select `sonnet` and `luna`, and a held run can resume on another profile. Of the roles, only `wfrun` remains. `service_http` was dropped from `provides`.
- **Local files**: the overlay lost its `roles.coding`/trial lines, and `.local/instance.toml` was moved to `.local/retired-20261019/`.
- **Tests**: 30 passed, 1 skipped, 13 failed in `test_wfnotify.py`/`test_wfconversation.py`. The same 13 fail on the untouched previous HEAD against the current WFE: event-number expectations such as `#4:question` vs `#5:question` predate this change. They were not fixed here.
- **Mistake**: removing the leftover `agent/front/` directory also deleted its ignored `.local/topics/` chatlogs, which were 2026-08-13 trial mission records. They were untracked and are not recoverable from Git.

## Deployment definitions

- **pj-agdev** `9576f7e`:
  - deleted the `com.agdev.agautolab-zulip` and `com.agdev.agautolab-gateway` launchd templates, with a note in `devenv/launchd/README.md`;
  - removed the retired `autolab-agstudio1` and `agforge-agstudio1` roots from agentroom's `AGENTROOM_AGENT_ROOTS`;
  - removed the autolab health-interface paragraph and the mission wording from Observer's introduction (Observer: 114 tests passed).
- **ansible_agdev** `f86e590`:
  - deleted the `autolab_node` role, `setup_autolab_node.yml` and both `autolab_node` profile entries (variables and reconciliation with its `/healthz` check);
  - rewrote the README section;
  - moved the ignored static `inventories/agautolab.yml` to local notes.
- **nctl** `d5659a1`: the profile test no longer lists `autolab_node`, and the planner's manual-placement test uses `agag_agent`. 1358 tests passed.

## agdevworld (subagent)

Commit `e5ce398` (pushed).

- **Deleted**:
  - agentroom `autolab.py`, `projectroom.py`, `projecttalk.py`, and `forge.py` (forge is retired), with their tests;
  - the `/projects*` and `/work/<anchor>*` routes;
  - the frontend Project Room (`projectRoom.ts`, `projectState.ts`, `projectDemo.ts`, `ProjectRoomScene.ts`) and its entries.
- **Edited**:
  - `closing.py`, `close.py` and `ops.py`: the mission/task kinds, `work-` channel archiving, `_accept_mission` (its `agag.acceptance` use) and forge branches are gone. The Front, desk, routine-run and argue completion core is kept.
  - `room.py`: the `pj-`/`work-` boards are gone from `/work`.
  - The completion, close and popup wording; demo fixtures; tests; `agentroom/README.md`; and `README_DEV.md`.
- **Results**: agentroom 268 passed; `tsc --noEmit` clean; `npm run build` succeeds. WFE is untouched; its `autolab` is the executor identity.

## pyagag, agfront, archsage (subagent)

Commits pyagag `03a2a32`, agfront `a6a0370`, archsage `4dac08d` (all pushed). Tests: pyagag 1174, agfront 185 and archsage 41 passed, with no failures.

- **pyagag**:
  - Deleted:
    - `agag.acceptance`;
    - `agentchat accept`/`reserve`, with their claims-reader acts and tags;
    - the writer `selfnote.replaces_note` and the lookup `zulip.inherited_rootchat`;
    - the `agag.fixture` agautolab driver, its probes (`planner-other-project`, `entrance-plans`, `delegate-answer`, `delegate-decision`) and `AUTOLAB_DOORS`/`DOORS`;
    - the tests for all of these.
  - `agag.project` lost the whole autolab setup handshake (`workplan-setup-`, setup blocks, `established` lines, the workspace role, `--about`). `open` now creates the `pj-` channel and posts the goal or plan. Its help says repositories and workflows belong to a WFE managed project (`wfe create`).
  - `agent.py` and `dispositions.py` text was reworded.
- **agfront**:
  - Argue `verify_outcome` checks only the channel and its goal/plan topic. Front has no `wfe` grant, so no WFE check was added.
  - The guides (`shared/work.md`, `requests.md`, `board.md`, `argue/guide.md`, `routine_run/guide.md`) now describe development work as a WFE workflow request (`wf-…` channels, `request-…` topics, `run-…` progress). `workplan-`/`workrun-` topics are described as history. Accept/reserve is gone.
  - Deleted `tests/test_replacement.py`, plus the replacement and `delegate-decision` tests.
  - Stale comments were fixed.
- **archsage**:
  - The guide no longer asks autolab for workspaces or missions. Its study-routine section and the `agroutine` grant were removed: they existed only to ask autolab for missions.
  - The docstring, the intro line and the listener test fixture were updated.
- **Kept, with the reason**:
  - `agag.document`: cagent's `change_record.py` uses it.
  - `selfnote.parse_replaces`, `replaced_anchor` and `REPLACES_TAG`: `agag.trace` reads historical conversations with them.
  - The mission and acceptance readers in `agag.trace`, `agag.progress` and `holds`: they read history the panel can still show. This was out of scope for this round.
  - `start_note`/`parent_rootchat`, and `AgentSpec` prefixes: live users.
  - The generic scripted responder in `agag.fixture`: now unused, a follow-up candidate.

## Search result

A search of active source, guides and deployment definitions finds no live caller of the removed modules, routes, CLI commands or topic prefixes. The remaining matches are:
- history: episode reports, retired agforge, recorded fixture JSON;
- generic fixture topic names in tests;
- WFE's executor identity `autolab`;
- the historical trace/progress readers listed above.
