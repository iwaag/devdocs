# operation_room p2 ex1 — confirming `done` by hand

p2's board keeps a `done` row for the life of the relay process. The plan asked
for a way to clear them that is *not* a timer: a human declares they have seen
the evidence, and the row goes.

**It is running**, on the same relay, and the whole cycle has been driven live
in `#ops-testbed` with the browser in the loop.

| step | outcome |
|---|---|
| 1 | `POST /ops/confirm` — all done rows, or one conversation. Only `done` may be dismissed and the relay is what refuses. 43 relay tests (34 + 9). |
| 2 | `✓ confirm N done` on the chip row, a `✓ confirm` button on each done card, and `done` no longer counted under the word *open*. |
| 3 | The full cycle live: stalled → refused → ✔ → done → confirmed away → back on a new post → back on an unresolve. Five screenshots, one visual defect found and fixed. |

## The three constraints

1. **Only `done` can be confirmed**, checked in `Ops.confirm` and not only in
   the view. A button that clears `stalled` off a screen is p9's twenty-six
   unnoticed minutes with a shortcut to it, and a view that merely hides such a
   button is a habit rather than a rule. Proved live: a `409` naming the state.
2. **In memory only.** Visible from outside at the end of step 3 — the relay
   restarted and `confirmed` was `{rows: 0, topics: 0}` again. The marks and
   the rows they hide are then always the same age.
3. **Nothing is written to Zulip.** `confirm` touches one dict. The observer's
   only realm write is still its subscription.

## The one idea worth carrying forward

**Confirm is a mark, not a delete.** What is recorded is `(channel, bare topic)`
plus the id of the last post at that moment, so a row is hidden only while it
is still `done` and nothing newer has landed.

The cheap version — dropping the topic from the engine's table — passes every
test you would think to write and fails in the dark: the later un-✔ rename
arrives naming an `orig_subject` the engine no longer knows, `_apply_update`
returns early, and a re-opened conversation is never shown again. That is
precisely the p9 incident this whole board was built to catch, and it would
have been reintroduced *inside the feature meant to tidy p9's evidence away*.
The plan called this out before a line was written; step 3 then proved the
right version live, un-✔ ing a confirmed topic with no new post and watching
both rows come back as `awaiting`.

## What the screenshots were worth, again

One defect, and again it was neither in the data nor in the build: the confirm
button drawn flush against the card's own border, with the rounded corner
clipping its end. Three phases in a row now (`agent_room` 5, `operation_room`
p2 4, this one), the only defects that survived to the end were the ones only a
picture could show.

The pass also closed `agent_room` step 5's open hole — action-button cards were
never screenshot-verified — and answered the question the code could not: the
button takes the click and the card's evidence popup stays shut behind it.

## Left behind

- The relay is still started by hand, and now it holds a second kind of state a
  restart drops. That is deliberate for the marks; it remains the phase-level
  item p2 already recorded.
- The tasks view could not be re-screenshotted (its Plane backend answers
  non-JSON), so the claim that its cards are pixel-unchanged rests on the
  arithmetic rather than a picture. Pre-existing, and named rather than
  assumed away.
