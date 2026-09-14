# better_zulip_call p1 ex1 — step 4: validation, deployment, and documentation

Date: 2026-09-14.

## Final validation

Completion and mirror-facing relay suites:

```text
cd pj-agdev/agdevworld/agentroom
uv run pytest -q \
  tests/test_close.py tests/test_close_plan.py tests/test_closing.py \
  tests/test_scope.py tests/test_room.py tests/test_frontdesk.py
```

Result: **154 passed**.

Mirror suite:

```text
cd pyagag
uv run pytest -q tests/test_mirror.py
```

Result: **13 passed**. No pyagag source or dependency lock changed in this
exercise; the existing change feed was sufficient.

Frontend:

```text
cd pj-agdev/agdevworld
npm run build
```

Result: TypeScript and Vite build succeeded. The only advisory was the
existing Phaser chunk-size warning.

The unchanged warm completion fixture still proves one discovery, zero
preview calls, and exactly one scoped listing per dependency channel before
apply (**3 calls** in that fixture), followed only by the required writes and
checks. There is no realm-wide reload and no extra preview walk.

## Pre-deployment and deployment

The required read-only cluster checks were made through `nctl` before touching
services. Connectivity, authentication, worker health, repository state, and
the relevant desired/actual comparison were healthy.

The relay was restarted through its existing service manager and resumed its
persisted event queue. The web image was rebuilt and its container recreated.
No dependency lock or shared service needed an update.

Read-only smoke evidence after the final restart:

- relay health was `ok` and its mirror was live by queue resume;
- a preview of an existing resolved request returned immediately with schema
  `ag.completion.v1`, two already-done actions, and zero walk Zulip calls;
- the web endpoint returned HTTP 200;
- the served completion bundle contains the distinct `verification failed`
  and `retry verification` UI strings.

The edit and failed-read fault scenarios remain fixture-only; no live Zulip
record was mutated to reproduce them.

## Live-smoke correction

The first post-deployment preview exposed an overlap edge case: excluded
conversations had been remembered with a synthetic last id of zero. The new
coherence loop therefore kept seeing their real id as a mismatch. Excluded
rows now retain their actual last post id, and discovery overlap checks compare
feed/listing movement without treating the evidence just assembled as a
pre-existing snapshot. A focused regression proves that editing an excluded
conversation invalidates it; the final 154-test run and second live preview
are after this correction.

## Documentation correction

The p1 phase report now separates two facts accurately:

- ignoring a relevant edit/deletion that the mirror had already received was
  a cache invalidation defect and is fixed here;
- an older edit/deletion whose event has not arrived and whose topic listing
  is unchanged remains outside the listing-only pre-write check.

It also records why p1 chose per-process mirrors: simple independent
deployment and credential isolation, traded for duplicate initial history
reads and duplicate event intake. A shared collector remains outside scope.
