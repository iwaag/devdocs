# p1 plan — remove the rate-limit latch, make the sweep event-driven

Goal (reconcile): no agag listener can latch into the 429 spin described in
`../problem.md`. Concretely:

1. A rate-limited listener backs off honouring the server's `Retry-After`,
   keeps its event queue, and recovers on its own — proven live by
   temporarily lowering one bot's quota and firing a burst at it.
2. Steady-state cost of one incoming message is ~1 API call, not a full
   sweep. The full sweep runs only at startup and queue re-registration,
   which keeps the lossless-downtime guarantee.
3. The bots' per-user quota is raised server-side, as headroom — not as the
   fix. The latch fix must be provable with the *old* quota (that is what
   the lowered-quota test in (1) demonstrates).

Backward compatibility of `sweep_serve`'s signature is required — four
consumers call it (agforge, agautolab, agfront, cagent) and this phase
should not have to edit their handler code, only bump pyagag and restart.

## Facts discovered during planning

- The measurements and the three-layer root cause are in `../problem.md`:
  200 req/60s per user, ~30 calls per agforge sweep, fixed 5s retry that
  drops and re-registers the queue on *every* `ZulipError`, 360/min sustained
  demand against a 200/min budget. 10 hours stable.
- `sweep_serve` deliberately ignores the event payload
  (`pyagag/src/agag/zulip.py`, end of the loop) — but the payload carries
  `display_recipient` (channel), `subject` (topic) and `sender_id`, which is
  everything needed to turn an event into one targeted check instead of a
  full sweep. The "don't trust the event" discipline is preserved by
  verifying with `topic_history(num_before=1)` before dispatching, exactly
  the check `sweep_topics` already does per topic.
- `register()` subscribes to `event_types=["message"]` only. A topic that
  becomes awaiting through a rename (un-resolving a `✔ ` topic) emits
  `update_message`, not `message` — the *current* loop misses that too, so
  parity is acceptable; noted under out-of-scope.
- `urllib.error.HTTPError` exposes response headers via `.headers`, so
  `Retry-After` / `x-ratelimit-*` are readable where `call()` already
  catches the 429.
- Every Zulip response carries `x-ratelimit-remaining`; the client can track
  it for free and know the budget before spending it (problem.md item 4).
- agforge and cagent run **two** loops as one bot user (DM `serve` thread +
  `sweep_serve`); agautolab and agfront run `sweep_serve` alone. Merging the
  dual loops into one queue would halve their idle cost, but once the sweep
  is event-driven the steady-state cost of a second loop is one long-poll
  per 90s — negligible. Deferred (out of scope) to keep this phase's blast
  radius inside pyagag.
- The Zulip server is docker-zulip at `agstudio.local:8543`, deployed from
  `pj-agdev/.local/zulip-selfhost/`. Two ways to raise limits:
  - **Per-user**: `UserProfile.rate_limits` (a `"seconds:count,…"` string,
    empty = server default) via `manage.py shell` in the `zulip` container.
    Targeted, survives restarts (it is in the DB), leaves humans and the
    server default untouched. Also the mechanism for the *lowered*-quota
    latch test.
  - **Server-wide**: `ZULIP_CUSTOM_SETTINGS` is appended verbatim to
    `/etc/zulip/settings.py` by `entrypoint.sh` (`zulipConfiguration()`),
    so a `RATE_LIMITING_RULES` override is possible — but it replaces the
    whole dict (several categories besides `api_by_user`), so per-user is
    the safer instrument. Use per-user.
- Dead subscriptions (17 of forge's 21 are finished `pj-*` experiment
  channels) multiply the *startup* sweep cost even after the redesign.
  `ZulipClient` has `subscribe_channels` but no unsubscribe.
- `agag.status` already makes the spin *detectable* (the status file goes
  stale because it is written only on success); what is missing is a reader.
  That is monitoring work in pj-clusterintent territory — its own episode,
  out of scope here.

## Steps

Write `report<N>.md` beside this plan after each step. Steps 1–2 are pure
pyagag work with tests against the existing fake-client fixtures
(`pyagag/tests/test_zulip.py`); nothing deploys until step 4.

### Step 1 — 429 is its own error class, and it does not cost the queue

In `agag/zulip.py`:

- `RateLimited(ZulipError)` with a `retry_after: float` attribute. `call()`
  raises it on HTTP 429, taking `retry_after` from `Retry-After` (fall back
  to `x-ratelimit-reset`, then to a default).
- In **both** `serve` and `sweep_serve`, catch `RateLimited` *before* the
  generic `ZulipError` arm: sleep `max(retry_after, RETRY_SECONDS)` plus
  jitter, doubling on consecutive 429s up to a ceiling (~5 min), and do
  **not** clear `queue_id` and do **not** touch `dirty` — nothing is wrong
  with the queue, and the pending work must survive the wait. If the queue
  dies during a long backoff, the existing `QueueExpired` arm already
  recovers it.
