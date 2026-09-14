# better_zulip_call p1 — step 1: the call baseline and the shared boundary

Date: 2026-09-14. Everything below was measured on this host against the
running realm; nothing is taken from earlier documents.

## The environment as found

- `nctl drift` (pj-clusterintent): **converged=46, 0 errors**. Every
  placement in this note is one of those rows; nothing was changed to take
  the measurements except one relay restart, which is one of the scenarios.
- Zulip 12.2, feature level 500. The realm rate-limits **per credential**,
  and the credentials are not equal: `Front`, `autolab-agstudio1`,
  `agforge-agstudio1` and `Cagent` answer `x-ratelimit-limit: 1000`; the
  Developer, `Opsroom Observer`, `agobserver-agstudio1` and `arxivsage`
  answer **200** per minute.
- launchd jobs reading Zulip on this host: `com.agdev.agentroom` (the relay
  — three credentials: the Developer for `/agents`, `/work` and every
  completion read, `Opsroom Observer` for `/ops`, the Developer again for
  writes), `com.agdev.agfront-zulip`, `com.agdev.agautolab-zulip`,
  `com.agdev.agforge-zulip`, `com.agdev.agobserver-zulip`,
  `com.agdev.arxivsage-zulip`, `com.agdev.comfy-notifier`,
  `com.clusterintent.cagent-zulip`. The VM node's `Autolab Agautolab1` bot
  made no call at all in the 22 h window — noted, not investigated here.

## How calls were counted

Zulip's own request log is the ground truth: every request is one line with
method, status, path, the narrow in brackets, and the caller as
`<user id>@root`. Long polls are marked `lp:` and counted separately — one
per queue per ~50–90 s is the floor every listener pays and never the
problem.

The counter is `pj-agdev/devenv/zulip/callcount.py`, fed by
`docker exec <zulip container> cat /var/log/zulip/server.log`. It takes a
time window and an ignored `users.json` (user id → label, from `GET /users`,
kept at `pj-agdev/.local/zulip/users.json`) and prints a per-caller table,
optionally per endpoint. Client-side `calls` counters in the relay's own
payloads (`work.calls`, `gaps.zulip_calls`) were read beside it and agreed
with the log in every scenario below.

## Baseline

### Idle listening — one quiet hour (2026-09-14 02:00–03:00 UTC)

| caller | calls | long polls |
|---|---:|---:|
| Comfy Notifier | 696 | 0 |
| agobserver-agstudio1 | 12 | (in the pool below) |
| every other listener and the relay | 0 | 1221 across 19 queues |

The notifier's command intake polls `GET /messages [is:mentioned]` **every
5 s** — 78 % of all non-long-poll API calls on the realm over 22 h (15,332
of 19,525). It is on its own credential (12 of its 200 per minute) so it
starves nobody, but it is the largest single reader by an order of magnitude
and it reads the same answer 700 times an hour. The Observer's 12 are its
reconcile (`get_stream_id` + `topics`) every tenth tick with no watch active.

### The whole current log (2026-09-13 06:27 → 2026-09-14 04:30, 22 h)

| caller | calls | 429 | what they were |
|---|---:|---:|---|
| Comfy Notifier | 15,332 | 0 | `is:mentioned` poll |
| Developer | 1,841 | **250** | the relay: 675 topic reads, 591 topic listings, 304 `get_stream_id`, 103 `streams`; 18 posts, 6 archives, 16 resolves |
| Omni Agent | 1,260 | 0 | `agentchat` from earlier sessions: 640 topic reads, 297 listings, 297 `get_stream_id` |
| autolab-agstudio1 | 527 | 0 | servings (below) |
| agobserver-agstudio1 | 439 | 0 | watches (below) |
| Front | 60 | 0 | event polls that returned; no serving in the window |
| Opsroom Observer | 58 | 0 | event polls; the relay was not restarted in the window |

### Board reload (the relay, Developer credential)

| scenario | calls | of which |
|---|---:|---|
| cold `/agents` + `/work` | 36 | 20 topic listings, 12 topic reads, 2 `streams`, 2 `get_stream_id` |
| cold `/work` alone | 27 | 19 listings (18 boards + `#agents`), 6 intro reads for `#front` prefixes, 1 `streams`, 1 `get_stream_id` |
| cold `/work?resolved=1` | 27 | the same reads; only the filter differs |
| warm (inside the 30 s cache) | 0 | — |

Today the room sweeps 18 boards (11 open topics); the "~50 calls" figure in
the environment notes was written when there were 47 channels. `/agents`
is ~10 (1 `streams`, 1 `get_stream_id`, 1 listing, 7 intro reads).

### Completion preview (Developer credential; the relay's `closing.Reader`)

| root | actions | Zulip calls | of which |
|---|---:|---:|---|
| `front-desk-20260908-164810` (resolved desk conversation) | 5 | 12 | 6 topic reads (each topic under both names), 3 `get_stream_id` (one 400 on an archived channel), 2 listings, 1 `streams` |
| `front-routine-mediagen` (29 targets) | 29 | 59 | 46 topic reads, 9 `get_stream_id` (6 × 400), 3 listings, 1 `streams` |

