# better_zulip_call p1 — step 3: the views served from shared state

Date: 2026-09-14. agdevworld commit "agentroom: every read comes from the
mirror; the Developer credential only writes"; pj-agdev pointer bumped.
The launchd relay on this host still runs the previous code — deployment,
with the pyagag push and relock every consumer needs, is step 7. The new
relay was verified on a side port against the live realm.

## What changed

**The relay holds two credentials and reads with one.** `OPSROOM_ZULIP_ENV`
(the `Opsroom Observer` bot) is the mirror's; `AGENTROOM_CHAT_ZULIP_ENV`
(the Developer) writes and does nothing else. `AGENTROOM_ZULIP_ENV`, the
per-request read credential, is no longer read; a plist that still sets it
is told so at startup. The mirror's store lives in `.local/mirror/` under
the relay (`AGENTROOM_MIRROR_DIR` moves it) and is disposable.

**`Room`** (`/agents`, `/work`) is a query of the mirror: intros from the
`#agents` index, boards from the channel table and the topic index, `#front`
filed by roster prefix as before. The 30-second cache, `forget()` and the
per-request `ZulipClient` are gone. Each payload carries `health` — `live`,
or `stale` with the reason and `stale_since` — instead of a per-channel
`errors` list, and the agent room's headline says "the mirror is stale since
… — showing the last good copy" over the rows it still draws.

**`Ops`** keeps its judgement (the two routes, `awaiting`/`stalled`/`acked`/
`done`, the shared-prefix note, confirmations) and loses its sweep, its
queue, its subscription writes, its served-note search and its event
handlers. It **derives** `_topics`, `_rosters`, `_marks`, `_channels` and
`_front_names` from the mirror once per mirror revision and not again while
the realm is quiet; a board read is a revision check. Every topic is held
with its full history now — resolved ones included — so the routine board
and the session trees read whole conversations rather than the newest 200
of a chosen few. Two behaviours were re-decided because the memory now
persists:

- a `done` row is a receipt for a resolve the mirror **watched happen**
  within `AGENTROOM_DONE_SECONDS` (a day); a topic resolved before the
  mirror existed, or long ago, is history and makes no row. The old engine
  showed receipts for as long as the process lived and forgot them all at
  every restart;
- confirmations (`POST /ops/confirm`) persist in the mirror's store, so a
  restart does not resurrect receipts a human already dismissed.

The `health` block keeps the views' vocabulary (`state`, `reason`, `error`,
`queue`, `last_event_at`, `channels`, `topics`) and replaces the sweep
fields with the mirror's: `last_resync_at`, `resyncs`, `resync_calls`,
`revision`, `stale_since`. The frontend types and the detail popup follow.

**The Front Desk** reads through the engine and, for a conversation whose
coverage is not complete, hydrates it once through the mirror; the
`ReadingClient` reader is gone. `zulip_url` comes from the mirror's realm
URL and the `#front` channel it holds.

**Completion** discovers over `MirrorRealm` — `closing.Realm` answered from
the store, with the walk's shapes kept exactly: a topic read under the name
given, `stream_id` raising for an archived channel, `channels()` listing
live ones. A topic in an **archived** channel is hydrated once on demand
(the mirror's bot can read it; the resync never does, because there are
132 archived channels to 30 live ones and nothing changes in them) and kept
for good, the empty bare-name answer included. After the writes the answer
waits — up to five seconds, usually well under one — for the mirror to carry
them back as events, and every applied result says `confirmed: true` or
`false`; `room.forget()` after a close is gone. The preview's
`gaps.zulip_calls` is now what the discovery actually spent (from the
mirror's ledger) beside `gaps.reads` for what it read from the copy.

## Verification

### Fixtures (`agentroom/tests`, 269 tests, 9.4 s; pyagag 11 mirror tests)

The relay's tests build a `FakeRealm`, mirror it with `mirror_over()` (no
thread; `pump()` applies the fake queue), and read the boards off that. The
rewritten ones:

