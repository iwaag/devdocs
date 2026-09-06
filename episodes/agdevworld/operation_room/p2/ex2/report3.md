# ex2 step C — retiring `agping-agstudio1`

The board's one live row is gone, and it went the way the plan chose: the
agent was **retired**, not regenerated.

`agping-agstudio1` was p3's fixture in `runsmoke1` — an agent created by
another agent, the proof that the chain works. Its project exists on no
machine now, so it can never re-post an introduction, and p2's board could
only say *unknown, in amber, forever*. Regenerating a whole fixture to satisfy
a card would have been the tail wagging the dog; `agag init` from the
`runsmoke1` record can make it again the day anyone wants it.

## The write to Zulip

One, and it is the episode's only one: `✔ intro-agping-agstudio1` in
`#agents`, made **with the developer credential** (constraint 1). The observer
bot wrote nothing, as ever. `propagate_mode: change_all` over the topic's five
posts, and the topic is the only thing that changed — no message was posted,
deleted or edited.

## The resolve alone did nothing, exactly as the plan suspected

This was checked before any code was written, and it is worth stating as a
measurement rather than a reading:

```
/ops    → rows: 1 — agping-agstudio1 unknown "introduction carries no roster block"
/agents → agping-agstudio1  ✔ intro-agping-agstudio1
```

The relay was *designed* to survive the ✔ rename — that is the p9 lesson,
written into `room.py` as a docstring: *"a resolved introduction topic is
still an agent rather than a disappearance."* Which was right when nothing
used resolution to mean anything about an agent, and became the wrong default
the moment the realm needed a way to say one had gone.

## So both halves learned the rule

**A ✔ on an `intro-` topic retires the instance.** There is no other signal
available: a project can be deleted from every machine and Zulip never hears
about it, which is precisely how this row was born.

- `/ops`: `Ops._retired`, filled by the sweep and kept current by
  `update_message` events, so a resolve — or an un-resolve — retires or
  restores an agent without a restart. Retired instances are skipped when the
  snapshot is built.
- `/agents`: `Room._intro_topics` returns the live introductions and the
  retired names separately.
- `/work`: a retired agent's own channel is no longer walked for open work.
- Both payloads carry `retired: [...]`, and both views print `· N retired` in
  the subtitle. **An agent that merely stopped being drawn would be the same
  screen as a relay that failed to read it**, which is the one thing this
  board may never be.
- It is a **mark, not a deletion** — the same shape ex1 chose for `confirm`,
  for the same reason. Un-✔ the topic and the agent is back on the board, live.

Seven new tests (51 total, all passing), including the un-resolve and a
resolve elsewhere retiring nobody.

## What the screenshots were worth — a fourth time, and this one was mine

The change broke the agent room. `_intro_topics` has **two** callers, and the
second is the work half; it unpacked two values from a function that now
returns a tuple of two lists.

`GET /agents` answered perfectly. The 50 tests passed. The build passed. The
screenshot said:

> **the agent room is unreadable**
> `ValueError: too many values to unpack (expected 2)`

`GET /work` would have said it too, and I did not ask it — I asked the route I
had just changed. That is the shape of the mistake, and it is the same shape
as the three visual defects of p2 step 4 and the clipped button of ex1: the
build has no way to know which of its outputs a human was going to look at.
The regression test now exercises `_read_work` through a fake client, because
a signature with two callers is the one place a type checker would have earned
its keep and this project does not run one over Python.

## The board now

| | |
|---|---|
| `/ops` | `live`, **0 rows**, `retired: ["agping-agstudio1"]`, 6 instances |
| `/ops` sweep | 239 calls (was 241 — a retired introduction's history is not read) |
| `/agents` | 6 agents, `retired: ["agping-agstudio1"]` |
| `/work` | 93 topics in 46 channels (was 94 in 47 — agping's channel had one open topic) |
| operation room, agents mode | *"6 agents on the board · 1 retired"*, six green cards, no amber |
| agent room, agents mode | *"6 agents have introduced themselves · 1 retired"* |
| agent room, open work | 9 boards, no `agping-agstudio1` |

Four screenshots in `agdevworld/.local/shots/ex2/` (ignored, as ever).

**The agping Zulip bot account was left enabled.** The plan makes that
discretionary and the reason to leave it is that nothing now reads it: it is
off both boards, it has no listener, and a disabled account is one more thing
to undo if the fixture is ever regenerated. It is named here so it is a
decision rather than an oversight.

## Constraints

1. The observer bot wrote nothing. The one realm write was the ✔, as the
   Developer.
2. No credential value and no absolute local path in a tracked file.
3. Nothing about `confirm` changed.

## Deus Ex Machina note

The Omni Agent resolved the introduction of an agent that could not resolve
its own — there was no process left to ask. Not a handoff candidate: an agent
whose project no longer exists is precisely the case no in-system agent can
handle for itself. What *is* a candidate is the sibling p2 already recorded —
an agent told its introduction is stale re-posting it — and this phase is the
proof of what happens when that is impossible.
