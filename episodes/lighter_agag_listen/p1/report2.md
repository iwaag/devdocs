# lighter_agag_listen p1 — Step 2 report: events become targeted checks

AI-generated (Omni Agent, 2026-08-17).

## What changed

`pyagag/src/agag/zulip.py`. Step 1 made the listener survive crossing the
quota. This step makes it stop crossing it. `sweep_serve`'s signature is
untouched — all four consumers still call
`sweep_serve(client, handler, topic_filter=…)` and need only a version bump.

**`dirty: bool` → `pending: set[tuple[str, str]]` plus `full_sweep: bool`.**
The boolean conflated two different things: "a full re-scan is owed" (true
after a queue registration) and "something happened somewhere" (true after any
event). They are now separate, and only the first is expensive.

**An event names a place to look.** New `topic_from_event(message, self_id,
topic_filter)` reads `display_recipient`, `subject` and `sender_id` off the
payload and applies exactly the rules `sweep_topics` applies to a topic name:
stream message, not our own echo, matching prefix, not `✔ `-resolved. It
returns `(channel, topic)` or `None`. A match joins `pending`; a non-match
costs nothing at all — no call, no flag.

**Serving `pending` is one call per topic.** For each entry, one
`topic_history(num_before=1)` decides whether the last poster is really
somebody else, and only then is `handler(channel, topic)` invoked. The event
is a hint, never the verdict — the "don't trust the payload" discipline the
old loop bought with a whole sweep is now bought with one call. Entries are
**peeked, not popped**: the discard happens after the check returns, so a
`RateLimited` raised inside it leaves that entry *and* every other entry
pending for after the backoff. That is problem.md item 3 without sweep
cursors.

Bursts coalesce for free: five messages on one topic are one set entry, hence
one check, hence one handler run.

**The full sweep is for recovery only.** It still runs on every queue
(re-)registration — startup and `QueueExpired` — so downtime stays lossless,
and it now seeds `pending` rather than calling the handler directly.

**Budget visibility (problem.md item 4).** `ZulipClient` gained `calls`,
`rate_limit_remaining` and `rate_limit_limit`, filled from the
`x-ratelimit-*` headers Zulip puts on *every* response — success and 429
alike, so a listener knows its budget before spending it, for free. Before a
full sweep, if fewer than `SWEEP_BUDGET_RESERVE` (40, about one sweep's worth)
requests are left in the window, the sweep is **deferred**: it logs one line
and falls through to the long poll, which is both the wait and the thing that
refreshes the headers. No sleep is added and no call is wasted; `full_sweep`
stays true, so the pass happens as soon as the window has slid. Each completed
sweep logs exactly one line:

```
full sweep: 1 awaiting, 4 calls spent, 190 left in the window
```

The loop reads the budget with `getattr(..., None)`, so a consumer's own
client stand-in that lacks the attributes still works.

## The cost, before and after

For agforge (21 channels, 8 matching topics), per incoming message in a
`create-` topic:

| | before | after |
|---|---|---|
| a matching message | 30 calls (full sweep) | **1** (`topic_history`) |
| a burst of 5 on one topic | 30 × up to 5 | **1** |
| a message in a non-matching topic | 30 | **0** |
| startup / queue re-registration | 30 | 30 + 1 per match |

The `+1 per match` is deliberate: a topic the full sweep matched is verified
again when it is served, because the pending set is served by one uniform
rule. It costs one call per *actually awaiting* topic on a *rare* code path,
and it is what keeps the serve logic single-branched.

Against the old 200/60s quota, the sustained demand that produced the 10-hour
latch (360 calls/min) is gone: steady state is one long poll per 90s plus one
call per message that could matter.

## Evidence

`uv run pytest tests/test_zulip.py -q` → **52 passed** (42 after step 1). Full
pyagag suite: 225 passed.

The plan asked for four proofs; each is a test:

| Plan asks | Test | Assertion |
|---|---|---|
| a burst of N events on one topic costs one check | `test_sweep_serve_coalesces_a_burst_on_one_topic_into_one_check` | 5 events → `len(history_calls) == 1` |
| a non-matching topic costs zero calls | `test_sweep_serve_spends_nothing_on_an_event_that_cannot_match` | wrong prefix, own echo, resolved, and a DM → `history_calls == []` |
| registration still triggers the full sweep | `test_sweep_serve_sweeps_on_startup_before_any_event`, `..._sweeps_again_after_queue_expiry` (both pre-existing, unmodified) | the waiting topic is served on startup and again after `QueueExpired` |
| a 429 mid-pending leaves the remainder served after recovery | `test_sweep_serve_serves_the_rest_of_pending_after_a_rate_limit` | 429 on the first entry's check → both topics served, `registrations == 1` |

Also new: `test_sweep_serve_turns_an_event_into_one_targeted_check` (the event's
topic is checked and no channel listing is re-read),
`..._rechecks_before_serving_an_event_it_already_answered` (the check overrules
a stale hint), `test_topic_from_event_applies_the_same_rules_as_the_sweep`,
and for the budget: `test_call_records_the_quota_headers_from_a_successful_response`,
`test_call_records_a_spent_budget_from_the_429_itself` (0 is a reading, not a
missing value), `..._defers_the_full_sweep_when_the_window_is_nearly_spent`,
`..._runs_the_deferred_sweep_once_the_window_slides`, and
`..._logs_one_line_per_full_sweep_with_its_cost`.

One pre-existing test was replaced rather than kept:
`test_sweep_serve_resweeps_when_a_message_event_arrives` asserted the removed
behaviour (any event → a whole second sweep). Its successors above assert the
behaviour that replaces it.

## Behaviour that changed on purpose

An event in a topic that does **not** match the filter no longer triggers a
re-scan. Previously it did, so a topic that became awaiting for some *other*
reason could be picked up by an unrelated message. That was accidental, not
designed, and paying 30 calls for it is what this episode exists to stop. The
designed recovery paths remain: the startup/re-registration sweep, and the
topic's own next message. Topic renames (`update_message`) are still not seen
— the old loop missed them too, which the plan records as accepted parity.

## Not done in this step

Nothing is deployed. Step 3 is subscription hygiene, step 4 is the quota
raise, the rollout, and the live latch proof.