| test | what it proves |
|---|---|
| a rename flips the row, a real rename moves it | a resolve arriving as an event flips the ops row under its bare key; a rename moves the key; both through the mirror, no `_apply` |
| a resolve older than the receipt window is history | `done_seconds` bounds the receipts |
| resolving / un-resolving an introduction | retirement is a flag read off the realm, live, without a resync |
| the room's boards | retired agents leave `/agents` and their channel is not walked; resolved work only on request; `#front` filed by roster prefix; a post and a resolve reach the next read with no cache to clear |
| the Front Desk | a resolved conversation is held (no hydrate call), its Zulip link comes from the mirror; resuming finds the last post in the mirror; no reader, no TTL |
| each applied write is confirmed | a close over a mirror whose thread applies the writer's resolve returns `confirmed: true` in under the deadline and the post-write plan reads `done` off the copy; a writer whose write never reaches the realm returns `confirmed: false` at the deadline |

### Live, on a side port (`AGENTROOM_PORT=8095`, the launchd job's own environment)

| measure | value |
|---|---|
| start to live | mirror filled in **62 calls, 1.2 s** |
| `/agents`, `/work`, `/work?resolved=1`, `/ops`, `/routines`, three times each | every one 200 in **3–5 ms** warm; Zulip calls in the window: **0** (server log, `Opsroom Observer`: only the fill at 05:07:31–32 and long polls afterwards) |
| the Developer credential | **one** call in the whole session, `users/me`, the chat client learning its own id; no read at all |
| `/routines/<name>`, `/frontdesk`, `/frontdesk/<id>` | 200, from the store; the desk conversation `held`, 14 posts, resolved, with its Zulip link |
| preview `front-desk-20260908-164810` | 5 actions, **the same fingerprint the old relay answers**; 4 hydrate calls the first time (its archived `work-g-17` channel's two topics under both names), **0** afterwards |
| preview `front-routine-mediagen` | 50 actions to the old relay's 29; 54 hydrate calls the first time (eight archived `work-m-*` channels), **0** afterwards; stable across repeats once hydrated |

The larger preview differs from the old one on purpose. The old engine held
only unresolved topics, so a child conversation reachable only by **its own**
root note — a delegate that never called back, a resolved `workplan-` that
named this request — was invisible unless a served note in the parent
happened to name it. The mirror holds every resolved topic with its notes,
and the walk reaches them: 21 more targets, all `done` (already ✔) or `kept`
(in an archived channel, which the realm refuses to move). Nothing became
ready or blocked; what changed is that the plan now shows the whole request.

The first preview after a cold start of a root with archived work channels
can differ from the second (some topics were hydrated during the walk), and
settles once hydration is done. Step 4 rebuilds the preview on the indexed
relationships and revalidates before applying, which is where that belongs.

## What was deleted

`Room._cached`/`forget`/`_intro_topics`/`_read_agents`/`_read_work` and its
per-request client; `Ops.start`/`stop`/`_run`/`_register`/`_fail`/`_resync`/
`_read_rosters`/`_served_marks`/`_poll_forever`/`_apply*`/`_patient`,
`TOPIC_LOOKBACK`, `SERVED_LOOKBACK`, `BUDGET_RESERVE`, `RESYNC_BACKOFF`;
`FrontDesk.reader`/`reader_factory`/`_reads`/`READ_TTL_SECONDS`;
`server.py`'s `room.forget()`; `main.py`'s `AGENTROOM_ZULIP_ENV`,
`AGENTROOM_CACHE_SECONDS` and the `Ops.start()`; the tests' `FakeClient`,
`ReadingClient`, and every `_apply`-driven test.

## Not done in this step, on purpose

- **Deployment.** The launchd job needs pyagag pushed and every consumer
  relocked; that is step 7 with the rest. Until then the running relay is
  the old code and the step 1 baseline still describes it.
- **Live close.** Closing a real request writes to the realm; it is part of
  step 7's bounded demonstration, where "another browser sees the updated
  state" is checked on the deployed relay rather than a side port.
- The agent room's `chatPanel`/`frontDeskState` comments still mention
  Front's sweep where they describe Front, not the relay; they are correct
  about Front until step 5.
