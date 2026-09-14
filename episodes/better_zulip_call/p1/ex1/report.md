# better_zulip_call p1 ex1 — completion evidence and failed verification

Date: 2026-09-14. Four steps completed and deployed. The step reports contain
the command-level evidence.

## Reproduced defects

Two focused tests first failed against the p1 implementation:

1. editing an existing task state from `completed` to `open`, then waiting
   until the mirror held that edit, did not invalidate the remembered plan;
   completion wrote acceptance notes and closed the request;
2. making every scoped pre-write listing raise `ConnectionError` was encoded
   as a changed topic, followed by rediscovery from the same stale mirror and
   successful writes when the fingerprint happened to match.

The baseline was **2 failed, 5 passed** in `test_close_plan.py`.

## Fix

Remembered discovery now combines its index tuple with the mirror change feed.
Relevant edits and deletions—including older messages whose topic maximum id
does not move—invalidate reached and excluded conversations. Moves invalidate
both ends, and resync or lost feed coverage forces rediscovery. Unrelated
conversation edits retain reuse. Discovery detects relevant overlap while it
walks and keeps the revision from before its coherent evidence view.

Pre-write revalidation now reports `ok`, concrete changes, and concrete read
failures separately. Any failed required read returns a retryable
`verification_failed` preview before the first write. Retry verifies again;
after recovery it either completes normally or refuses an old approval if the
work changed. Existing partial failures after writes begin retain their
idempotent retry behavior.

Both completion panels distinguish verification failure from changed-plan
refusal and completion. They retain the useful preview, state that nothing was
closed, show the reason, and offer `retry verification`.

## Validation and cost

- completion/closing/scope/room/frontend relay tests: **154 passed**;
- pyagag mirror tests: **13 passed**;
- frontend TypeScript/Vite build: passed (existing Phaser size advisory only);
- focused failure/recovery matrix: received edit, older deletion, excluded
  evidence, unrelated edit reuse, lost feed coverage, one/all failed listings,
  `RateLimited`, recovered retry, changed-during-failure retry, and existing
  partial-write retry all pass.

The hot path remains unchanged in remote cost: a warm preview uses zero Zulip
calls, and apply uses one topic listing per scoped dependency channel plus only
the necessary writes/checks. The representative fixture remains **3 scoped
listings**, one discovery, no realm-wide reload, and no extra preview read.

## Deployment

Read-only `nctl` checks were healthy before deployment. The relay was restarted
and resumed its persisted queue; the web image was rebuilt and recreated. A
read-only live preview returned at zero walk calls, the relay remained live,
and the served frontend carries the new failure/retry wording. No live fault
was injected and no dependency lock changed.

## Remaining boundary

An edit or deletion already received by the mirror is now handled correctly.
The remaining event-lag limitation is narrower: the remote pre-write check is
a topic listing, so it cannot reveal an older edit/deletion whose event has not
yet arrived when that topic's name and maximum id are unchanged. Full
transactional isolation from concurrent Zulip users is still not claimed.

Per-process mirrors remain intentional: they simplify independent deployment
and isolate credentials/quotas, at the cost of duplicate initial history reads
and duplicate event intake. A shared collector redesign is outside this fix.
