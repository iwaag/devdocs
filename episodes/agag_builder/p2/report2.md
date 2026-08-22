# agag_builder p2 — step 2: agautolab on the skeleton

agautolab `96f09ff` (+ README `HEAD`), pyagag `5da31c9`, `a8fa481`.
`uv run pytest`: 168 passed (was 179; see "tests").

## pyagag additions (two small pushes)

- `AgentSpec.extra_prefixes` — autolab sweeps three prefixes, not two
  (`bmining-`); `sweep_prefixes` now includes them, the default guide does
  not name them.
- `run_role(extra_meta=)` — autolab stamps `project` into the returned run
  record (it was always filtered out of the file by `write_run_record`;
  only the in-memory dict carried it, same as before).
- `run_role(agent=)` — a caller that has to look at the resolution before
  the run (autolab decides agcode's budget and claude_code's bypass from
  `agent.harness`) hands the resolved agent over instead of resolving twice.

## agautolab

| file | before | after | what |
|---|---|---|---|
| `instance.py` | 45 | 47 | `SPEC = AgentSpec("autolab", ROOT, plan_prefix="workplan-", run_prefix="workrun-", extra_prefixes=("bmining-",))`; the prefix constants moved here from `zulip_listener` |
| `intro.py` | 34 | 14 | `intro_main(SPEC)` |
| `agent_settings.py` | 25 | — | deleted; `resolve_spec_role` is it. `project_init`/`project_settings`/`agent/gateway.py` took `PROJECT_ROOT`/config paths from it → `instance.AGAUTOLAB_ROOT`, `SPEC.agents_config` |
| `role_run.py` | 196 | 118 | `ROLE_ALLOWED_TOOLS`/`WORKING_ALLOWED_TOOLS` (≈60 lines) → `agents.toml` v2; `tool_environment` → skeleton. Keeps `ROLE_WORKSPACES`, `agcode_args`, the project-profile lookup, `skip_permissions` under claude_code |
| `listener.py` | — | 38 | the entry: `listener_main(SPEC, {workplan-, workrun-, bmining-}, on_mention=handle_mention)` |
| `zulip_listener.py` | 1254 | 1115 | lost `topic_filter`, `main`, `observe_topic`, `entrance_prompt`/`serve_entrance`/`handle_entrance`, `dispatch`, the four workspace wrappers, and its own ACK/EMPTY/NO_CLOSING constants (now the skeleton's) |
| `agents.toml` | v1 | v2 | every role carries `allowed_tools`; `summarizer` = `Read,Glob,Grep` |
| `agent/zulip_listen.sh` | | | `python -m agautolab.listener` |

Total of the five concerns: 1554 → 1332 lines, of which 1115 are mission
logic that did not move.

## Skeleton differences found (the listener swap "spoke")

1. **Own-channel prefixed topics.** The skeleton routes by prefix before
   checking the own channel (right for forge, whose `assetplan-` requests
   live in its own channel). autolab's `dispatch` did the opposite: every
   topic in `#autolab-agstudio1` went to the entrance, so a `workplan-`
   name there was a question, not work. Kept autolab's rule inside its
   handlers (`at_the_entrance` → `agag.entrance.handle_entrance`) rather
   than changing the skeleton for one agent. The `pj-`-only guard for
   `workplan-`/`bmining-` likewise moved from `dispatch` into the handlers
   (`in_project_channel`).
2. **Entrance timeout**: autolab's was 900 s, the skeleton's is 600 s.
   Accepted: the entrance reads chat only.
3. **DM thread**: autolab's `main` had none; the skeleton always starts a
   passive one (log only). Harmless.
4. `format_chatlog(..., drop=is_ack)` — the skeleton drops acks from the
   entrance chatlog; autolab's `serve_entrance` did not. Better, not worse.

## Tests

- deleted `test_instance.py` (3 tests of `agag.instance` through a wrapper).
- `test_intro.py`: 2 → 2, now against `intro_main(SPEC)`; plus a SPEC check.
- `test_role_run.py`: 10 → 5. Dropped the `tool_environment` tests
  (skeleton's `chat_environment`, tested in pyagag) and the
  `ROLE_ALLOWED_TOOLS` table tests; `test_every_role_carries_its_grant_in_agents_toml`
  checks the v2 grants instead.
- `test_zulip_listener.py`: 1643 → 1585 lines. Deleted the `topic_filter`,
  `entrance_prompt`, `serve_entrance`, failed-entrance tests (skeleton);
  `test_dispatch_…` → `test_the_routes_are_the_three_prefixes_…` (what
  `listener.main` hands the skeleton + the handlers' own guards);
  `test_the_own_channel_is_answered_and_never_executes` rewritten for the
  handler-side rule; `test_handle_topic_reports_a_channel_that_is_not_a_project`
  → `…ignores_…` (the guard now precedes the serving, so nothing is posted
  into a stray topic — what `dispatch` already did for the live path).
  Added `test_the_entrance_answers_with_autolab_s_own_guide`.
- `test_gateway_window.py` broke on the `agent_settings` import; fixed in
  `agent/gateway.py`.

## Left as it was

- `project_init.PLANE_ENV = ROOT.parent/.local/plane-credentials.env`
  (and `mission_done`'s use of it). Switching to `credentials_path(root)`
  needs a symlink in every autolab checkout's `.local/` — including the
  agautolab1 VM's — so it is not done blind here. One symlink + one line
  when that node is next touched.
- The standardize p10 TODOs were not pruned (out of scope).
