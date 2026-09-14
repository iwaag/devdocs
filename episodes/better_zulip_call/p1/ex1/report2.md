# better_zulip_call p1 ex1 — step 2: invalidate changed evidence

Date: 2026-09-14.

## What changed

The remembered completion discovery now uses the mirror's change feed in
addition to topic names, last ids, and resolved flags.

- Any edit or deletion in a reached or excluded conversation invalidates its
  remembered evidence, including an older relationship/state note whose
  removal leaves the topic's maximum message id unchanged.
- A move invalidates both its source and destination conversation when either
  is a dependency. A relevant channel change and a deep-resync boundary also
  force rediscovery.
- A pruned or truncated change feed is treated as lost coverage and forces
  rediscovery. It is never read as proof that nothing changed.
- An edit in an unrelated conversation, including another topic in the realm,
  retains the warm discovery. Existing listing/evidence tuple checks continue
  to catch new posts, new topics, moves, resolves, and changed last ids.

Discovery also detects changes that overlap its graph walk and retries when
they affect the evidence it just assembled. The `Plan` keeps the revision from
before that coherent walk. It deliberately does not attach a newer unrelated
revision to an older view; later feed entries remain available to
`_unchanged()`.

No remote history read was added. This is entirely a local mirror/feed check,
so an unchanged warm preview still costs zero Zulip calls.

## Verification

Command:

```text
cd pj-agdev/agdevworld/agentroom
uv run pytest -q tests/test_close_plan.py -k 'not failed_pre_write'
```

Result: **9 passed, 1 deselected**.

The focused coverage proves:

- the received `completed` → `open` edit refuses the old approval with zero
  writes;
- deleting the older `completed` note invalidates the discovery while a later
  message keeps the topic maximum id unchanged;
- an unrelated conversation edit reuses the remembered walk
  (`discoveries == 1` through apply);
- lost change-feed coverage rebuilds discovery even when the resulting action
  fingerprint is unchanged;
- the existing warm-preview, event-lag listing, partial retry, and unrelated
  growth cases still pass.

The one deselected test is Step 3's intentional failure for an unavailable
pre-write listing.