- The generic arm (transient disconnects, Zulip restarts) keeps its proven
  5s-retry-and-re-register behaviour unchanged.

Tests: a fake client returning 429-with-header shows (a) no re-register,
(b) the sleep honours the header, (c) backoff growth and reset on success.

### Step 2 — events become targeted checks; the full sweep is for recovery

Rework `sweep_serve`'s loop, keeping its signature:

- Replace the `dirty` boolean with `pending: set[tuple[str, str]]` of
  `(channel, topic)`.
- On a `message` event: skip unless `type == "stream"`, sender is not this
  bot, the topic matches `topic_filter`, and it does not start with
  `RESOLVED_TOPIC_PREFIX`. Otherwise add `(channel, topic)` to `pending`.
  Bursts on one topic coalesce in the set for free (problem.md item 2).
- Serving `pending`: for each entry, one `topic_history(num_before=1)` call
  verifies the last poster is somebody else (the event is a hint, not the
  truth), then `handler(channel, topic)`. Entries are consumed one at a
  time, so a `RateLimited` mid-way keeps the rest pending — the resume
  problem.md item 3 asks for, without sweep cursors.
- Queue (re-)registration still sets up a **full** `sweep_topics` pass, by
  seeding `pending` with a sentinel or a separate flag — downtime stays
  lossless.
- Budget visibility (problem.md item 4): `ZulipClient` records the last
  `x-ratelimit-remaining` it saw (headers are available on *successful*
  responses too). `sweep_serve` logs it when starting a full sweep and
  defers the sweep (short sleep, retry) when remaining is under a reserve
  (~40, one sweep's worth). One log line per sweep: matched count, calls
  spent, remaining.

Tests: fake-client event scripts proving a burst of N events on one topic
costs one check; a non-matching topic costs zero calls; registration still
triggers the full sweep; a 429 mid-pending leaves the remainder served
after recovery.

### Step 3 — subscription hygiene

- Add `ZulipClient.unsubscribe_channels(names)` (`DELETE
  users/me/subscriptions`).
- One-off pass (script under the episode's `.local/`, or `zulip-api.py`
  interactively): for each of forge / autolab / cagent bots, list
  subscriptions, unsubscribe the dead `pj-*` experiment channels. Record
  before/after subscription counts per bot in the report.
- Note for the episode's closing: experiment teardown should include
  unsubscribing bots — where that rule belongs (devpolicy, episode
  template) is decided at the end, not silently here.

### Step 4 — raise the bots' quota, roll out, and prove the latch is gone

- Per-user raise: in the `zulip` container, `manage.py shell`, set
  `rate_limits = "60:1000"` for the four bot users (forge, autolab ×2 nodes
  share one bot? — check; front, cagent). Verify with any authenticated
  call: `x-ratelimit-limit` now reads 1000. Document the command in the
  report so it survives a Zulip rebuild (the DB persists, but the knowledge
  should not live only in one head).
- Roll out pyagag: commit, push to GitHub (pyagag is consumed from GitHub —
  never repoint at the gitea mirror), bump/reinstall in agforge, agautolab,
  agfront, cagent per their dependency spelling, restart the listeners
  (launchd `kickstart -k` on agstudio; the ansible'd node for agautolab1;
  cagent per its own deploy).
- **Latch proof, live**: pick one bot (forge), temporarily set its
  `rate_limits = "60:30"` — *below* one sweep — restart its listener, and
  post into a `create-` topic. Expected: one 429 → one logged backoff
  honouring `Retry-After`, **zero** `registered event queue` lines during
  the backoff, recovery and a served topic afterwards. Restore `60:1000`.
  Log excerpt into `.local/` as evidence.
- Steady-state proof: with normal quota, post a burst into one topic and
  confirm from the log exactly one targeted check ran (no full sweep), and
  `x-ratelimit-remaining` stays near the ceiling.

## Out of scope (later phases or separate episodes)

- **Silence-is-an-alarm** (problem.md item 6): a reader for stale
  `agag-status.json` files surfacing where humans/agents already look
  (nctl / Nautobot). Separate episode in pj-clusterintent territory.
- Merging the DM `serve` thread and `sweep_serve` into one event queue
  (agforge, cagent) — negligible steady-state win after step 2.
- `update_message` events (topic renames / un-resolves) as sweep triggers —
  today's loop misses them too; parity kept.
- A recurring janitor that unsubscribes bots from long-dead channels; step 3
  does it once by hand and names the teardown rule instead.
