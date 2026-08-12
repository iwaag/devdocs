# Step 4 report — the sender side, in the braindump's order

Date: 2026-08-12. Result: **both senders work.** First the Omni Agent drove
the whole workflow from the devworld-assistant account by curl; then the
assistant service itself did it through a new endpoint. Both requests got the
DM-equivalent outcome: ack, then a working presigned asset URL, in a channel
the Developer can watch.

## 4-1 Omni Agent as devworld-assistant (curl)

Channel `create-20260812-204213-8f21e1`: created with principals
[Developer 8, assistant 10, Forge 13] → announced in `#FreeForge` (topic
`requests`) → request posted ("red bird on a snowy branch") → Forge ack +
reply with URL (run `728f9986…`, $0.0721, 25.6 s) → asset `200`, 156 KB →
topic resolved as the assistant account → completion note in `#FreeForge`.

Deus Ex Machina note: did the FreeForge send-side walkthrough for agent
devworld-assistant — handoff candidate (and the handoff happened in 4-2).

## 4-2 the assistant service (`agdevworld/assistant`)

New module `assistant/zulip.mjs` (node:http(s), no dependencies):

- `ZulipSender` — Basic-auth client reading the mounted
  `/run/secrets/zulip.env`; per-request `rejectUnauthorized: false` for the
  self-signed realm (scoped, not the process-wide env kill switch); one retry
  on socket-level failures, because the first call right after a container
  start lost its TLS socket once while Zulip was fine (HTTP errors are not
  retried).
- `openForgeRequest(sender, desire)` — the whole send side in one call:
  channel `create-YYYYMMDD-HHMMSS-<hex>` with principals Forge 13 +
  Developer 8 + self (via `users/me`, ids overridable by env), announcement
  in `#FreeForge`, desire posted to topic `request`.

New endpoints in `server.mjs` under `/api/freeforge/` (a separate prefix — 
`/api/forge/` is the passthrough to the `:8092` service and swallows
everything under it):

- `POST /api/freeforge/requests {"desire": "..."}` → 201
  `{kind: "freeforge.request.v1", channel, topic, message_id}`
- `POST /api/freeforge/resolve {"message_id": N, "topic": "..."}` → the ✔
  rename.

The role prompt and `GUIDE.md` both explain the new route and when to prefer
it over `/api/forge/requests` (when the human should see the conversation) —
Tool Giving includes the usage info. The `:8092` passthrough stays for the
browser UI; retiring it is a later episode's decision, same as zulip_receive
left it.

Verified through the container (`localhost:8091`, per the macOS
local-network-privacy note): request → channel
`create-20260812-114734-b69c3c` → Forge reply (run `fef55296…`, $0.0722,
22.3 s) → asset `200`, 142 KB → resolve via the endpoint → topic shows
`✔ request` for every message, and `#FreeForge` carries the open/done index.

Tests: 4 new cases in `assistant/tests/zulip.test.mjs` (env parsing, channel
naming, the open-announce-post order and its recipients, the resolved
marker); suite 37/37.

## Observations

- The one real failure: the first-ever `/api/freeforge/requests` call died
  with an empty socket error right after the container started; the retry in
  `ZulipSender.call` is its direct, evidence-driven fix.
- Zulip's Notification Bot posts into a topic when it is resolved; the
  listener's resolved-topic guard (Step 3) is what keeps that from starting a
  run.
- A truncated log line almost produced a false "asset 403" — the presigned
  URL must be read from the raw message content, never from a display-cut
  transcript.
