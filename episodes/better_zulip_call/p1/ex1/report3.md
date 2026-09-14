# better_zulip_call p1 ex1 — step 3: failed verification is not change

Date: 2026-09-14.

## What changed

`Closer._revalidate()` now returns three separate facts:

- `ok`: whether every required scoped listing was read;
- `changed`: conversations whose successful listing read changed the mirror;
- `failures`: the affected channel and exception reason for every unsuccessful
  read.

If any required read fails, `close()` returns before `_apply()`. The payload
keeps the approved preview and board data, sets `verification_failed: true`,
`retryable: true`, and `applied: false`, and names the unreadable channels.
The remembered plan is not consumed and no operation-history row is written.
A retry performs every scoped listing again.

This is distinct from a changed-plan refusal: verification failure says the
relay could not establish the facts; refusal says it did establish them and
the action plan differs from what the human approved.

The shared completion UI now preserves both non-writing responses as preview
states. A verification failure has its own title, summary, reason styling, and
`retry verification` action. Its HTTP 503 payload is accepted as a useful plan
rather than collapsed into an unreadable relay error. The Front Desk Phaser
panel and the DOM completion panel use the same view model.

## Verification

Focused regressions:

```text
cd pj-agdev/agdevworld/agentroom
uv run pytest -q tests/test_close_plan.py
```

Result: **14 passed**. The cases include:

- all scoped listings fail: zero writes;
- one scoped listing fails while the others succeed: zero writes;
- an injected `RateLimited` (HTTP 429 class): retryable and zero writes;
- retry after recovery: all checks run again and completion succeeds;
- work changes while verification is unavailable: the recovered retry finds
  it and refuses the old approval;
- the existing partial-write failure/retry behavior remains unchanged.

Existing completion and HTTP scope tests:

```text
uv run pytest -q tests/test_close.py tests/test_close_plan.py tests/test_scope.py
```

Result: **93 passed**.

Frontend validation:

```text
cd pj-agdev/agdevworld
npm run build
```

Result: TypeScript and Vite build succeeded. Vite emitted only the existing
Phaser bundle-size advisory.

During the broader suite, the first coherence implementation exposed an
infinite retry for legacy fixtures that intentionally construct a `Closer`
without a mirror. That path has no concurrent mirror ingestion to stabilize;
it now keeps its established behavior of deriving once from the supplied
realm. The 93-test run above is after that correction.
