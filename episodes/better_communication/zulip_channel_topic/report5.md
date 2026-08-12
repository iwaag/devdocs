# Step 5 report — pivot to topic-per-request

Date: 2026-08-12. Result: requests are now **topics in `#FreeForge`**, not
per-request channels. Live-verified end to end; all suites pass (agforge
71/71, assistant 37/37).

## Why

The Developer read the braindump's `#create-…` as topics inside the standing
channel, while Steps 1–4 had implemented channel-per-request from its literal
"チャンネルを作る". The deciding concern was a future one: a shared project
channel with every agent subscribed, where an asset topic should involve
agforge and autolab but must not fire "strange cagent confusion". That is
suppressible at the listener: every subscribed bot receives every event, but
an `accept` predicate discards non-matching messages before any run starts
(cagent's listener is DM-only today and ignores all channel traffic).
Since per-topic participant restriction was the only thing channels bought —
Zulip has no topic-level ACL, only channel-level access plus per-user
mute/follow — and the fixed three participants don't need it, topics won.

## What changed

- **agforge** (`zulip_listener.py`): the accept rule is now **topic-based and
  channel-agnostic** — any channel message whose topic starts with `create-`
  (plus DMs as before). Resolving renames the topic to `✔ create-…`, which
  stops matching the prefix by itself, so the explicit resolved-topic check
  disappeared. A future project channel works with no listener change: just
  subscribe the bot and use the topic convention.
- **assistant** (`zulip.mjs`): `openForgeRequest` is one call — post the
  desire as a fresh `create-YYYYMMDD-HHMMSS-<hex>` topic in `#FreeForge`.
  Channel creation, principals, self-id lookup, and the announcement message
  all deleted; the topic list is its own index. The API shape
  (`{channel, topic, message_id}`, `/api/freeforge/requests` + `/resolve`)
  is unchanged.
- `GUIDE.md`, the role prompt, and the `#FreeForge` channel description now
  describe the topic workflow.

## Live verification

`POST /api/freeforge/requests` ("paper crane on a wooden desk") → topic
`create-20260812-121324-2b0533` in `#FreeForge` → Forge ack + URL (run
`2c907d42…`, $0.104, 23.0 s) → asset `200`, 114 KB →
`POST /api/freeforge/resolve` → FreeForge's topic list shows
`✔ create-20260812-121324-2b0533`.

## Leftovers

- The Step-1/4 channels (`create-20260812-spike-0001`, `…-204213-8f21e1`,
  `…-114734-b69c3c`) and the old `requests` announcement topic stay as
  episode evidence. Messages there no longer match the topic rule, so they
  are inert.
- The general filtering answer for the multi-agent future, for the record:
  each agent's listener owns one `accept` rule; topic-name conventions,
  required mentions, or sender lists are all expressible there, and an event
  an agent's rule rejects costs nothing.
