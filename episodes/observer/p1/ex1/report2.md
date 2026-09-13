# observer p1 ex1 — step 2: the watch is found by its own anchor

## What changed

The worker no longer compares topic names. Before every evaluation and again
before every notification it asks Zulip **where the watch's own anchor
message is now**, and that single answer settles all three questions at once:

| answer | meaning | what happens |
|---|---|---|
| `open` | the conversation it is in, under its current name | continue, and refresh the cached name |
| `closed` | the anchor sits in a `✔ ` topic | **cancelled** — the whole gesture |
| `absent` | Zulip says the anchor is gone | **removed**, locally and silently |
| `failed` | the lookup got no answer | skip this attempt; conclude nothing |

`Worker.cancelled()` is gone, replaced by `locate()` (the lookup),
`relocated()` (the cache write-back) and `stand_down()` (the decision). Both
of the moments the plan names are now the same call, because they were always
the same question.

## The defect, precisely

`cancelled()` asked whether the *bare cached name* was gone from the channel
listing and a `✔ <that name>` had appeared. A watch renamed before being
resolved therefore looked like neither: the old name was gone, but the ✔ name
that existed was `✔ <the new name>`. Cancellation was not recognized, the
watch kept evaluating, and everything it had to say was addressed to a name
that had moved on — or to whoever had since taken it.

`reconcile()` made it durable. It skipped any record whose state matched and
whose acceptance was present, which is true of every renamed watch, so the
stale name was never corrected even on the ten-tick sweep. It now compares
the fields it would write — including channel and topic — and writes when any
of them differ.

## What a watch says, and where

Failure reports, state notes, the completion post and the final resolve all
address `watch.channel` / `watch.topic`, and that `Watch` is now built from
the record *after* `stand_down()` refreshed it. A rename that lands in the
middle of an evaluation is picked up by the second check, so the completion
post goes to where the conversation is, not where it was when the look
started.

Two stops that are deliberately not alike:

- **cancelled** says nothing. The ✔ was made in the topic itself; a post
  back would only re-open what somebody just closed.
- **removed** says nothing either, for the opposite reason: the anchor is
  gone, so there is no conversation left to say it in. It is recorded in the
  store, which is where a human looking for a vanished watch would look.

## Evidence

`uv run --frozen pytest -q`: **57 passed** (50 after step 1).

New in `tests/test_lifecycle.py`, each one a sentence from the plan:

- `test_a_renamed_watch_is_continued_under_its_new_name` — renamed, still
  evaluated, and the record's topic corrected.
- `test_what_a_renamed_watch_says_goes_to_where_it_is_now` — the failure
  report lands in the renamed topic; the conversation that took the freed
  name gets nothing.
- `test_a_renamed_then_resolved_watch_is_cancelled_without_a_look` — **the
  reproduction**: rename, then ✔. No evaluation, no notification. This is
  what the name comparison could not see.
- `test_a_cancelled_watch_posts_nothing_into_its_reused_name` — the old name,
  now somebody else's conversation, receives no state or completion post.
- `test_a_deleted_anchor_ends_the_watch_locally` — `removed`, nothing posted.
- `test_a_watch_pending_delivery_re_reads_its_own_location` — a rename inside
  the evaluation; delivery is handed the new topic.
- `test_an_unanswered_anchor_lookup_is_not_a_cancellation` — the transient
  failure: the tick is skipped, the state is untouched, and the next tick
  simply works.
- `test_a_reconcile_refreshes_a_name_it_used_to_leave_stale` — and is idle
  once the record is current.

`test_a_resolve_during_the_evaluation_still_cancels` from p1 is unchanged and
still passes, now through the anchor lookup rather than the name comparison.

## Limitations

- Locating costs one `GET messages/<id>` per watch per tick, in place of the
  channel listing the old check shared between watches. For a handful of
  watches on a local Zulip this is the cheaper call anyway (it does not grow
  with the channel), but it is per-watch where the old one was per-tick.
- A `removed` watch leaves its store record and its topic workspaces behind.
  Nothing prunes either, as in p1.
- Delivery still turns a failed destination lookup into a terminal
  `undeliverable`. That is step 3.
