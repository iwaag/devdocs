# Better Zulip calls p1 ex1 — completion evidence and failed verification

## Goal and scope

Fix two completion defects reproduced during the p1 review: a plan survives a relevant edit already received by the mirror, and failed pre-write reads still allow completion from stale evidence. Keep p1's reduction in Zulip calls.

This remains a breaking change in a private experimental environment. Choose the implementation freely; compatibility adapters, security hardening, and full transactional isolation from concurrent Zulip users are unnecessary. Focus on these two defects, their regression tests, and an accurate report. Consolidating per-process mirrors into a shared collector is outside this fix.

## Step 1 — Reproduce both failures with focused tests

- Extend `pj-agdev/agdevworld/agentroom/tests/test_close_plan.py`, reusing `chain()`, `door_over()`, and `FakeRealm`.
- Preview the finished mission, edit its existing `[selfnote][state] completed` message to `open`, and wait until the mirror contains the edit. Apply the old preview. The current implementation writes `accepted` and `done` and archives the work channel; the corrected result must refuse the obsolete plan without writes.
- Preview again on a fresh fixture and make `mirror.refresh_listing()` raise `ConnectionError`. The current implementation records `<unread: ...>` as a change, rediscovers from the old mirror, and applies when the fingerprint matches. The corrected result must identify verification failure and leave completion unapplied.
- Use fixture events or explicit synchronization instead of depending on arbitrary sleeps. These cases need no live Zulip writes or deliberate quota exhaustion.

Verify: both tests expose the current defects for the intended reasons before implementation.

## Step 2 — Invalidate plans when their evidence changes

- Make relevant edits and deletions invalidate remembered discovery even when topic names, last message IDs, and resolved flags remain unchanged. Include relationship/state notes and excluded conversations whose evidence affected the decision.
- Choose a simple mechanism: per-conversation revisions, dependency content signatures, or inspection of the mirror's change feed are candidates. If using the feed, treat lost feed coverage as requiring rediscovery. Unrelated conversation changes should still allow reuse.
- Keep dependency evidence and its revision coherent while ingestion runs. Use a consistent snapshot or detect changes during discovery and retry; avoid attaching a newer revision to an older view of the evidence.
- Rebuild the plan locally when its evidence changes. Continue comparing the resulting decision against the approved preview. A harmless edit need not force another approval when the resulting actions are unchanged; an edit changing work state or closure scope must affect the decision.

Verify: the received-edit regression passes; deletion of an older state/reference message also invalidates discovery without requiring a changed topic maximum ID. Relevant changes are reflected, unrelated edits retain reuse, and unchanged warm previews add no Zulip calls.

## Step 3 — Represent verification failure separately from change

- Return explicit verification success/failure and the affected channels/reasons from `_revalidate()`. A failed read is neither evidence of no change nor evidence that local rediscovery has refreshed the missing facts.
- If a required pre-write check fails, leave that completion attempt unapplied and return a retryable result. Pausing the whole attempt before its first write is the simplest approach; partial execution across uncertain dependencies is unnecessary.
- Make the completion UI distinguish verification failure from a changed-plan refusal and successful completion. Retain the preview and useful board data, show the reason, and let the user retry after recovery. Adapt the response shape as needed; no compatibility layer is required.
- On retry, perform the required checks again and recompute affected evidence. Do not cache verification failure as success. Preserve the existing handling of partial failures that occur after actual writes begin.

Verify: one failed scoped listing and all failed listings both cause zero completion writes. An injected 429/read error follows the same rule. After recovery, retry can complete normally; if work changed while verification was unavailable, the old approval is refused. Existing partial-write retry tests still pass.

## Step 4 — Validate, deploy, and document

- Run the focused regressions and the existing completion/mirror tests relevant to the change. Run the frontend build if UI code changes. Confirm that unchanged completion still costs the scoped listings plus necessary writes/checks, with no realm-wide reload or extra preview reads.
- Before touching running services, inspect state through Nautobot or `pj-clusterintent/nctl`; use ignored environment notes for deployment commands. Update dependency locks and affected services if shared code changes. A read-only preview/UI smoke check is sufficient live evidence; the two fault scenarios belong in fixtures.
- Write `report.md` with the reproduced failures, fixes, test results, call-count comparison, and deployment status. Correct the p1 report's limitation: ignoring an edit already received by the mirror was a defect, distinct from an older edit whose event has not yet arrived and whose topic listing is unchanged. State any remaining event-lag limitation plainly.
- Add the rationale and tradeoff for p1's per-process mirrors to its report: they simplify deployment and isolate credentials, but duplicate initial history reads and event intake. No collector redesign is required here.
- Commit and push changed repositories and required submodule pointers. Keep credentials and machine-specific facts in ignored files.

## Implementation hints

- `agentroom/src/agentroom/close.py`: `_evidence_of()` stores only `(live_topic, last_post_id, resolved)`; `_unchanged()` checks that tuple after a global revision change. Receiving an edit advances the mirror revision but leaves this tuple unchanged, so the cached discovery survives.
- `_revalidate()` currently appends read exceptions to `changed`; `close()` responds by calling `_discover()` against the same mirror, then writes if the fingerprint matches. Handle verification failure before `_apply()` rather than trying to encode it into the fingerprint.
- `pyagag/src/agag/mirror/store.py` and `mirror/__init__.py` already maintain edits, deletions, notes, and a revision/change feed. Reuse these rather than adding remote history reads to every preview. Check move/deletion handling when deciding which conversation revision to update.
- The current fingerprint describes action keys/states, not evidence freshness. Keep approval comparison and evidence validity separate. Listing verification still cannot detect every unseen edit/deletion in an unchanged topic; this fix need not introduce a full history fetch or claim to close that race.
