# Step 3 — the state engine

`agdevworld/agentroom/src/agentroom/ops.py`, served at `GET /ops` on the
existing relay. No new service: the plan asked for the endpoint, and the agent
room's process already holds a Zulip client and a loopback door.

It is p1's throwaway probe (`.local/opsprobe/stepa2.py`) promoted — the two
serving routes, the served-note differencing, the bare-topic keying — with the
three things a probe does not need: a roster it did not write itself, an event
queue instead of a sweep, and an answer for what to say when it cannot see.

## The shape

```
#agents intro topics ──→ roster (per instance)          ┐
all public channels   ──→ subscribe, then sweep topics  ├─→ in-memory state ──→ /ops
per-channel served notes ──────────────────────────────-┘        ↑
                          Zulip event queue (message, update_message)
```

**Startup:** register the queue *first*, then sweep. A message that lands
during the 240 calls is replayed off the queue rather than lost; the other
order loses exactly the messages that arrive while the sweep is in flight.

**Steady state:** `message` events update a topic's last real post, its
mentions, the served marks and — when the post is an introduction — the roster
itself, in place. `update_message` carries `orig_subject` and `subject` in one
event, which is the whole of the `done` path. No sweep runs on a timer;
re-sweeping happens only when the queue expires.

**Restart is a re-sweep.** Nothing is written to disk, the same decision the
agent room made.

## What `/ops` answers

`{schema, generated_at, settings, health, instances[], rows[], errors[]}`.

A row is one `(instance, channel, bare topic)` with something outstanding, and
it carries the evidence rather than only the verdict:

```json
{"instance": "front-agstudio1", "channel": "front", "topic": "front-a-request",
 "state": "stalled", "route": "owner", "age_seconds": 1020,
 "provenance": {"text": "stalled — 17 min since Developer's post #4931 (threshold 15 min), no served note, owner route",
                "message_id": 4931, "message_at": 1788…, "by": "Developer",
                "served_mark": null, "route": "owner", "resolved": false}}
```

`provenance.text` is a finished sentence, so Step 4 can render a row without
knowing anything about how a state is decided. Everything green on this board
is an inference from a trace somebody left for another purpose; the sentence is
what makes it checkable.

`health` carries the queue's liveness, the last event and sweep times, the
sweep's call count and the channel/topic counts. Rows the engine cannot vouch
for are `unknown` with the last known verdict kept in `stale_state`.

## The five constraints, and where each one lives

1. **No hard-coded roster.** `_read_rosters` parses each `intro-` topic with
   `agag.intro.parse_roster`. `None` stays `None`: an instance with no roster
   block is its own `unknown` row and its own `unknown` instance state.
2. **Event-driven, own credential.** `OPSROOM_ZULIP_ENV`, with **no fallback**
   to `AGENTROOM_ZULIP_ENV` — unset, `/ops` answers `503` and says why. A
   fallback would silently spend the agents' quota, which is the one thing this
   view can do to harm the system it watches.
3. **Unknown is never idle.** A dead queue rewrites every row's state to
   `unknown`, and every instance's too.
4. **Selfnotes are not speech.** `is_real` runs `agag.selfnote.is_selfnote`
   before anything is recorded, so a note never reaches `last`, never reaches
   `mentions`, and can never be the reason a row exists. Zulip's own notices
   (`zulipinternal`) go the same way — and the resolve notice arrives on the
   very event the engine watches for.
5. **It never posts.** The only write it makes to the realm is a
   *subscription*, and it has to: see below.

## Three things the live realm taught that no test could

**1. An event queue delivers only subscribed channels.** A bot reads any
public channel unsubscribed — `#agents` and `#front` both answered before this
bot was in either — but a queue registered by that same bot received the post
made to its own `#ops-testbed` and *not* the one made to `#sandbox`. So the
engine subscribes to every public channel on each sweep, which also picks up
the `work-` channels autolab opens as it goes.

**2. A global search returns nothing for a young credential.** The first live
run reported **53 stalled rows for Front** — every one of them a callback Front
had answered. p1's probe read served notes with
`narrow=[sender:<agent>, search:served]` and got 326; the same narrow from the
observation bot got **0**, and so did `sender:<Developer>`, and so did
`sender:<anyone>`. Zulip answers a search with no channel operator from the
reader's own per-user message index, and an account created yesterday has no
rows in it for anything posted before. Scoped by channel the same search reads
the channel's own messages: `[channel:front, search:served]` returns 354.

The engine now makes **one served-note call per channel** rather than one per
agent, and attributes each note by its sender. It is more correct *and* fewer
calls, and it is right for a credential of any age — which matters, because
the whole point of Step 2 was that this identity is new.

The corrected count matches p1 exactly: 326 served notes on the realm, 324 in
`#front` and 2 in `#agents`, all of them Front's, keying 155 topics. autolab's
10 are gone with the archived `pj-simpleshooter` / `work-s2-*` channels, which
is p1's own footnote about archived channels showing up as a number.

**3. A sweep is followed by a 429, and the queue poll is what pays.** The
first run's `GET events` came back rate-limited and the engine called itself
dead. Two guards now, both `agag.zulip`'s own discipline: stay off the last 40
calls of the quota, and when 429 arrives wait exactly as long as Zulip asked.
A rate limit in the poll loop is a wait, not a resync — resyncing would spend
the budget that caused it.

## Cost, measured

One sweep: **241 calls**, 55 channels, 120 unresolved topics, and it takes
about a minute when the realm is not already throttled. Twice in a relay's
life. Everything after that is one long-polled `GET events`.

Resolved topics are not read on a sweep. `done` is a transition this engine
*watches happen*, not a history it reconstructs — reading every ✔ topic on the
realm would multiply the one cost the plan caps, and a finished conversation is
not what an operation room is for.

## What it says about the realm right now

```
agecho-agautolab1     roster=intro    all zero
agecho-agstudio1      roster=intro    all zero
agforge-agstudio1     roster=intro    all zero
agping-agstudio1      roster=missing  unknown: 1
arxivsage-agstudio1   roster=intro    all zero
autolab-agstudio1     roster=intro    all zero
front-agstudio1       roster=intro    all zero   channel_exists=false
```

The board is genuinely quiet, and both of p1's phantoms are gone: Front's seven
`routine-*` stalls (it declares `front-agstudio1`, which is not a channel — and
the engine reports `channel_exists: false` rather than softening it) and the 59
rows from a second autolab bot with the same prefixes (`Autolab Agautolab1` has
no introduction, so it is not on this board at all).

The one row on the board is the honest one: `agping-agstudio1`, `unknown`,
*"has an introduction on #agents but no roster block, so nothing can be said
about what it owes"*.

## Changed

- `agentroom/src/agentroom/ops.py` — new, the engine.
- `agentroom/src/agentroom/server.py` — `/ops`, `503` when unconfigured.
- `agentroom/src/agentroom/main.py` — `OPSROOM_ZULIP_ENV`,
  `AGENTROOM_STALLED_SECONDS`, and `serve.sh check` now prints the sweep.
- `agentroom/tests/test_ops.py` — 20 new tests; 30 pass.
- `agentroom/README.md` — the operator's half of the above.