Every topic is read under its bare and its ✔ name (two calls) even when
the engine already holds it, because the engine's `Topic.history` is empty
for anything but routine and desk topics.

### The real thing — nine closes in twelve minutes (2026-09-14 03:49–04:01)

The relay's completion history shows nine `/complete` operations by the
Developer between 03:49:35 and 04:00:55. The server log for that window:

| minute | Developer calls | 429 |
|---|---:|---:|
| 03:49 | 270 | 13 |
| 03:52 | 188 | 0 |
| 03:53 | 142 | 0 |
| 03:55 | 207 | 0 |
| 03:56 | 105 | 40 |
| 03:57 | 283 | 23 |
| 03:58 | 236 | 121 |
| 04:00 | 316 | 47 |
| 04:01 | 37 | 11 |

About **1,800 calls and 250 refusals** for nine closes — roughly 200 per
close: preview (a discovery), apply (a second discovery, the writes, a
third discovery for the post-apply answer), then the frontend's reload of
`/agents` and `/work` with the cache just cleared by `room.forget()`. The
web app the Developer had open received 429s too (`/json/events`), which is
the "everything on this account stalls" the problem file describes.

### Restart

| what | calls | measured |
|---|---:|---|
| relay (`Opsroom Observer` sweep) | **120** in 3 s: 56 topic reads, 31 listings, 30 served-note searches, 1 `streams`, 1 `subscriptions`, 1 `register` | today, `kickstart -k`, live at +3 s |
| agfront listener full sweep | 84 | its own log, 2026-09-13 06:07 |
| agautolab listener full sweep | 69 | its own log, 2026-09-13 06:07 |

The relay's sweep now covers 30 channels and 50 unresolved topics; the
"241 calls, about a minute" in the notes is from a larger realm. A listener
restart is deliberately *not* repeated here: Front's log shows one restart
serving a mention on every full sweep (`1 mentioning`), and that is a paid
run. The figures are from the deployed code's own log lines of yesterday.

### An active Observer watch (`w6972`, opened 04:32, met 04:37)

| phase | Observer calls |
|---|---:|
| intake serving (accept, write three notes and the visible post) | 13 |
| idle tick (locate the anchor) | 1 (`GET /messages/<id>`) |
| every tenth tick, reconcile | +3 (`get_stream_id`, listing, one topic read) |
| the met tick (locate twice, deliver, read-back, finish, resolve) | 10 |

Waiting is already cheap here: an evaluation costs nothing on Zulip and the
per-tick lookup is one call. What the watch *record* costs is the intake and
the finish, both once. This measurement is the 60 s interval and the local
model (14–21 s per look).

### One serving

| agent | calls per serving | shape |
|---|---:|---|
| Front (`front-observer-p1`, 2026-09-13 04:11 and 04:13) | 13–25 | 8–12 topic reads (the chatlog, the threads, the `#agents` intros), 2–4 own-note searches, 1 listing, 1–3 posts |
| autolab (`workrun-task3-m6770`, 2026-09-13 08:24–08:30) | ~60, plus ~16 by the run's own `agentchat` | 16–18 topic reads, 2 own-note searches, 4 listings/`get_stream_id`, 5 posts, 1 resolve |

## Where the reads overlap

Five readers reconstruct the same public conversations independently, and
none of them keeps what it read:

1. **The relay's `Room`** (Developer): `streams`, `#agents` listing, every
   board's listing, seven intro reads — on every cache miss, and the cache is
   30 s and is cleared by every completion.
2. **The relay's `Ops`** (Opsroom Observer): the whole realm at startup —
   every unresolved topic's last 50 posts, every channel's served notes —
   then an event queue. It *does* keep state, in memory only, and it keeps
   the history of routine and desk topics only; for everything else it holds
   one `last` message and the link notes.
3. **The relay's `closing.Reader`** (Developer): every related topic, under
   two names, at 400 posts, on every preview and twice more on every apply —
   because the engine's copy is not complete enough to decide from.
4. **Every listener's `sweep_serve`** (its own bot): a full listing-and-read
   of every subscribed channel on every queue registration, and one more
   topic read per event before dispatch, plus the two own-note searches
   (`rootchat`, `served`) and a thread read per remote on every serving.
5. **The Observer's worker** (its own bot): the channel listing and one read
   per topic every tenth tick, and one anchor lookup per watch per tick.

Beside them, `agentchat` (Omni Agent, 1,260 calls in 22 h) reads topics
and listings from a shell with no memory at all, and the notifier reads its
mentions 700 times an hour to find nothing.

The same facts are wanted everywhere: which channels exist and what folder
they are in; which topics a channel has and whether each is resolved; a
topic's messages; the selfnotes (`rootchat`, `served`, `rootchat-moved`,
`replaces`, autolab's `mission`/`task`/`state`/`doc`, forge's
`asset`/`assetrun`/`result`, the Observer's `watch`/`accepted`/`state`,
Front's `delivered`); the introductions and their roster blocks; and, over
all of it, "who really spoke last".

## The boundary chosen

