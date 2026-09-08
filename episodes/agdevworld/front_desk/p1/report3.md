# Step 3 — relay and conversation integration

agdevworld `agentroom/src/agentroom/frontdesk.py` and its hooks in
`ops.py`, `chat.py`, `server.py`, `main.py`; 18 new tests in
`tests/test_frontdesk.py` (relay suite: 148 passed). Committed as
agdevworld `69e878e` (pj-agdev `f513bac`).

## Three routes

| route | answers |
|---|---|
| `GET /frontdesk` | every Front Desk conversation, newest first, with `status` + evidence; name-only rows for conversations the sweep saw but does not hold |
| `GET /frontdesk/<id>` | the posts (`developer` / `agent` / `ack` / `other`), `latest_reply`, `status`, `history` window, `zulip_url`, and **`known`** |
| `POST /frontdesk/<id>/post` `{text, token}` | a post as the Developer into `#front` › `front-desk-<id>`, once per token |

The Front Desk got an entrance of its own rather than a widened routine check
in `chat.py`: `allowed_topic` still admits only routine topics for `/chat`,
and the desk door shares the text guards (`Chat.text_check`, factored out)
and the credential, nothing else.

## Zulip stays the authority; the queue carries updates

- The `/ops` engine now keeps Front Desk topics' history the way it keeps
  routine topics': the newest 8 (`DESK_DEEP`) are read whole and under ✔ on
  the sweep, every open one is read on the sweep, and the event queue adds
  each new post. The sweep also remembers every `#front` topic name it saw
  (`_front_names`), resolved ones included, so an old conversation is at
  least *listed*.
- **Restore after reload**: the browser re-reads `/frontdesk/<id>` from
  memory — no Zulip call. **Restore after relay restart**: the sweep brings
  the newest conversations back whole; an older resolved one is read from
  Zulip on request with the relay's read credential, under both its names,
  cached 30 s, and reported as `known: read`. When neither can answer the
  payload says `known: unknown` with no posts, and the scene keeps its last
  known history under the amber caption rather than drawing an empty frame.
- **States** are read off the realm's own facts and nothing else: Front's
  transport ack (`agag.agent.is_ack`) is `received`, a post by Front's
  roster id is `answered`, the Developer's post being newest is `waiting`,
  ✔ is `done`. No timer, no harness, no in-flight signal is needed for the
  waiting indicator. A `zulipinternal` notice (the un-resolve notice) is not
  speech, so resuming a conversation does not read as the Developer waiting.

## Boundary checks, in the relay

- `id` is one shape, `^[a-z0-9][a-z0-9-]{0,47}$`; anything else is refused
  before a lookup (`GET` → 400, `POST` → 403).
- length (`AGENTROOM_CHAT_MAX_CHARS`, 4000) and hand-typed selfnotes are
  refused by the shared guard; an unconfigured chat credential names the
  variable.
- **Submit tokens**: the scene mints one per submit; the relay remembers
  the last 200 and answers a repeat with the first result and
  `duplicate: true`, posting nothing. A double click, a second Enter and a
  retry after a timeout all collapse to one run.
- **Uncertain outcomes**: a post that raised after leaving is `502` with
  `uncertain: true` and a note to check the topic; it is never retried, and
  its token is remembered too, so a retry with the same token does not post.
- **Resume**: posting into a ✔'d conversation un-resolves the topic first
  (rename back, `propagate_mode: change_all`) so the post lands in the same
  topic Front's sweep will read; the last message id comes from memory or
  from a one-post read of the ✔ name.
- Credentials: none added. Reads use `AGENTROOM_ZULIP_ENV` (the relay's
  existing read credential), writes `AGENTROOM_CHAT_ZULIP_ENV` (the
  Developer's), both already in the launchd plist. `zulip_url` is built from
  the credential file's realm URL at runtime and is never in a tracked file.

## Live probe (foreground relay on :8095, nothing posted)

```
GET /frontdesk                → live · chat configured · 0 conversations
GET /frontdesk/20260908-1600  → known: read · quiet · zulip_url into #front
POST …/post {"token": ""}     → 403 "a submit token is required…"
POST /frontdesk/Bad/post      → 403 "'Bad' is not a Front Desk conversation id"
```

The launchd relay still runs the previous code; step 4 restarts it.
