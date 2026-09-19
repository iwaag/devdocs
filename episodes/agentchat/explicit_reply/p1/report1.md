# Step 1 report — Make unanswered work recoverable

Plan: [plan.md](plan.md) step 1. Repository: `pyagag` (shared listener and
topic lifecycle). No consumer was touched in this step; consumer test
suites are re-pinned in step 5 together with their lock files.

## What was wrong, reproduced with fixtures

All four hints from the plan were confirmed by fixtures before being fixed
(`tests/test_serving_lifecycle.py`, which runs the real `serve_topic` under
the real `agag.listen` over a mirrored fake realm with a client whose sends
can be lost, refused, or turn into a crash at a chosen point):

- **Crash after the ack.** The ack is this bot's own post, so the restart
  rule "a reply of ours after the ack means the run finished" and `owed()`'s
  "last speaker is the bot" both read an acked-and-abandoned request as
  answered. The request vanished from the queue.
- **Final-send failure.** The reply's `topic_write` sat outside the handler
  exception boundary; a `ZulipError` there ended the serving with the reply
  lost, and the executor's `finally: queue.finish()` dropped the entry.
- **Ambiguous send.** A send the server accepted whose response was lost
  was indistinguishable from a send that never happened; a retry posted the
  reply twice.
- **Input during a resolving run.** `resolve_after` returned before the
  post-run re-check; the listener's `again` flag re-pended the entry, but
  `owed()` found the topic resolved and dropped it. A human who posted
  while a cancelling run was in flight was never answered.
- **An unrelated later post as evidence.** `owed()` judged by who spoke
  last; a reply of ours that processed input up to id N, posted after a
  human's post N+1 arrived, hid N+1 forever.

## What changed

### `agag.serving` (new) — the serving record

One record per serving with the stages the plan asks to distinguish:
`received` (queue took the entry; the trigger message id), `acked` (the
ack's message id), `executed` (the input boundary `input_up_to` and the
requester), `prepared` (the reply text, destination and resolve flag, on
disk *before* the send), `delivered` (the confirmed message id); plus
`failed` (terminal, with reason) and `interrupted` (what a restart writes
over a record found between `received` and `prepared`).

The record reaches `serve_topic` through a thread-local (`serving.current()`)
bound by the executor, so no consumer passes it by hand. Outside a listener
a `NullJournal` records in memory and `serve_topic` returns it, so a test
or a one-off command can read the outcome.

### `agag.delivery` (new) — exactly-once posting

`deliver()` distinguishes a rejection (`ZulipRejected`, terminal) from an
ambiguous failure (`ZulipError`): before any retry the destination is read
back for this bot's own post of the same text newer than `after_id` (a
realm-global message-id floor taken from the ack and the processed input);
found means delivered. Three attempts with backoff, then `DeliveryError`
with the text still prepared. `redeliver()` reads back first,
unconditionally, for a text that may already have landed.

### `agag.topics.serve_topic`

- Journals every stage; the reply is prepared before it is sent; the send
  goes through `deliver()`. `DeliveryError` escapes the function on purpose
  — the listener owns the retry of the *delivery*, never of the run.
- **One completion rule**: `unprocessed_input(history, self_id,
  processed_up_to)` — speech by somebody else past the boundary this
  serving processed. It is asked after the ordinary reply, **before a
  resolve** (a resolving run that finds new input answers it first and
  defers the resolve to the next result), and by the listener's `owed()`.
- A failed post-run read is recorded on the serving (`extra.recheck_failed`)
  and left to the listener's own evidence rather than silently returned.
- The requester is read from the history the serving processed
  (`requester_of`, bounded by `processed_up_to`), not looked up at send time;
  for a reply posted elsewhere the reply conversation is read once *before*
  the run. This is step 3's contract landing early because it fell out of
  the journal; `handoff_mention` stays for callers that want the send-time
  answer on purpose.
- `resume_prepared()` finishes an interrupted serving: redeliver the
  prepared text, then the resolve it asked for.

### `agag.listen`

- Queue schema v2 (disposable: an old file is rebuilt): `pending` gains
  `next_at` and `failure`; a `servings` table holds the records;
  `QueueJournal` writes them.
- `owed()`: for an owner entry with a delivered record, owed iff there is
  unprocessed input past that record's `input_up_to`. Otherwise the last
  real speaker, **skipping this bot's own acks** (`_last_real`, also in
  `recover()`).
- `_execute_one()`: a prepared (or non-terminally failed) record with an
  undelivered reply is redelivered first, whatever the conversation looks
  like; a delivered mention record without its served mark gets the mark;
  only then is the entry judged and served. The handler runs under
  `serving.bound(journal)`.
- `_retry()`: an exception out of the handler is a transport failure
  (handler failures are already posted as replies). Bounded backoff
  (`RETRY_SECONDS` doubling, `MAX_ATTEMPTS`); exhausted or terminal, the
  entry stays in the queue as `failed` with its reason, visible in
  `entries("failed")`, the status file's `last_error` and the log, and is
  re-armed by the next post in that conversation (attempts reset).
- `_resume_running()`: every running entry is requeued; records between
  `received` and `prepared` are marked `interrupted` and handed to the
  next serving as `context.previous` (the evidence a run may need to
  reconcile actions it started).
- The served mark on the mention route is written by the listener after a
  confirmed delivery, bound to the mention id that triggered the serving
  (`trigger_id`), never to the newest post in the remote topic — so a
  mention arriving during the run stays owed. (Consumers that mark
  themselves keep working; the duplicate note is harmless and is removed
  from Front in step 5.)

## Evidence

`uv run pytest` in `pyagag`: **639 passed** (was 624; 15 new in
`tests/test_serving_lifecycle.py`, 5 call-sequence pins in
`tests/test_topics.py` updated for the removed send-time lookup).

The new fixtures, each asserting eventual response or an explicit failure
state and no duplicate confirmed delivery:

| Scenario | Outcome pinned |
|---|---|
| crash after ack | restart serves it again; `context.previous` carries the interrupted record; one reply |
| crash before the send | prepared reply redelivered; the model does not run again |
| crash after the send | read-back recognises the delivery; nothing posted twice |
| server accepted, response lost | read-back on the first retry; `delivered_id` is the real id |
| send refused (4xx) | entry `failed` (terminal) with reason; status `last_error`; reply kept |
| dropped connection, retries exhausted | entry `failed`, visible; next post re-arms; prepared reply delivered first, then the new input served |
| handler exception | explicit `failed during <step>` reply, delivered; entry done; next post re-arms |
| post-run read fails | recorded on the serving; the listener still finds the input |
| input during a resolving run | resolve deferred, input answered, then resolved |
| second mention during a mention serving | first mark = first mention; second mention served and marked separately |
| restart between home reply and served mark | mark written from the record; no rerun |

Consumer suites run against this checkout (`PYTHONPATH=pyagag/src`):
agobserver 72 passed; agfront, agautolab, agforge, archsage and cagent fail
only where their fakes pin the old call sequence (one fewer `history` read)
or lack `send_to_channel` (the ack and the reply now need the message id).
Those are re-pinned in step 5 with the lock bumps.

## Remaining gaps

- A handler that posts on its own (argue participants) journals only
  `received → executed`; delivery confirmation for those posts is theirs.
- The served mark written by the listener duplicates Front's own
  `note_served` until step 5 removes the latter.
- `context.previous` is available to handlers; the prompt-side use of it
  ("a previous run of this input was interrupted after the ack") is step 4.
