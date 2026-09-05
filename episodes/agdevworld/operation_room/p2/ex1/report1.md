# Step 1 — `POST /ops/confirm`, the only way a row leaves this board

The p2 board keeps a `done` row forever. `done` is the ✔ rename witnessed by
the relay's event queue, so the rows accumulate for as long as the process
lives and nothing ever takes them off. The plan's answer is not a timer but a
human: a row is dismissed when somebody says they have seen it.

## What was added

`Ops.confirm(target=None)` and one POST route.

```
POST /ops/confirm            (empty body, or {"all": true})   → every done row
POST /ops/confirm            {"channel": …, "topic": …}       → one conversation
```

The answer is `{"confirmed": n, "topics": [...], "refused": [...]}`. `409` when
the target is not `done`, `404` when it is not on the board at all, `503` when
the engine has no credential — the same answer `GET /ops` already gives.

## The three constraints, and where each one lives

**1. Only `done` may be confirmed, and the relay is what refuses.** Not the
frontend. The plan's reasoning is exact and worth keeping: a button that clears
`stalled` off a screen is p9's twenty-six unnoticed minutes with a shortcut to
it. A view that merely hides such a button is a habit; a check in
`Ops.confirm` is a rule. `test_live_debt_cannot_be_confirmed_and_the_relay_is_what_refuses`
asks the engine directly, with no view in the loop.

The check reads `shown_state(row)` — `stale_state` if the queue is dead,
`state` otherwise — because confirm has to act on **what the human is looking
at**. While the queue is down every row reads `unknown` and carries its last
verdict underneath; the `done` receipt there is still a receipt and the stall
beside it is still a stall.

**2. In memory only.** One dict on the `Ops` object, `_confirmed`, and no file
anywhere. The marks and the rows they hide are then the same age exactly: a
restart drops both, so the board can never open showing debts somebody already
cleared, and two browsers cannot disagree about what has been seen.

**3. Nothing is written to Zulip.** `confirm` touches `self._confirmed` and
returns. The observer's only realm write is still its subscription.

## Why it is a mark and not a delete

The plan's implementation hint, taken. What is recorded is the row *as it
stood*: `(channel, bare topic)` and the id of its last post. A row is hidden
only while it is **still done and no post newer than that id has landed**.

That buys two behaviours for nothing:

- A reply into a closed topic floats the row straight back up
  (`test_a_new_post_floats_a_confirmed_row_back_up`).
- **An unresolve brings it back with no new post at all**, because the row
  stops being `done` and the hide only applies to `done`
  (`test_an_unresolve_brings_the_row_back_even_with_no_new_post`).

The alternative the plan warns against — `del self._topics[key]` — fails the
second one silently and in the worst possible way. The later un-✔ rename
arrives at `_apply_update` naming an `orig_subject` the engine no longer knows,
the lookup misses, and the event is dropped. A re-opened conversation that
nobody is shown is the exact shape of the p9 incident this board exists to
catch, so the cheap implementation would have reintroduced it inside the
feature meant to tidy it away.

## Counts, and the small honesty

A hidden row is also decremented out of its instance's `counts.done` and
added to a new `confirmed` count on the agent summary, and the payload gained
`"confirmed": {"rows": n, "topics": n}`. Without that the board would hide a
row and the agent card beside it would still be counting it — two numbers for
one debt, which is the class of defect this whole view is against.

## The door

`server.py` grew `do_POST` and its docstring grew a paragraph, because it
opened by declaring itself read-only and saying that a write route would need
cagent's other doors rather than a flag. That statement stands and is not
being quietly walked past: this POST writes to *this process's own memory*,
reaches neither Zulip nor any node, and its whole effect dies with the relay.
A route that could change the realm would still need a different door.

`OPTIONS` now answers `GET, POST, OPTIONS`, which the browser needs before it
will send a cross-origin POST with a JSON content type.

## Proof

`43 passed` — the 34 from p2 step 3 plus 9 new ones, covering: hides done,
leaves the stall, refuses the stall by name, single-topic confirm, unknown
topic, the new-post resurface, the unresolve resurface, the dead-queue case,
and the counts.

The HTTP door itself was driven in-process against a fake board rather than
mocked:

```
rows before   : ['b', 'a']            (b stalled, a done)
refuse stalled: 409 {'refused': ['stalled'],
                     'error': 'only done rows can be confirmed; stalled is still owed'}
bad pair      : 400 {'error': 'channel and topic go together'}
empty body    : 200 {'confirmed': 1, 'topics': [{… 'topic': 'a', 'message_id': 1}]}
rows after    : ['b']
POST /nope    : 404 {'post': ['/ops/confirm']}
```

Live `#ops-testbed` proof is step 3, with the frontend in front of it.
