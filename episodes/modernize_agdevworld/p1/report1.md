# p1 step 1 — Zulip resources

Done. Front exists as a Zulip bot, has its own credential file, and its
subscriptions already encode the routing this phase needs.

## What was created

- **Front bot** — full name `Front`, short name `front`, generic bot,
  user id **15**. Created through the API from the Developer account.
- **`pj-agdev/.local/zulip/front.env`** — mode 0600, same three keys as the
  sibling files (`ZULIP_URL`, `ZULIP_EMAIL`, `ZULIP_API_KEY`).
- **`#front`** — new channel, stream id **24**, description
  "The Developer's conversation with Front." Subscribers: **8 (Developer)
  and 15 (Front)**, nobody else.
- **`#general` (stream 3)** — Front subscribed itself with its own
  credentials, so the subscription is real in the UI as well.

Front is subscribed to exactly two channels: `#front` (its entrance) and
`#general` (its one outbound channel this phase). Nothing else — subscription
is the routing decision, so an extra subscription would be an extra route.

## Helpers left behind

Both live in the git-ignored `pj-agdev/.local/zulip-selfhost/`, beside
`provision-realm.py` which they are modelled on:

- `provision-front.py` — idempotent: creates the bot only if absent, rewrites
  `front.env` at 0600, re-asserts both subscriptions.
- `zulip-api.py` — `zulip-api.py <cred-name> <METHOD> <path> [k=v …]`, an
  ad-hoc caller against any credential file in `.local/zulip/`. Used for the
  verification below and useful for the later steps.

## Verification (API, 2026-08-17)

| Check | Result |
|---|---|
| `GET users/me` as Front | `15 Front front-bot@… is_bot True` |
| `GET users/me/subscriptions` as Front | `24 front`, `3 general` |
| `GET streams/24/members` | `[8, 15]` |
| `GET streams/3/members` | `[8, 9, 10, 11, 12, 13, 14, 15]` — Front joined the pre-existing set |
| `ls -l front.env` | `-rw-------` |

`#general` already contained both autolab bots (11 agstudio, 12 agautolab1),
as the plan recorded; Front is simply an eighth member there. Only agstudio
runs a Zulip listener, so a `run-*` topic posted there still wakes exactly
one node.
