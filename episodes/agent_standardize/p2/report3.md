# p2 step 3 — the Front run reworked

AI-generated (Omni Agent, 2026-08-20).

## The route agfront no longer owns

Removed from `zulip_listener.py`: `handle_create`, `create_topic_name`,
`CREATE_FILE`, `CREATE_TOPIC_PREFIX`, `OUTBOUND_CHANNEL`, and the
`topic_write` import that carried the `#general` post. `serve()` is now three
steps — write `chatlog.md`, write `tools/agents.md`, run — and the run's own
answer is the whole reply.

That p1 route was a shackle around an agent that could not read the board:
Front described an asset in `create.md`, and *the handler* decided the
channel and derived the topic, because "Front never names the channel or the
topic" was the safety. p2 deliberately retires that safety. The replacement
is the intro: Front finds the entrance by reading, which is the capability
this phase exists to prove, and a handler that still chose the channel would
have made the proof impossible.

## What the run is given instead

`role_run.py` gained the tool handover agfront never had, in agforge's shape:

- `tool_environment()` prepends the directory holding the interpreter that
  runs the listener — `.venv/bin` in this uv project, verified live: the
  launchd job's process is `agfront/.venv/bin/python3 -m
  agfront.zulip_listener` — so `agentchat` resolves by its bare name with no
  deployment path written down anywhere.
- It sets `AGENTCHAT_ZULIP_ENV` to `agfront/.local/zulip.env`. Per the
  phase constraint the identity travels as a **path**, never as a value; no
  `ZULIP_*` key is inlined into the run environment, and a test pins that.
  The variable name is spelled once per side and both sides are pinned
  (`agag.chat.ENV_VARIABLE` / `role_run.AGENTCHAT_ENV_VARIABLE`).
- `resolve_agfront_role` merges the handover into the resolved agent, so
  every path to a run carries it.
- The grant became `Read,Glob,Grep,Bash(agentchat:*)`. `Write` is gone with
  the command file it existed for; the shell is only the one command that
  reaches another agent, not a general shell.

Front speaks as **itself**, not as the Developer. A post it makes is
attributable to the Front bot, which is what makes the guide's
ask-permission-first flow mean something.

## Multi-turn needed no code

The guide asks permission, then acts on the next turn. `serve_topic` already
re-checks the topic after every run, so the Developer's answer is simply the
next serving — a new generation, a new chatlog, a fresh board.

## Verification

`uv run pytest` in agfront: **31 passed**. The suite was reworked, not
patched:

- `tests/test_zulip_listener.py` — the run's answer is the reply and the only
  channel posted to is the front topic's own; the prompt and chatlog are the
  run's whole input; a run that writes nothing is the normal case (the
  workspace holds only `chatlog.md` and `tools/`); the harvest tests from
  step 2; a failure names its step; each serving gets its own generation.
- `tests/test_role_run.py` — the grant is reading plus `agentchat` and no
  other `Bash(`; the identity is a path; `agentchat` is reachable by bare
  name; a missing bin directory leaves PATH alone; a resolved role carries
  the handover.
- The stub-run test still runs the whole route with no `run_front`
  monkeypatch — real config pair, real `run_harness`, real workspace, real
  harvest — and now checks what the p2 run actually depends on: the stub sees
  its prompt on stdin, `tools/agents.md` with the harvested intro,
  `$AGENTCHAT_ZULIP_ENV` pointing at the front bot's file, and
  `command -v agentchat` resolving. Only the harness process and Zulip are
  stubs.
- `test_agfront_knows_no_other_agent_s_channel` makes success criterion 3 a
  test rather than only a grep: `agforge-agstudio1` appears in no file under
  `src/` and no guide.

The implementation was checked against the already-rewritten guide, not the
other way round.

## Reload

```
$ launchctl kickstart -k gui/$(id -u)/com.agdev.agfront-zulip
2026-08-20T14:18:33Z agfront zulip listener starting (pull sweep, prefix 'front-')
2026-08-20T14:18:33Z sweeping as user_id=15 (front-bot@agstudio.local)
2026-08-20T14:18:33Z full sweep: 0 awaiting, 13 calls spent, 989 left in the window
```

## Delivery

agfront `f2bcaa8`, pushed to GitHub. The listener on this Mac runs it.

Every `front-*` post is one paid run, which is intended.
