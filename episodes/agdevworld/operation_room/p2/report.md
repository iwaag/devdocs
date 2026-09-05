# operation_room p2 — the conversation/task layer board

p1 concluded that the conversation layer is the only layer of this system that
is complete and honest, that `stalled` is the whole value of an operation
room, and that the screen must never render *unknown* as *idle*. p2 built that
board. Steps are reported in `report1.md`–`report5.md`; this is the summary.

**It is running.** `GET /ops` on the agentroom relay holds a live
reconstruction of who owes a reply across the realm, refreshed by a Zulip
event queue, and agdevworld's sixth view renders it with the evidence for
every row.

## What was built

| step | outcome |
|---|---|
| 1 | Every agent's `#agents` introduction now carries a machine-readable **roster block** (`ag.agent-roster.v1`), generated from the running instance. 6 of 7 instances re-posted; Front is on the board for the first time. |
| 2 | `Opsroom Observer`, a bot of its own, so a 241-call sweep does not come out of the agents' quota. `#ops-testbed`, the one channel it may post in, subscribed by nobody else. |
| 3 | `agentroom/ops.py` + `GET /ops`: startup sweep, then event queue; both serving routes, served-note differencing, bare-topic keying, provenance on every row, `unknown` when the queue is dead. |
| 4 | agdevworld's operation room view: owed replies (stalled first) and agents, with `unknown` in amber and a provenance line on every card. |
| 5 | Stalled proved live end to end, both resolutions proved, the phantoms re-checked, three visual defects found and fixed. |

## The five constraints

1. **No hard-coded roster.** Read from `#agents` with `agag.intro.parse_roster`.
   A missing block stays `unknown` — asserted in tests, rendered in amber,
   explained in the popup.
2. **Event-driven, own credential.** One sweep at startup and one per queue
   expiry; `OPSROOM_ZULIP_ENV` with no fallback, so `/ops` answers 503 rather
   than quietly spending the agents' quota.
3. **Unknown is never idle.** Amber, never the grey this app uses for idle
   elsewhere; a card rather than an empty grid when the relay is unreachable;
   `stale_state` keeps the last verdict as evidence and never as the answer.
4. **Selfnotes are not speech.** `agag.selfnote.is_selfnote` runs before
   anything is recorded — proved on the wire in step 5, not only in a fixture.
5. **The observer never posts** outside `#ops-testbed`, and no credential is
   committed. Its one write to the realm is a subscription, which the event
   queue requires and a read does not.

## The four things that were learned by doing it

**1. An event queue delivers only subscribed channels.** A bot reads any public
channel unsubscribed; a queue registered by that same bot did not receive the
post made to a channel it was not in. So the observer subscribes to the realm,
and keeps doing so, because `work-` channels appear whenever autolab opens a
task.

**2. A global Zulip search returns nothing for a young credential.** The first
live run reported **53 stalled rows for Front**, every one a callback Front had
answered. p1's probe read served notes with `narrow=[sender, search]` as the
Developer — an owner subscribed since the realm was built — and got 326. The
same narrow from a day-old bot got 0, and so did `sender:<anyone>`: Zulip
answers an unscoped search from the reader's own per-user index. Scoped per
channel it reads the channel's own messages and is correct at any age. The
corrected numbers match p1 exactly.

This one generalizes past this episode: **a measurement taken with the
developer's account is not a measurement of what a new identity can see.**

**3. A shared topic prefix is a real ambiguity, not an observer error.** p1
charged one bot 59 phantom stalled rows for carrying the same prefixes as the
instance that was really answering. The live test reproduced it with two agecho
instances — and both of them genuinely would sweep the topic. The roster cannot
fix it, because the ambiguity is in the vocabulary. The board states it: two
rows, each naming the other.

**4. Unreadable evidence is the same defect as wrong evidence.** Three visual
defects survived the build, 34 tests and a correct payload: a provenance
sentence that spilled through the card border, four rows that clipped to four
identical titles, and a topic name that drew straight into the neighbouring
card because Phaser wraps only on spaces. On a screen whose entire argument is
*"here is why I believe this"*, a reason nobody can read is no reason at all.

## What is on the board right now

Six instances with a roster, all quiet. One row: `agping-agstudio1`,
`unknown`, *"has an introduction on #agents but no roster block, so nothing can
be said about what it owes"* — a bot and an `intro-` topic that outlived their
project, and the live proof that this board refuses to paint an agent it cannot
route for as an agent with nothing to do.

Both of p1's phantoms are gone: Front's seven `routine-*` stalls and the second
autolab bot's 59.

## Left for p3

The whole process/backend layer, as the plan scoped it: listener health,
in-flight harness runs, ComfyUI and ollama tiles, routine-fire answers, and the
`prompt_id` join. p1's report §5 is the design brief for it, and the trap it
names first — macOS Local Network permission being per binary, so an
unapproved interpreter reports every backend as down — is the one to check
before writing any of it.

Two smaller things this phase leaves behind:

- **`agping-agstudio1` cannot re-post its introduction** because its project no
  longer exists on any machine. Either its `intro-` topic is resolved to retire
  it from the board, or the fixture is regenerated. It is a decision, not a bug,
  and the board is honest about it meanwhile.
- **The relay is still started by hand.** It now holds state that a restart
  costs 241 calls to rebuild, which is a stronger argument for a launchd job
  than the agent room's per-request read ever was.

## Deus Ex Machina note

The Omni Agent ran each instance's own `intro` module (and, on the agautolab1
node, the pull, sync and service restart), created the observation bot and its
testbed channel, and posted the artificial stalls. The introductions and the
rosters are the agents' own; the *triggering* of a re-post is work an agent
could do for itself when told its introduction is stale — handoff candidate.
