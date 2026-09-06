# Step 2 — a fire's conversations, reconstructed

`GET /routines/<name>` adds the **session tree**: one fire and everything
opened on its behalf, three runs deep, each node wearing the state `/ops`
already decided for it. Plus the fire topic as a chat log, which step 4 renders.

## Two edges, because each is blind where the other sees

The links are the selfnotes, which is what the plan permits and constraint 4
bounds: they may link, they are never rendered, and the chat log cannot leak
one because the engine drops selfnotes before `history` exists.

**`[selfnote][served] <channel>/<topic> <id>`, written in the fire topic.**
Front writes one into home after answering a callback, so the fire topic
accumulates the name of every conversation the run worked in. It is the only
edge that survives the remote being **resolved** — and a resolved topic is
never swept, so most of a *finished* session is invisible without it.
`front-routine-papers` carries 26 of these.

**`[selfnote][rootchat] <channel>/<topic>`, written in the remote.**
`agentchat send` writes it before the first real post a run makes elsewhere, so
it exists from the moment a child topic is opened — which is the whole of an
*in-flight* session, where nothing has been answered and no served note has
been written anywhere yet.

Neither alone is a session. The live `papers` runs show both in one tree: the
workplan topic arrives by `rootchat` (open, swept, readable) and the autolab
workrun topic by `served` (resolved, never swept, `note-only`).

## Runs are separated by message id

A session's boundary is the **id** of the fire that opened it and the id of the
next fire. Ids are realm-wide and monotonic in Zulip, and every link note has
one, so this needs no clock and no per-topic reasoning.

It has one honest consequence. A note written late about an old callback lands
in the run it was *written* in, not the run the callback belonged to — Front
re-served four `pj-papers` callbacks from August on 2026-09-04, and they are in
that day's session. That is what the record says: the note is when this
conversation started being worked again. `rtnotes`' newest session carries
seven nodes for this reason where three would be the run's own.

## Nodes do not get a second opinion

Every node's state is lifted from `/ops` unchanged — the plan's "do not
implement the state calculation twice", and p1's 66 phantom rows are what the
alternative costs. The tree adds exactly two words `/ops` has no row for:

- `quiet` — swept, on the board, nothing owed in it. Normal.
- `unknown` — this board has never read the topic. Every `note-only` node is
  one, and saying so is the difference between "quiet" and "not looked at",
  which is this whole view's one rule.

Caps: depth 4, 40 nodes. Two agents can anchor each other and an operation
room is not where a cycle should be discovered.

## What the live realm looks like

| routine | sessions | shape |
|---|---|---|
| `papers` | 3 fires, 2–3 nodes each | workplan by `rootchat`, autolab's workrun by `served`, one depth‑2 child |
| `publish` | 3 fires, 2 / 11 / 2 nodes | the 11 is one mission with five tasks, each its own `workrun-` topic |
| `rtnotes` | 3 fires, 7 / 2 / 1 | the 7 is the late-note effect above |
| `mediagen` | **one session, no fire** | 28 nodes, all by `rootchat` — every run of it was started by hand |

`mediagen` is the useful anomaly again: the busiest routine topic on the realm
has no dispatcher fire in it at all, so its whole history is one session by
default. An empty screen would have been the wrong answer for the wrong reason,
so a routine nobody has fired still shows its conversations, with the note
saying why there is only one session.

The chat log is the same history: 82 posts for `papers`, 164 for `mediagen`,
146 for `rtnotes` — real posts only, oldest first.

`78 passed`. The eight new tests are the two edges, depth, the ops-verdict
rule, the fire boundary, the three-session limit, the fire-less routine, and
that a selfnote never reaches the chat.
