# lighter_agag_listen p1 — Step 1 report: 429 is its own error class

AI-generated (Omni Agent, 2026-08-17).

## What changed

`pyagag/src/agag/zulip.py`. The latch described in `../problem.md` §2 is a
classification bug: `sweep_serve` treats "you are calling too much" with the
cure for "the connection dropped". Both listener loops now tell the two apart.

**`RateLimited(ZulipError)`** carries `retry_after: float`. `call()` raises it
whenever the HTTP status is 429 — checked *before* the JSON body is
interpreted, so a 429 with an unparseable body is still a rate limit and not a
generic error. The wait comes from, in order: the `Retry-After` header,
`x-ratelimit-reset`, the `retry-after` field Zulip also puts in the JSON error
body, then a 60s default. `QueueExpired` detection is unchanged and still wins
for the 400 it actually arrives on.

Two small pure functions carry the arithmetic so it is testable without a loop:

- `retry_after_seconds(headers, body)` — the fallback chain above.
- `rate_limit_backoff(retry_after, strikes, jitter=None)` — never shorter than
  what the server asked for, never shorter than `RETRY_SECONDS`, doubled per
  consecutive strike, capped at `RATE_LIMIT_MAX_SECONDS` (300), plus up to 10%
  jitter so listeners sharing one quota do not resynchronise. `jitter` resolves
  at call time, not at import time, so it is patchable.

**In `serve` and `sweep_serve`**, a `except RateLimited` arm sits *above* the
generic `except ZulipError`. It sleeps the backoff and does **not** clear
`queue_id` — the queue is fine, and dropping it is what turned one 429 into
360 calls/minute. A consecutive-strike counter grows the wait and is reset to
zero the moment a poll returns. The generic arm keeps its proven
5s-retry-and-re-register behaviour, untouched.

**One extra change in `sweep_serve`, needed for the plan's "the pending work
must survive the wait":** `dirty` was cleared *before* the sweep ran, so a
sweep interrupted part-way lost the fact that it was owed. It is now cleared
*after* the sweep completes. A rate limit mid-sweep therefore leaves `dirty`
true and the retry sweeps. This is the cheap half of problem.md item 3 — the
whole sweep is re-charged, not resumed; step 2's `pending` set is what makes
the resume incremental.

## Evidence

`uv run pytest tests/test_zulip.py -q` → **42 passed** (was 32; the 10 new
tests are listed below). Full suite: 215 passed.

The plan asked for three proofs; each is a test:

| Plan asks | Test |
|---|---|
| no re-register | `test_serve_keeps_its_queue_and_honours_retry_after_on_429`, `test_sweep_serve_keeps_its_queue_and_its_pending_sweep_on_429` — both assert `registrations == 1` |
| the sleep honours the header | same two tests assert the slept value against the 42s the fake server asked for |
| backoff growth and reset on success | `test_rate_limit_backoff_grows_while_429s_repeat_and_resets_after_success` — slept `[10, 20, 40, 10]`: three strikes double, then a poll returns and the fourth 429 is back to the floor |

Plus, on the parsing side: a real `ZulipClient` driven by a fabricated
`urllib.error.HTTPError` proves the header is read
(`test_call_raises_rate_limited_with_the_servers_retry_after`), the fallback
chain (`..._falls_back_to_the_json_body_then_to_the_default`, 12s then 60s),
that a dead queue is still a dead queue
(`..._still_distinguishes_a_dead_queue_from_a_rate_limit`), and the pure
functions directly (`test_retry_after_prefers_the_header_over_the_body`,
`test_rate_limit_backoff_floors_at_retry_seconds_doubles_and_is_capped`).

`test_sweep_serve_resumes_a_sweep_that_a_429_cut_short` covers the `dirty`
change: a fake client that rate-limits the *first* call inside `sweep_topics`
still ends up serving the waiting topic, with one registration.

## What this step does not do

Nothing deploys yet, as the plan directs. The steady-state cost is still one
full sweep per event burst — the listener now survives crossing the quota, but
does not yet avoid crossing it. That is step 2. `x-ratelimit-remaining`
tracking (problem.md item 4) belongs to step 2 as well and is not read here.

Public API is additive only: `RateLimited`, `retry_after_seconds`,
`rate_limit_backoff`, and three constants. `sweep_serve`'s signature is
unchanged, so the four consumers still only need a version bump.
