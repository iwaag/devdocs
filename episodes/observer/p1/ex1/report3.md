# observer p1 ex1 — step 3: uncertainty is retried, not concluded

## What changed

Delivery now ends a watch only on an answer **from Zulip**. Three things can
go wrong on the way to one notification, and all three used to end somewhere
they should not have:

| what happens | before | now |
|---|---|---|
| the destination cannot be read | terminal `undeliverable`, nobody told | retry at the ordinary interval |
| the read-back cannot be performed | read as "no delivery there" → send anyway | do not send; retry |
| the send fails | retried (this one was already right) | retried, through one shared path |
| Zulip says the destination is gone or ✔ | terminal | terminal, unchanged |

Everything uncertain goes through one function, `_retry_later`, which keeps
the met result and `pending_notification`, records the reason, counts the
attempt, and returns `False`. The condition is never re-judged: it was met,
and a delivery problem is not evidence about the world.

## The two defects, precisely

**`destination.resolve()` returned an absent-looking result for a read
exception.** `deliver()` then cleared the pending flag, posted "could not be
delivered" into the watch topic, and finished the watch. A watch that had
waited a day for its one moment lost it to a timeout — and the requester was
never told, because Observer deliberately does not open a conversation of its
own. Step 1 gave resolution a `failed` outcome; this step is `deliver()`
learning that `failed` is not `terminal`.

**`already_delivered()` returned `None` both for "I looked and there is no
delivery" and for "I could not look".** The only thing the caller does with
`None` is send, so a failed read-back after an ambiguous send is exactly how
one watch produces two notifications. It now raises, and `deliver()` declines
to send and waits an interval instead.

**One watch's failure took the tick down.** `tick()` guarded `evaluate_one`
but called `_hand_over` — and therefore delivery — outside the guard. A watch
owing a notification to an unreachable destination owes it for as long as
that lasts, which is precisely the situation where every other watch must
keep progressing. The guard now covers both.

## What is claimed, and what is not

The module docstring now states the bounds rather than implying exactly-once:

- the read-back looks at the newest `READBACK_MESSAGES` (40) of the
  destination. An ambiguous send followed by more than 40 messages arriving
  before the retry would hide the delivery and produce a second one. For a
  conversation quiet enough to be waited on this is not a real window; for a
  busy one it is;
- a crash after Zulip accepted the post but before the store was written
  leaves the watch owed. The retry's read-back finds the post — and a
  read-back that fails no longer sends, so this costs a duplicate only in a
  window that is now strictly smaller than it was.

## Evidence

`uv run --frozen pytest -q`: **63 passed** (57 after step 2).

In `tests/test_notify.py`, each against the anchored destination intake now
produces:

- `test_a_destination_that_cannot_be_read_is_retried_not_abandoned` — **the
  reproduction**: the state stays `active`, the notification stays owed,
  nothing is posted into the watch topic about being unable to deliver, and
  the next attempt delivers once.
- `test_a_failed_read_back_does_not_send` — an ambiguous send that landed,
  then a failed read-back: no second post, and the third attempt recognizes
  the delivery and finishes.
- `test_a_restart_while_delivery_is_pending_still_delivers_once` — the store
  is all that carries over, and the met result survives it.
- `test_a_deleted_destination_is_still_terminal` and
  `test_a_refused_destination_lookup_is_terminal_and_a_silent_one_is_not` —
  retrying uncertainty did not make a real deletion retry forever, and the
  two cases are shown on the same watch.

In `tests/test_lifecycle.py`:

- `test_an_undelivered_watch_is_not_re_judged_and_does_not_block_the_others`
  — a met watch whose delivery raises every time: it is never looked at
  again, it keeps its pending notification, and the watch beside it is
  evaluated on every tick regardless.

The p1 tests for a deleted and a closed destination are unchanged and still
pass, which is the point: terminal is still terminal.

## Limitations

- A watch whose destination is unreachable for a long time retries silently
  once per interval forever. There is no streak report for delivery the way
  there is for a failed look, and no cap on attempts. `delivery_attempts` and
  `delivery_error` are in the store for a human who goes looking.
- The retry interval is the ordinary tick interval. There is no backoff, so a
  destination that is down costs one Zulip call per watch per minute.
- Real transport behaviour is exercised through controlled failures, not
  against the live realm — deliberately, as the plan asks.
