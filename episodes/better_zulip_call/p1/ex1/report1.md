# better_zulip_call p1 ex1 — step 1: reproduce both completion defects

Date: 2026-09-14.

## Added regressions

`agdevworld/agentroom/tests/test_close_plan.py` now contains two focused
regressions over the existing `chain()`, `door_over()`, and `FakeRealm`
fixture:

- a finished task is previewed, its existing `[selfnote][state] completed`
  message is edited to `open`, and the test waits on the mirror revision until
  that exact edited content is present before applying the old preview;
- a fresh preview is followed by a `ConnectionError` from every
  `mirror.refresh_listing()` call, then the old preview is applied.

Both tests require zero completion writes in the corrected implementation.
The first requires a changed-plan refusal with the mission now blocked. The
second requires an explicit, retryable verification failure rather than a
changed-plan refusal.

## Reproduction result

Command:

```text
cd pj-agdev/agdevworld/agentroom
uv run pytest -q tests/test_close_plan.py
```

Result before implementation: **2 failed, 5 passed**.

- The received-edit case returned a successful applied payload with no
  `refused` key. `_unchanged()` noticed the global mirror revision but compared
  only the live topic name, last message id, and resolved flag; none changed
  when the older state message's content changed, so it reused the stale
  discovery and wrote the acceptance/closure.
- The failed-listing case returned a successful applied payload with no
  `verification_failed` key. `_revalidate()` represented each exception as an
  `<unread: ConnectionError: ...>` changed topic, `close()` rediscovered from
  the unchanged mirror, obtained the approved fingerprint again, and applied
  the stale plan.

The fixtures synchronize on the mirror revision/content and use an injected
exception. They do not depend on sleeps, live Zulip writes, or rate-limit
exhaustion.
