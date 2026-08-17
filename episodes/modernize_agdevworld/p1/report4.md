# p1 step 4 — launchd, and the two reconcile conversations

Done. Front is a supervised always-on service on agstudio, and both proof
conversations ran against the live realm. **The phase's reconcile is met.**

## The service

- `devenv/launchd/com.agdev.agfront-zulip.plist.in`, modelled on
  `com.agdev.agautolab-zulip.plist.in`: `/bin/sh …/agfront/agent/zulip_listen.sh`,
  `RunAtLoad`, `KeepAlive`, `ThrottleInterval 10`, logging to
  `agfront/.local/out/zulip-listener.log`.
- `agfront/agent/zulip_listen.sh` — the sibling's two-line script,
  `exec uv run python -m agfront.zulip_listener`.
- Credentials: `agfront/.local/zulip.env`, a 0600 copy of
  `.local/zulip/front.env` — the same placement every other agent uses.
- Installed with `__PROJECTS_ROOT__` substituted into
  `~/Library/LaunchAgents/`, then `launchctl bootstrap gui/$(id -u) …`.
  `launchctl print` reports `state = running`, `last exit code = (never
  exited)`.

Before bootstrapping, the listener was run by hand with
`AGFRONT_ZULIP_LOG_ONLY=1` for twelve seconds: it registered as
`user_id=15 (front-bot@agstudio.local)` and took an event queue, which proves
the credentials and TLS path without costing an agent run.

`.local/devenv.md` gained an "agfront Zulip listener" section beside the
agforge and autolab ones: label, log, credentials, the two subscriptions,
the cost warning, and where `dispatch.md` is written.

## Conversation 1 — "please advance the work"

Posted from the Developer account into `#front` > `front-20260817-advance`:

> workを進めてください。

The listener matched it 21 seconds later, ran the `front` role, and the topic
now reads:

| # | Sender | Content |
|---|---|---|
| 362 | Developer | workを進めてください。 |
| 363 | Front | Message received. Please wait for the reply. |
| 365 | Front | `run-20260817-advance-work` トピックにディスパッチしました。返信はそのトピック内に届きます（このチャットには戻ってきません）。<br>dispatched to #general > run-20260817-advance-work; the reply will appear there |

Front answered in the Developer's language, as the guide asks, and the
handler's own line named the destination. The generation workspace holds
`chatlog.md` and the `dispatch.md` that produced it:

```
run-20260817-advance-work

Developer からの依頼です。「workを進めてください」とのことなので、進行中の開発作業を前に進めてください。
```

**One post, into `#general`.** The topic there:

| # | Sender | Content |
|---|---|---|
| 364 | Front | Developer からの依頼です。… |
| 366 | Autolab Agstudio | Message received. Please wait for the reply. |
| 370 | Autolab Agstudio | running "Test and Refine the Title Screen" in assetpipe1 — failed during work run: work run exited 2: agcode exited 2: turn_budget_exhausted: max_turns (20) exhausted |

The agstudio autolab listener acked within seconds (its log: `sweep matched
'general'/'run-20260817-advance-work'` → `run topic …`), chose an eligible
Work out of Plane, and executed it. **The route is what this step proves, and
it is proven end to end** — not merely "no work", but a real Work selected
and started from a message Front wrote.

The work itself then failed on agcode's 20-turn budget. That is autolab's own
concern on its own project (`assetpipe1`) and has nothing to do with the
dispatch path; it is recorded here because the plan asked what the reply was.

Exactly one node reacted, as predicted: agautolab1 has no Zulip listener.

## Conversation 2 — something Front cannot do

`#front` > `front-20260817-impossible`:

> 今日の東京の天気を教えて。あと冷蔵庫の牛乳を買ってきて。

Front's reply (message 369) says these are not development requests, lists
both, states plainly that it cannot dispatch them, suggests where the human
should actually look, and invites a development request instead.

**No `dispatch.md` was written** — the generation workspace holds only
`chatlog.md` — so nothing was posted to `#general`, and the reply carried no
dispatch line. The refusal path costs one run and no post, exactly as
designed.

## Evidence

`agfront/.local/evidence/` (git-ignored):

- `front-20260817-advance.json`, `front-20260817-impossible.json` — the two
  `#front` topics as the API returns them
- `general-run-20260817-advance-work.json` — the dispatched topic including
  autolab's ack and result
- `dispatch.md` — the command file Front wrote

Live workspaces remain under `agfront/.local/topics/front/…/1/front/`.

## Observations for later phases

- Front named its own topic `run-20260817-advance-work` from the date it saw
  in the conversation. Uniqueness held here; a repeated request on the same
  day could collide, and a collided topic is not a new button. Worth a
  suffix rule in the guide if it ever happens — Failure Farming applies, so
  it is not being pre-empted now.
- The dispatched message quoted the Developer verbatim and added context. The
  `run-` topic body is never read by autolab, so this text is for humans
  reading the topic later; that is fine, and worth knowing before anyone
  tries to make the body meaningful.
