# p5/ex1 step 6 — verification

Plan section: [plan.md](plan.md) §6.

Final tested commits: agdevworld `78caec2`, agautolab `ca8e807` (both pushed).

Step 6 added three things:

- A roll-forward guard: a frozen delivery whose events were all recorded as sent by something else (the old release after a rollback) is closed with that message id and never resent. A partial overlap becomes `unresolved` for a person.
- The publication-during-retry test.
- A rollback test.

The test harness for the lock holder had a race: an exit event could be missed and the test would wait forever. It was fixed (`78caec2`); this was a test defect, not a store defect.

Everything ran in temporary registries and databases against a fake Zulip transport, with deterministic fault injection: lost answers, `on_post` hooks, held locks, unreadable channels, failing searches, live and killed owner processes, and the released p5 code from Git history. Nothing was posted to Zulip.

## The matrix

| Scenario | Evidence (test) | Result |
| --- | --- | --- |
| POST succeeds, response lost, event B after A | `test_growing_batch_is_not_posted_twice` | One post with A (`#6:answered`), acknowledged by its original id and content; B (`#13`, `#15`) after it, in order |
| Restart between POST and acknowledgment | `test_restart_between_post_and_acknowledgment` | Frozen delivery found; acknowledged; no repost |
| Restart after claim, before POST | `test_restart_after_claim_before_post`, `test_restart_after_begin_before_post` | Same delivery; begun → absence established → exact payload once |
| Acknowledgment cannot get the database lock | `test_acknowledgment_without_the_database_lock` | One POST; delivery stays `sending`; found on the next pass |
| New events or publication during a retry | growing-batch test; editor `a published checkpoint answers earlier unpublished details…` | The uncertain question keeps its frozen text ("not yet published") and goes first. The publication notice waits behind it. Then the `cp1:published` follow-up announces the plan link. Nothing is rewritten |
| Two notifier processes overlap | `test_two_notifiers_only_the_owner_sends` | B is refused while A's process lives. After A is killed, B reconciles and sends once. A's stale result is refused |
| Marker older than the latest 100 messages | `test_marker_older_than_the_last_hundred_messages`, `…_cannot_reach_far_enough…` | Found by paging. Where the history cannot be covered, it stays uncertain with no repost, and a problem is shown |
| Topic moves, search fails, access disappears | `test_moved_topic…`, `test_search_failure_is_not_absence`, `test_lost_access_and_reset…` | No false absence. The relocation is recorded. A reset keeps the uncertainty, and the person's `resolve` is the path |
| Legacy database migration | `test_legacy_state_is_migrated_and_reconciled` (state made by the released p5 editor + transport); editor `migration: …` | Sent evidence and bindings survive; pending work resumes; posted and never-posted legacy events are told apart by evidence; inconsistent states are diagnosed; a newer schema is refused |
| Rollback and forward again | `test_rollback_to_the_old_release_and_forward_again` | The released p5 CLI and transport open the migrated store and send only what was unsent. The new release then closes the frozen delivery as "sent outside this delivery" without reposting, and delivery continues |
| Concurrent first open and reopen of both stores | editor `test/sqlite.test.ts` and the open-stress harness | No failures; no lost rows; integrity ok |
| Controlled short and prolonged write locks | `test/sqlite.test.ts`; `test_prolonged_store_lock_fails_ticks_without_doubling_attempts` | Short: waited out. Prolonged: bounded (≈ 5 s) and named, and a later attempt succeeds. The executor keeps job/attempt fencing; the notifier recovers |
| Backend display | `test/backend.test.ts`, `checks/backend.ts` | Active, latest, multiple backends, missing and none all match the recorded evidence |
| Original long outage | `test_long_outage_keeps_work_and_delivers_in_order` (kept) | The run completes during the outage; afterwards all events are delivered in order, none twice |

## Suites

| Suite | Result |
| --- | --- |
| Editor `npm run check` | type check, 135/135 tests, build: pass |
| autolab `pytest tests/test_wfnotify.py tests/test_wfexec.py` | 24/24 (19 notifier, 5 executor) |
| Browser checks touching the changed views (private services) | `checks/exec.ts` all passed; `checks/docsteps.ts` all passed; `checks/backend.ts` all passed |

The older browser checks with failures recorded before p5 ([../report3.md](../report3.md)) were not touched by this change and were not rerun. They remain unrelated baseline items.

## Repetition (deliberate reliability check)

| Repetition | Count | Result |
| --- | --- | --- |
| `node --test test/sqlite.test.ts` (4 multi-process contention tests) | 20 runs | 20 × 4/4 pass |
| Open-stress harness (8 processes × 5 opens × 20 rounds; fresh and existing WAL; both stores) | 3,200 opens | 3,200 ok, 0 failures (baseline: 294 failures in the same 3,200) |
| autolab `pytest tests/test_wfnotify.py tests/test_wfexec.py` (combined, where the lock error was first seen) | 5 runs | 5 × 24/24 pass (≈ 3 min 35 s each) |

Before step 4 the combined autolab suite failed on the lock in 1 of 3 runs ([report3](report3.md)); after it, 0 of 5. During development one contention-test run hung. The cause was a race in the test's lock holder (an exit event could be missed), fixed in `78caec2`; the 20 runs above are after that fix.

These are finite repetitions, not a proof against all races.
