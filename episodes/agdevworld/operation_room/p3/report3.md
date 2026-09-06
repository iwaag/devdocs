# Step 3 — the first write agdevworld has ever had

`POST /chat` posts one message into a routine topic of `#front`, as the
Developer. It is the first route in this application that changes anything
outside this process — `POST /ops/confirm` writes to the relay's own memory and
nothing else — so the separations are code, not screen.

## Three separations

**A credential of its own.** `AGENTROOM_CHAT_ZULIP_ENV`, a third variable
beside the two read ones, with no fallback in either direction. The `/ops`
observer (`Opsroom Observer`) still never posts; unset means the relay is
**read-only for chat**, and every read payload now carries `chat.configured` so
a view knows before it draws an input box. A chat that refuses only on submit
is the same lie as an empty board.

It is the **Developer's** credential, not a new bot's, and that is the design:
a post from the operation room *is* the Developer speaking. Front is served
because the last real poster in a `front-` topic is not Front — the ordinary
path, with the GUI standing where the Developer's Zulip client would.

**One destination shape.** `#front`, and only `routine-<name>` or
`front-routine-<name>` **of a routine the relay has already found**. Not a
pattern match on the string: the name is compared against the routines the
engine read from the realm, so a topic that does not exist cannot be created
from here either. Posting into another agent's channel is refused because
routing work is Front's job and that is what Single Entrance means.

The rule lives in the relay, exactly as `POST /ops/confirm`'s "only `done` may
be dismissed" does. A view that forgot to hide the box would still be refused.

**A length guard.** This realm answers `max_message_length: 10000` and Zulip
**truncates past it silently** — no error, no marker, the tail simply gone,
which `comfynotify` learned the expensive way. The door sends up to
`AGENTROOM_CHAT_MAX_CHARS` (default 4000) and refuses anything longer with both
numbers in the message. 4000 rather than 9999 because a chat message longer
than that is a document, and a routine's documents belong in its standing
request where Front will read them on the next run.

## Two smaller decisions

**No retry, ever.** A post here starts a paid Front run. The failure a retry
would guard against — Zulip answering an error — is precisely the case where
the first post may already have landed, so a retry can buy a second paid run
for a thing the human asked once. A `502` says what happened and stops.

**A selfnote cannot be typed by hand.** `[selfnote]` at the start of a message
is refused. The notes are one agent's memory of its own run; a human writing
one would be forging a record, which is the thing `agag.selfnote` exists to
prevent, and it is the write-side half of constraint 4.

## Verified live, without sending anything

Against the running relay, with the credential configured:

| request | answer |
|---|---|
| `{"topic": "front-p10-cleanup"}` | `403` — "…is not one of them" |
| `{"topic": "workplan-papers-20260902"}` | `403` — same rule, another agent's topic |
| 5000 characters | `403` — "…over the 4000 this door sends; Zulip itself truncates silently past 10000" |
| whitespace only | `403` — "nothing to send" |
| `chat` in `GET /routines` | `{"configured": true, "max_chars": 4000, "realm_max_chars": 10000, "channel": "front"}` |

Every refusal is a `403` and not a `400`: the request was well formed and the
*rule* is what declined it.

**Nothing has been posted to the realm yet.** The send path's unit tests use a
fake client — one of them raises if a refused message reaches Zulip at all —
and the single real round trip is step 5, deliberately, once, by hand.

`85 passed`.