**A shared read model in pyagag, `agag.mirror`, one instance per process,
on that process's own credential.** Not a host-wide service.

- *In pyagag*, because four of the five readers are pyagag consumers already
  and the fifth (the relay) imports pyagag. Nothing frontend-specific goes in:
  the mirror knows messages, channels, topics, notes and introductions; the
  ops rows, the routine board, the closing graph and autolab's and forge's
  record vocabularies stay in the relay, computed *from* the mirror.
- *One per process* rather than one relay serving everybody over loopback,
  for three reasons. A listener must keep serving when the relay is down.
  Each bot has its own quota, so a shared collector saves no quota, only
  calls on the host — and steady state is one long poll per process either
  way. And the account-specific half (DMs, the `mentioned` flag on the
  bot's own queue) needs a queue per bot regardless, so the second queue
  would still exist.
- *Persisted*, so a restart hydrates from disk and a normal consumer
  restart costs no realm scan. The store is disposable: delete it and the
  next start rebuilds it, which is the "rebuild strategy" the plan allows.
- *`all_public_streams: true`* on the queue registration. Verified today
  with the Omni Agent bot: a message posted to `#ops-testbed`, which that
  bot is not subscribed to, arrived on its queue. So the mirror sees every
  public conversation without the subscription writes the ops engine makes
  today, and `work-` channels appear the moment autolab opens them.

The relay therefore becomes one mirror on the `Opsroom Observer`
credential for every read, and the Developer credential is used **only to
write** — which by itself removes the shape of the 03:49 burst: a
completion's writes no longer share a quota with the reads that follow.

What stays account-specific: the DM route (`agag.zulip.serve`, cagent's
window listener) and the per-user `mentioned` flag, both read off the
consumer's own queue. Mentions are also derivable from content
(`@**<name>**`) and the mirror indexes them that way for every bot whose
name the roster declares, so a mention is found in the shared model and
confirmed by the flag when the consumer's own queue carries it.

## The read-model interface (what step 2 builds)

```
Mirror.open(env_path, store_dir, *, log)  -> Mirror   # starts the ingest thread
mirror.health()      -> live, stale_since, queue_id, last_event_id, revision,
                        coverage summary, calls by purpose
mirror.channels()    -> [Channel(stream_id, name, folder_id, archived, description)]
mirror.topics(channel=None, *, include_resolved=True)
                     -> [TopicIndex(channel, name (bare), live_name, resolved,
                         stream_id, first_id, last_id, last_real (Message | None),
                         count, complete)]
mirror.messages(channel, topic, *, since_id=0, limit=None) -> [Message]
mirror.message(message_id)       -> Message | None            # local
mirror.verify(message_id)        -> Message | None            # one targeted read
mirror.hydrate(channel, topic)   -> Coverage                  # fetch what is missing, single-flight
mirror.notes(*, tag=None, sender_id=None, channel=None, topic=None) -> [Note]
mirror.intros()                  -> {instance: Intro(roster, message, retired)}
mirror.changes(since_revision)   -> [Change]                  # for a listener's pending queue
```

`Message` keeps id, stream id, channel, topic (current), sender id, sender
name, sender realm, timestamp, content, edit timestamp, deleted flag.
`Note` is `(tag, value, message_id, sender_id, channel, topic)` for every
`[selfnote][<tag>] <value>` line — generic, so a consumer's vocabulary is
the consumer's. `Coverage` says, per topic, whether the store holds the
whole conversation (`found_oldest` on the last history read) or a window
and how far back.

## What Zulip's event API gives (verified today, feature level 500)

- `register` accepts `all_public_streams=true` and the event types
  `message`, `update_message`, `delete_message`, `subscription`, `stream`
  for a bot; `fetch_event_types=[]` keeps the registration payload empty.
- An `update_message` for a **content edit** carries `content`,
  `orig_content`, `message_id`, `edit_timestamp`. For a **topic move** it
  carries `orig_subject`, `subject`, `message_ids` (every moved message),
  `propagate_mode`, `stream_id`; a move between channels adds
  `new_stream_id`. A resolve is a topic move whose `subject` starts with
  `✔ `.
- `delete_message` carries `message_id`, `stream_id`, `topic`.
- `GET /users/me/<stream>/topics` returns `max_id` per topic, which is what
  a resync can compare against without reading the topic.
- `GET /messages` answers `found_oldest` / `found_newest`, which is the
  coverage flag the store needs.

## Notes for the following steps

- The 429 burst was not the room sweep alone: the biggest single reader in
  it was the completion's discovery (46 topic reads for one preview), run
  three times per close. Step 4 has the most to win per operation; step 3
  removes the reload after it.
- The notifier's poll is outside the plan's named steps but is the realm's
  largest reader. It should register a queue like everything else; step 5
  will take it if the listener refactor makes it a one-line change, and it
  is otherwise recorded here as the first follow-up.
- `retry_after_seconds` in `agag.zulip` reads `x-ratelimit-reset` as a
  number of seconds; on this server it is an absolute epoch. Only the
  `Retry-After` header saves it today. Step 6 fixes it at the transport.
