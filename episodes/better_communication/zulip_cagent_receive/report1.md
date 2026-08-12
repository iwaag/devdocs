# zulip_cagent_receive — Step 1 report: `agag.zulip` + one round trip

Date: 2026-08-12. Status: **complete**. The Zulip chat entrance is now shared
code in `pyagag`, agforge runs on it unregressed, and the Cagent bot has
completed one manual receive/reply round trip.

## What moved

`agforge/src/agforge/zulip.py` and the generic half of
`agforge/src/agforge/zulip_listener.py` are now `agag.zulip` in `pyagag`
(commit `36723a7`, pushed by the developer on request). The module is
stdlib-only and carries:

- `ZulipClient` — HTTP Basic bot client, `from_env(path)` over a `KEY=value`
  credentials file, self-signed TLS accepted when no `ZULIP_CA_BUNDLE` is
  named; `whoami`, `register`, `poll`, `deregister`, `dm_history`, `send_dm`.
- `dm_partners`, `is_dm_for_us` — the numeric-user-id helpers the self-loop
  guard rests on.
- `serve(client, handler, log=log)` — the long-poll loop: register,
  re-register on `QueueExpired`, sleep-and-retry on any other `ZulipError`,
  skip non-DM and self-sent messages, and never let a handler exception end
  the loop.

The one agforge-specific thing removed on the way is the default credentials
path: `from_env` now takes the path, because each agent keeps its own env file.

The three bugs the plan warned about are pinned by tests in `pyagag`
(`tests/test_zulip.py`, 11 cases, fake client, no network): `whoami` inside
the retry loop, `ZulipError` on an unwrapped `RemoteDisconnected` leading to
re-registration, and every identity comparison keyed on numeric ids. The
loop's swallow-handler-errors behaviour is pinned too — it is the reason the
throwaway round-trip script below had to stop the loop with a `BaseException`.

What stays in agforge is the credentials path, the passive log-only handler,
and the chat route (`zulip_chat.py`, unchanged apart from its import).
`agforge/tests/test_zulip.py` lost the assertions that now live upstream and
keeps the ones about transcript assembly and the `reply` field.

## Forge is unregressed

Two DM round trips with the Forge bot, sent from the Omni Agent bot account
(the developer account was left alone; the realm treats every sender the same
and the listener keys on numeric ids):

| # | Setup | Result |
|---|---|---|
| 1 | manual process, local `pyagag` on `PYTHONPATH`, launchd job booted out | ack + answer, 8.4 s, `cost_usd` 0.289 |
| 2 | launchd job restored, `pyagag` installed from GitHub `main` | ack + answer, 8.0 s, `cost_usd` 0.105 |

Both runs report `role=generator profile=sonnet harness=claude_code
provider=anthropic model=anthropic/claude-sonnet-5`, i.e. the run record still
names its backend. `uv run pytest -q` in agforge: 68 passed. The second run is
the one that matters — it proves the published package, not a path override.

The listener is back under `com.agdev.agforge-zulip` and re-registered its
queue on start.

## cagent's first dependency

`cagent/pyproject.toml` gains `dependencies = ["pyagag"]` and a
`[tool.uv.sources]` git pin on `main`, matching how agforge and agautolab
consume it. Everything else in `cagent_api` stays stdlib-only.

## The Cagent bot round trip

A throwaway script (scratch, not committed) registered an event queue with the
Cagent bot's own credentials from the ignored
`pj-clusterintent/.local/zulip/cagent.env`, received one DM, replied, and
exited:

```text
listening as user_id=14 (cagent-bot@...)
registered event queue 83d2… (last_event_id=-1)
received DM #44 from 9 ([9]): 'Hello Cagent — Omni Agent here, …'
replied with message #45
round trip complete
```

Receive-to-reply was under a second. No agent ran: the window entrance is
Step 3, and this step only proves credentials, client, and realm mechanics.

## Notes

- Realm-local numeric ids in play: 8 developer, 9 Omni Agent, 13 Forge bot,
  14 Cagent bot. No secrets, tokens, or host/IP values are in this file.
- Deus Ex Machina: the Omni Agent did this move, not an in-system agent —
  handoff candidate, carried into `report.md`.
