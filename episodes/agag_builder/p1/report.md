# agag_builder p1 — report

Goal: a new agag agent in one command + a few questions, introduced on
`#agents`, reachable by agfront from that introduction alone. **Done**; all
four success criteria met. Steps: [report1](report1.md) map,
[report2](report2.md) skeleton + agforge, [report3](report3.md) `agag init`,
[report4](report4.md) agecho.

## Commits

| repo | commits |
|---|---|
| pyagag | `7c29696` skeleton (`agag.agent`, `agag.entrance`, config v2, `plane.credentials_path`), `cecd21b` `agag init`, `2eabf40` wheel fix, `084c42a` checklist: claude overlay |
| agforge | `cb37dc7` on the skeleton (pin → 7c29696) |
| pj-agdev | `5fcdc8a` submodule pointer |
| devdocs | step reports, `README_DEV.md` In-System Agents line |
| agecho | `/Users/eiji/projects/agecho`, local only, 2 commits |

## How thin the generated agent is

111 tracked lines for a working agent, 34 of them code:

```
src/agecho/listener.py   26   AgentSpec("agecho", ROOT, plan_prefix=…, run_prefix=…); listener_main(SPEC, {})
src/agecho/intro.py       8
agents.toml              21   ag.agent-config.v2, sonnet/stub, roles.front with allowed_tools
pyproject.toml           22   pyagag from GitHub
params/intro.md          12   the contract ({instance} filled at post time)
instance.example.toml     8 · guide stub 5 · listen.sh 6 · .gitignore 3
```

Listener, entrance, role run, intro, instance name, ack/turn-taking, selfnote
handling: all in pyagag. A convention change is one pyagag push plus
`uv lock --upgrade-package pyagag` on each consumer.

agforge's same five modules went 521 → 297 lines, and 128 of those are its
own `tool_environment` plus thin wrappers kept for existing callers/tests.

## Step 4 Zulip log (excerpt)

```
06:15:54 front   serving 'agents'/'front-greet-agecho'
06:16:07 agecho  serving 'agecho-agstudio1'/'hello-from-front' → entrance topic
06:16:16 agecho  @**Front** Hi Front — hello back from agecho-agstudio1! …
06:16:16 front   mention in 'agecho-agstudio1'/'hello-from-front' serves agents/front-greet-agecho
06:16:26 front   @**Developer** agecho replied. … "Hi Front — hello back …" Task complete.
```

Front read `tools/agents.md`, chose a plain topic in `#agecho-agstudio1`;
agecho answered with the **built-in default entrance guide** and default
grant; the rootchat selfnote brought Front back; Front reported at home.
No agecho-specific code was involved.

## Failures and fixes

1. **Wheel build**: `force-include` of `agag/templates` duplicated files
   hatch already packaged → `uvx` install failed. Dropped the table.
2. **`claude` not on PATH**: a fresh agent needs
   `.local/agents.local.toml` with `[local.harness.claude_code] command_glob`.
   Added as checklist step 4. (Candidate for later: `agag init` could copy
   the overlay from a sibling agent when asked.)
3. **Wording**: my `assetplan-` test post said "register it", which forge
   read as a second deliverable. Not a skeleton fault; noted because an
   agent's own vocabulary leaks into how requests are read.
4. forge's Plane lookup was `ROOT.parent/.local/…`; now
   `credentials_path(root)` and a symlink in `agforge/.local/`. Verified by
   forge registering `F2-22` in Plane through the new path.

## Still in agforge (seeds for the next phase)

- `zulip_chat.py` DM charter route (`react`) — passed as `dm_handler`.
- `tool_environment` (ACE Studio CLI, `.local/bin`, `scripts/` on PATH) — `SPEC.extra_environment`.
- `anchor.py` (`[selfnote][work]`), `plane.py` label/project choice, `assetplan_`/`assetrun_topic`.
- `zulip_listener.topic_filter`/`dispatch` and `role_run.chat_environment`
  wrappers exist only for old tests.
- agautolab (agcode `extra_args`, `skip_permissions`, per-project profiles,
  `ROLE_WORKSPACES`) and agfront (`on_mention` + `note_served`,
  `write_agents_md`) are untouched; `run_role`/`listener_main` already take
  the arguments they need.

## Human-side notes

- agecho's bot was created over the API as the Developer (owner); `#agecho-agstudio1`
  and the `#agents` subscription likewise. Creds: `agecho/.local/zulip.env`,
  copy at `pj-agdev/.local/zulip/agecho.env`.
- agecho's listener is a `nohup` process, not launchd; it ends with this
  session. agecho stays as the standardize fixture (plan, Step 5).
- The `assetplan-skeleton-check-icon` plan (F2-22) in forge was registered
  but never run; resolve or run it at leisure.
