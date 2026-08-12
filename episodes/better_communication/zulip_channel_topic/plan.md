# zulip_channel_topic — plan

Date: 2026-08-12. Follows `zulip_receive` (agforge DM entrance) and
`zulip_cagent_receive`. Braindump: agent-to-agent DMs are invisible to the
Developer; move the agforge request conversation into channels/topics that are
browsable and searchable.

Developer's supplements (2026-08-12):

- Lifecycle end is **resolving the topic only**; per-request channels are NOT
  archived, they stay.
- Do not skimp on cost: real agent runs are fine at every step.
- This is an experimental, non-public environment: minimum prohibitions,
  maximum implementer discretion. Destructive phase — no backward
  compatibility required.

## Target workflow

1. Standing **public** channel `#FreeForge` — the interest area for one-off
   agforge requests. Participants: forge bot (user id 13), devworld-assistant
   bot, Developer (user id 8).
2. Per request, the **devworld-assistant account creates a public channel**
   named `create-YYYYMMDD-HHMMSS-<shortid>` (unique, sortable), subscribing
   forge bot + Developer, and announces the new channel with a link in
   `#FreeForge`.
3. The request message is posted in that channel (single topic, e.g.
   `request`). agforge reacts in the same topic, with the topic history as the
   run's context — the channel analog of `dm_history`.
4. When the exchange is done, the topic is resolved (Zulip's ✔ rename). The
   channel stays; public channels keep the whole history browsable and
   searchable by the Developer (`channels: public` search operator).

Success = the same result as the DM entrance: a request posted by the
devworld-assistant account makes agforge produce its normal reply, visible to
the Developer in the channel.

## Steps

### Step 1 — manual spike (curl only, no code)

Prove every Zulip 12.2 mechanic before writing code, using the real bot
credentials in `pj-agdev/.local/zulip/` (never print key values):

- Discover user ids (`GET /users`) — devworld-assistant and omni-agent ids are
  unknown; forge=13, developer=8, cagent=14 are known.
- Check whether the devworld-assistant bot can create a public channel
  (`POST /users/me/subscriptions` with a new name + `principals`). If refused,
  fix the realm permission with the Developer (owner) account:
  Zulip 12 group-setting `can_create_public_channel_group`
  (`PATCH /realm`, value may need to be a group id or an anonymous group dict
  including the bots). Record what the default was.
- Create `#FreeForge` and one throwaway `create-…` channel; post as
  devworld-assistant; verify as forge bot that the message is readable
  (`GET /messages`, narrow `channel`+`topic`) — the listener event side was
  proven in zulip_receive with the same queue.
- Resolve the topic as devworld-assistant: `PATCH /messages/{id}` with
  `{"topic": "✔ <topic>", "propagate_mode": "change_all"}`. If a bot may not
  edit another sender's topic, the realm's `move_messages_within_stream_limit`
  / topic-edit permission is the knob; record what was needed.
- Verify searchability as Developer without subscribing to the throwaway
  channel: narrow `[{"operator":"channels","operand":"public"},{"operator":"search",...}]`.

Report: `report1.md` — every permission that had to change, every id
discovered, the exact working curl calls.

### Step 2 — stream mechanics in `agag.zulip` (pyagag)

Extend `pyagag/src/agag/zulip.py`, stdlib-only like the rest:

- `create_channel(name, description, principals, announce=False) -> stream_id`
- `send_to_channel(channel, topic, content) -> message_id`
- `topic_history(channel, topic, num_before=50) -> list[dict]`
- `resolve_topic(message_id, topic)` (the ✔ rename)
- `is_channel_message_for_us(message, self_id)` — a stream message from
  somebody else; the caller decides which channels matter.
- Widen `serve()` (or its filter) so handlers can also see stream messages.
  No backward compatibility owed to current callers.

Add tests beside the existing ones in `pyagag/tests/test_zulip.py`. Make the
change visible to agforge immediately with an editable install into
`agforge/.venv` (`uv pip install -e ../../pyagag` or equivalent) — the
git-pinned `pyagag` in `uv.lock` is updated by a later push, which the
Developer does. Beware: if agforge's launchd services run through `uv run`,
a re-sync can silently undo the editable install — check `service/*.sh` and
pin around it if so.

Report: `report2.md`.

### Step 3 — agforge reacts in `create-*` channels

Extend `agforge/src/agforge/zulip_listener.py` + `zulip_chat.py`:

- React to stream messages **only in channels whose name starts with
  `create-`** (minimal rule; keeps agforge quiet in `#FreeForge` where the
  assistant posts announcements). DMs keep working unchanged.
- Context = the topic narrow's last 50 messages, speaker-labelled, same
  formatting as the DM path; reply goes to the same channel + topic.
- Ack-first behavior carries over (generation takes tens of seconds).
- Reload with `launchctl kickstart -k gui/$(id -u)/com.agdev.agforge-zulip`;
  log at `agforge/.local/out/zulip-listener.log`.
- Verify with a real run: post into the Step-1 throwaway channel as the
  Developer or assistant account and watch agforge answer in-topic. Cost is
  explicitly accepted.

Report: `report3.md`.

### Step 4 — the sender side, in the braindump's order

1. **Omni Agent as devworld-assistant** (curl): full workflow against the live
   listener — create `create-…` channel, announce in `#FreeForge`, post a real
   production request, get agforge's reply, resolve the topic. This is the
   DM-equivalence check. Leave the Deus Ex Machina note.
2. **The assistant itself**: add a small fetch-based Zulip sender to
   `agdevworld/assistant/server.mjs` (credentials already mounted at
   `/run/secrets/zulip.env`; host source `pj-agdev/.local/zulip/devworld-assistant.env`).
   Wire a new endpoint (e.g. `POST /api/forge/channel-requests` with
   `{"desire": "..."}`) that creates the channel, announces, and posts the
   request. The existing `:8092` passthrough may stay or go at the
   implementer's discretion — no compatibility owed. Verify end-to-end through
   the Docker container (`localhost:8091`), not a bare `node` process (macOS
   local-network privacy breaks bare node on this Mac).

Report: `report4.md`, then the episode `report.md`.

## Hints and known facts (from prior episodes and this planning pass)

- Zulip stack: healthy today at `https://agstudio.local:8543`, docker-zulip
  12.2 under `pj-agdev/.local/zulip-selfhost/`; self-signed TLS — `curl -k`,
  and `agag.zulip` already defaults to an unverified SSL context.
- The realm hides email addresses: key everything on numeric user ids.
- `agag.zulip.ZulipClient.call()` already handles dict/list params via JSON
  encoding, form-encoded POST/DELETE, `BAD_EVENT_QUEUE_ID`, timeouts, and
  dropped connections. `PATCH` is not in its vocabulary yet — Zulip accepts
  method override or extend `call()` to allow PATCH bodies like POST.
- Event queues registered with `event_types: ["message"]` already deliver
  stream messages for subscribed channels — the subscription created at
  channel creation is what routes them to agforge.
- One agforge run costs ~0.13–0.33 USD (sonnet profile) and takes 9–35 s.
- `AGFORGE_ZULIP_LOG_ONLY=1` exists but is not required for this episode.
- Zulip naming: the modern API operator is `channel`/`channels`; `stream`
  remains as an alias. Use the modern names in new code.
- Don't put key values or absolute local paths in reports; ids and hostnames
  used by prior episode docs are fine.
