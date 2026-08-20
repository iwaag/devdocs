# agent_standardize p1 — Step 3 report: the own channel is the entrance

AI-generated (Omni Agent, 2026-08-20).

## What changed

`agforge-agstudio1` is now special to its listener: every unresolved topic in
that channel is an input to the instance. Other subscribed channels retain
their existing, deliberately narrow routes — only `create-` and `runcreate-`
topics are eligible there.

The routing rule lives in `agforge.zulip_listener.topic_filter()` and takes
the channel name from the same local identity seed added in Step 1. It is:

```
own channel (`agforge-agstudio1`)    -> every topic
other subscribed channels            -> `create-` / `runcreate-` only
```

This preserves the existing generation behavior exactly: `create-` still
goes to `create_topic.handle_topic`; `runcreate-` still goes to
`runcreate_topic.handle_runcreate`. A plain topic in the own channel receives
the p1 canned answer, identifying the instance as an asset-generation agent
and directing the requester to open a `create-…` topic there. Because the
shared sweep continues to skip a topic whose last post is the bot's own, that
answer is also the loop guard: it silences the topic until someone else posts
again.

## Shared-library change and delivery chain

The existing `agag.zulip.sweep_serve` only accepted a prefix (or prefix
tuple). `pyagag` now also accepts a callable `(channel, topic) -> bool`;
both the startup/re-registration sweep and the event path use the same
callable. This is what makes the policy durable across listener downtime,
rather than treating a live event differently from a startup sweep.

Required delivery chain completed:

1. `pyagag` commit `c33d1ee` — *Let Zulip sweeps use channel-aware topic
   filters* — was pushed to GitHub `main`.
2. agforge ran `uv lock --upgrade-package pyagag`; its lock moved from
   `97d2f8dc` to `c33d1eec` (the pushed GitHub revision).
3. agforge commit `3939f26` — *agent_standardize p1 step 3: make the own
   channel an entrance* — was pushed, and `pj-agdev` now records it in
   `7b02aac`.

No local path dependency or local Git remote was introduced.

## Verification

- `pyagag`: `uv run pytest -q` passed before its commit.
- `agforge`: `uv run pytest -q` passed: **187 passed**.
- Focused tests cover the callable in both sweep paths, the own-channel vs
  foreign-channel selection, and the canned plain-topic reply.
- The launchd listener was reloaded with
  `launchctl kickstart -k gui/$(id -u)/com.agdev.agforge-zulip`. Its fresh log
  records the new rule and a clean startup sweep:

  ```text
  agforge zulip listener starting (pull sweep: all topics in 'agforge-agstudio1', prefixes ('runcreate-', 'create-') elsewhere + DM thread)
  full sweep: 0 awaiting, 12 calls spent, 986 left in the window
  ```

- Before the reload, `nctl status --json` reported an authenticated,
  reachable Nautobot, a live worker, and no pending jobs (`ok: true`).

## Not done in this step

- The response text is intentionally a code-level p1 placeholder. Step 4
  moves the self-description into a committed Markdown file and adds the
  intro-post CLI.
- No human/Omni test message has been sent yet; the paid generation and plain
  entrance smoke belong to Step 5.
- The existing DM route and `create-` handling in `#general` / `pj-*` remain
  available. Re-pointing agfront is explicitly deferred to the next phase.

Changed the listener behavior that an in-system agforge instance could
eventually maintain itself — handoff candidate.

