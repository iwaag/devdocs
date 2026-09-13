# observer p1 ex1 — step 1: a destination is an id before it is accepted

## What changed

A destination written as `<channel>/<topic>` is now resolved **at intake** to
a concrete message id in the conversation the requester meant, and that id is
what the accepted record carries. The name is kept beside it, for saying what
the requester wrote; nothing looks a destination up by name again once an id
exists.

Before this, `intake.py` copied the destination text verbatim into the
accepted note, and `notify.route()` re-parsed that text at send time. For a
name, "re-parse at send time" means asking Zulip *which conversation is
called that today* — so a rename freed the name, somebody else took it, and
the notification went to them. The p1 tests did not catch it because the one
rename test starts from a record that already holds `msg:<id>`.

### The three-valued answer, introduced here rather than in step 3

Step 1 cannot be honest without it: "I could not check this destination" and
"this destination is not there" have to lead to different replies, and
`destination.resolve()` previously returned the same absent-looking value for
both. So the result type arrives now and step 3 will apply it to delivery.

`destination.Resolved` carries an `outcome` of `open`, `absent`, `closed` or
`failed`, with `terminal` meaning *answered, and the answer is no*.

### The shared client had to learn the same distinction (pyagag `f613feb`)

`ZulipClient.message()` caught every `ZulipError` and returned `None`, so a
timeout reading an anchor was indistinguishable from that anchor having been
deleted. `conversation_of` inherited it, and Observer built a terminal
decision on top of it.

- `call()` now raises **`ZulipRejected`** (a `ZulipError` subclass) for an
  answered 4xx, and keeps a plain `ZulipError` for a 5xx, a timeout or a
  dropped connection. 429 stays `RateLimited` — it means *wait*, which is the
  opposite of a fact about the object. `QueueExpired` became a subclass of
  `ZulipRejected`, which it always was in substance.
- `message()`, `conversation_of()` and `topic_history_across_resolve()` take
  `strict=True`: under it, absence means Zulip said so and an unanswered call
  is raised. **The default is unchanged**, and the lenient path makes the
  exact same call it always made, so every other agent and every existing
  stand-in client is untouched.

pyagag is a shared checkout *and* an installed dependency; the change is
committed and pushed (`f613feb`) and `agobserver/uv.lock` is re-pinned to it.

## Behaviour now

| input | stored | on a rename of that conversation |
|---|---|---|
| `front/front-x` | `destination_id: <newest message there>` | follows the conversation |
| a message link / `msg:<id>` | `destination_id: <that id>` | follows the conversation |
| a phrase ("tell me") | not accepted | — |

At intake a destination that **cannot be checked** leaves the request
unaccepted with `needs-input`, and the reply explains the lookup failure
without asking for a different conversation — an outage is not the
requester's problem to route around, and the conversation they named is very
likely fine. A destination Zulip **confirms** is absent or closed is the one
case that asks them to name another.

A watch accepted before this change carries a name and no id; `notify.route`
falls back to resolving the name, so nothing existing stops delivering. No
migration, as the plan allows.

## Evidence

`uv run --frozen pytest -q` in `agobserver`: **50 passed** (was 33).
`uv run --frozen pytest -q` in `pyagag`: **592 passed** (was 584).

New, in `tests/test_intake.py` — the real intake path with only the model run
stubbed, so what is asserted is what reaches the **Zulip** record:

- `test_a_named_destination_is_stored_as_a_message_id` — the normalization
  itself, checked in the accepted note rather than the local cache.
- `test_a_message_link_is_stored_the_same_way` — one stored representation.
- `test_the_notification_follows_a_named_destination_through_a_rename` —
  accept a name, rename the conversation, let somebody else take the freed
  name, deliver: the original conversation is told and the impostor is not.
  **This fails without the fix.**
- `test_the_anchor_survives_rebuilding_the_store_from_zulip` — the same, with
  the store deleted and rebuilt by `reconcile()` in between.
- `test_a_destination_that_cannot_be_checked_is_not_accepted_and_not_reassigned`
  and `test_a_destination_that_is_confirmed_gone_asks_for_another` — the two
  replies that must not be the same one.

New, in `tests/test_destination.py`: the four outcomes, including a refused
lookup staying absent and an unanswered one recovering on the next call.

In `pyagag/tests/test_zulip.py`: 4xx versus 5xx classification, and `strict`
on all three helpers.

## Limitations

- The anchor taken for a named destination is the conversation's **newest**
  message at intake. Zulip can move a single message between topics, so an
  anchor that is individually moved would take the destination with it. Any
  choice of anchor message has this property; the newest costs one call.
- The model run itself is stubbed in the intake tests. What the local model
  does with a request is p1's evidence and is unchanged here — this step only
  moves what happens to the destination *after* the decision.
- Delivery still treats a failed lookup as terminal. That is step 3.
