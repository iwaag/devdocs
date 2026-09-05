# Step 5 — looking at it, and making it fail on purpose

Four things the plan asks to prove, plus the visual pass. The visual defects
are reported with the view they belong to (report 4); this is the behaviour.

## 1. Stalled detection, live

`#ops-testbed` (Step 2's exception) with the relay run at
`AGENTROOM_STALLED_SECONDS=60`, so a transition could be watched rather than
waited out. The threshold is configuration, so testing it at 60 s tests the
same code path as 900.

The artificial situation is a topic named `agechoplan-…` — agecho's own sweep
prefix — posted by the Provisioner into a channel **no agent is subscribed
to**. The observer computes an owner-route row for it; agecho's listener
sweeps only its subscribed channels, so nothing is served and nothing is woken.
The stall is real and it is safe.

```
posted message 4933
t+4s            awaiting  0 min unanswered · #4933 by Provisioner · no served note
t+40s           awaiting  1 min unanswered · #4933 by Provisioner · no served note
t+75s (>60s)    stalled   1 min unanswered · #4933 by Provisioner · no served note
--- resolving the topic ---
after ✔         done      ✔ resolved · last post #4933 by Provisioner
health: live | sweeps: 1
```

The whole sequence ran on the event queue: `sweeps` stayed at 1 throughout, so
`awaiting → stalled → done` came from `message` and `update_message` events,
not from a re-read.

## 2. Resolution by a reply, and the selfnote invariant

The other way an owed reply is cleared: the agent answers in the topic. Run
with two instances on the board, both of which really do own the topic (see §4):

```
after the request:
    agecho-agautolab1    awaiting  0 min unanswered · #4942 by Provisioner
    agecho-agstudio1     awaiting  0 min unanswered · #4942 by Provisioner
after agecho-agstudio1 replies:
    agecho-agautolab1    awaiting  0 min unanswered · #4943 by agecho-agstudio1
after a selfnote (must change nothing):
    agecho-agautolab1    awaiting  0 min unanswered · #4943 by agecho-agstudio1
after the ✔ rename:
    agecho-agautolab1    done      ✔ resolved · last post #4943
    agecho-agstudio1     done      ✔ resolved · last post #4943
```

Three things at once. The reply cleared **exactly** the replying instance's row.
The sibling's row survived and correctly re-pointed at the newest real post,
which is now the other agent's. And a `[selfnote]` posted straight afterwards
changed **nothing** — not the row, not the age, not the last speaker. That is
plan constraint 4 proved on the wire rather than in a fixture.

## 3. The ✔ rename, and bare-topic keying

Every `done` above arrived through the `update_message` event, which carries
`orig_subject` and `subject` together. The row's key is the **bare** topic
name, so the rename flips a flag on the existing row rather than opening a
second one beside it — asserted in the tests, and confirmed live by the row
count not changing across the resolve.

This is the p9 mechanism reproduced end to end: a lookup that cannot see past
the `✔ ` rename reads an empty topic and drops the callback silently.

## 4. The phantom re-check — and one phantom that was not one

p1's two worst errors were roster guesses. With the roster read from `#agents`:

- **Front: 0 rows.** p1's first pass reported 7 stalled `front/routine-*`
  topics on the assumption that Front owns `#front`. Front's roster declares
  `front-agstudio1`, no channel of that name exists, the board says
  `channel_exists: false`, and it is served by its `front-` prefix alone.
  155 topics of served notes suppress the mention route correctly.
- **The second autolab bot: not on the board at all.** `Autolab Agautolab1`
  contributed 59 phantom rows to p1 by carrying the same prefixes as the
  instance that was really answering. It has no introduction, so the observer
  knows nothing about it and claims nothing about it.

**But the ambiguity behind those 59 rows is real, and the live test found it.**
Two agecho instances both declare `agechoplan-`, so a `agechoplan-` topic in a
channel neither owns is genuinely owed by **both** — both listeners would
sweep it. p1 read that as observer error; it is not. A prefix says which *kind*
of agent owns a topic and never which instance, and the roster cannot fix that
because the ambiguity is in the vocabulary, not in the observation.

So the board states it instead of arbitrating: two rows, each naming the other
(*"the topic prefix is shared with agecho-agautolab1, which would sweep it
too"*). It cost the relay about fifteen lines and two tests, and it is the only
honest rendering available. Where it does not arise in practice is a topic in
an instance's **own channel**, which is decidable — and there the sibling's
reach into that channel is still stated, because that is the surprising half.

## 5. The unknown paths

- **Relay unreachable.** Stopped the process and reloaded: the view renders an
  amber card and the headline *"⚠ UNKNOWN — the agentroom relay is not
  answering on http://localhost:8094. Every row below is the last thing known,
  not the state now."* Not an empty grid — an empty grid is a calm, green board
  and is the failure mode this whole view exists to prevent.
- **Queue dead, relay up.** Covered by test rather than live: every row's state
  becomes `unknown`, the last verdict is kept as `stale_state`, and each
  caption becomes *"relay not reading Zulip · last known stalled"*. Killing the
  queue under a live Zulip is not something the realm offers on request.
- **Roster missing.** `agping-agstudio1` provides this one live, permanently
  and by accident: a bot and an `intro-` topic that outlived their project. It
  is the single row on the quiet board.

## 6. Cost, and what was left running

The sweep is 241 calls and happens twice in a relay's life. Across the whole of
this step the engine never re-swept: `sweeps` stayed at 1 through every
transition above.

The four proof topics in `#ops-testbed` are resolved. The relay is back on its
default 900-second threshold and the board is quiet apart from
`agping-agstudio1`.

## What the screenshots were worth

Three defects (report 4), none of which the build, the 34 relay tests, or a
`curl` of the payload could have found, because none of them was about the
data. Two of them were **legibility of the evidence** — a card that showed the
right provenance in a form nobody could read, and four rows that showed the
right instances in a form that made them look identical. On a screen whose
entire argument is *"here is why I believe this"*, an unreadable reason is the
same defect as a wrong one.
