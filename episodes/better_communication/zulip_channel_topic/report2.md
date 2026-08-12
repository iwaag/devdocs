# Step 2 report — channel/topic mechanics in `agag.zulip`

Date: 2026-08-12. Result: `pyagag` carries the channel mechanics; agforge's
venv sees them; all tests pass (pyagag 16/16, agforge 68/68).

## What changed in `pyagag/src/agag/zulip.py`

- `ZulipClient.call()` now sends a form body for `PATCH` too (was POST/DELETE
  only) — resolve-topic needs it.
- New client methods, all stdlib, mirroring what Step 1 proved by curl:
  - `create_channel(name, description, principals, announce=False)` — the
    subscribe call that creates a public channel and subscribes everyone at
    once.
  - `send_to_channel(channel, topic, content) -> message_id`
  - `topic_history(channel, topic, num_before=50)` — `dm_history`'s channel
    analog (narrow: `channel` + `topic`, raw text, newest last).
  - `resolve_topic(message_id, topic)` — the ✔ rename with
    `propagate_mode=change_all`; no-op if the topic is already resolved.
    `RESOLVED_TOPIC_PREFIX = "✔ "` is exported.
- New helpers `is_channel_message_for_us(message, self_id)` and
  `channel_name(message)`.
- `serve()` gained an `accept` predicate parameter, **defaulting to
  `is_dm_for_us`** — existing listeners (cagent) keep their DM-only behavior
  without being touched; agforge passes a wider predicate in Step 3. This was
  the one place a deliberate compatibility choice was made, not out of
  obligation but because silently feeding stream events to cagent's DM
  handler would be a regression farmed for no lesson.

Tests added in `pyagag/tests/test_zulip.py`: the two new predicates, default
`accept` still ignoring channel messages, a wider `accept` seeing both types,
and `resolve_topic` idempotence (scripted client, no network).

## How agforge sees the change

`agforge/pyproject.toml`'s `tool.uv.sources` temporarily points `pyagag` at
the sibling checkout (`path = "../../pyagag", editable = true`) and the lock
was updated, because `listen.sh`/`serve.sh` run through `uv run`, which
re-syncs from the lock and would undo a bare editable install (the trap the
plan warned about).

**Debt, on purpose:** before this episode's changes are pushed, `pyagag` must
be pushed to GitHub first, then the git source restored in
`agforge/pyproject.toml` and `uv lock` re-run. The TEMP comment in the file
says exactly this.
