# p5/ex1 step 4 — recoverable SQLite initialization and contention

Plan section: [plan.md](plan.md) §4.

Commits: agdevworld `d98efc2` (helper, both stores, CLI reporting, multi-process tests) and `78caec2` (a race in the test's lock holder fixed); agautolab `05bb549` (executor test). The helper `server/sqlite.ts` was introduced in `501d0fa`, because the notify migration needed it. All pushed; deployed in step 7.

## What was wrong (from step 1)

Each store ran `PRAGMA journal_mode = WAL; PRAGMA busy_timeout = 5000;` as its first statement. So the WAL pragma ran with no busy handler, and in 8-process opening bursts 3–16 % of opens failed at once with `SQLITE_BUSY` (5) or `SQLITE_BUSY_RECOVERY` (261) ([report1](report1.md)). The same failure then appeared naturally in the combined autolab suite during step 3: `wfe exec heartbeat` died at `ExecStore`'s WAL pragma with code 261.

## Change: `server/sqlite.ts`, used by both stores

1. `openStore(store, file, migrations)` opens with `{ timeout: 5000 }` and sets `PRAGMA busy_timeout` before any statement that takes a lock.
2. It then sets WAL, reads `user_version`, and applies each missing migration in its own `BEGIN IMMEDIATE` transaction that re-reads the version inside. Concurrent first opens therefore apply a migration once. A version newer than the release is refused.
3. These open steps are retried **only on lock contention** (`errcode & 0xff` ∈ {5, 6}). Backoff starts at 10 ms and doubles up to 250 ms, with jitter, within a 15 s deadline. Any other error propagates at once. A pragma, or a transaction that rolled back, leaves nothing behind, so retrying them is safe.
4. On failure the connection is closed and a `StoreError` is thrown, naming the store, the operation and the SQLite code (for example `notify store: schema version failed (SQLite code …)`). It never includes paths' contents, credentials or run text.
5. `transaction(db, f, store)` is the one way the stores open a write transaction (`BEGIN IMMEDIATE`, rollback on any error). It is **not** retried: whether repeating it is safe is for the caller to know. Its SQLite failures, and those of every autocommit statement and read through the stores' helpers (`named()`), are reported the same way ("exec store: a write failed (SQLite code 5, database is locked)").
6. `wfe` prints a `StoreError` as one refusal line (exit 1), not a stack trace.

`ExecStore` now opens through `openStore`. Its schema version 1 is the existing tables, with `IF NOT EXISTS`, so the live database is adopted as it is. `NotifyStore` has versions 1 and 2 (step 3).

There is no generic retry around agent execution, publication or a Zulip POST. A POST whose acknowledgment could not be written is reconciled, not reposted ([report2](report2.md), `test_acknowledgment_without_the_database_lock`).

## Does the order alone fix it? (measured, not assumed)

Each case below is 8 independent processes per round, starting together; 5 opens each, plus a read and a short write; 20 rounds; 800 opens per case.

| Variant | `exec` fresh | `notify` fresh | `exec` existing WAL | `notify` existing WAL |
| --- | --- | --- | --- | --- |
| Before (WAL, then busy timeout) | 106 failed | 129 failed | 23 failed | 36 failed |
| Busy timeout first, **no** retry (temporary local variant, not committed) | 5 failed | 6 failed | 0 | 0 |
| Committed (busy timeout first + bounded retry of open steps) | 0 | 0 | 0 | 0 |

Every remaining failure of the no-retry variant was the WAL switch on a freshly created file, `SQLITE_BUSY` returned without the busy handler. The order fixes reopening an existing database. Creating one also needs the bounded retry.

## Verification

### Editor: `test/sqlite.test.ts`, independent processes

| Test | What it does | Result |
| --- | --- | --- |
| Concurrent first open and reopen (`exec`, `notify`) | 4 rounds × 2 phases. In each phase, 8 child processes are released together by a file barrier; each opens the store with the package's constructor and writes 3 rows through `transaction()`. | No failures. All 48 rows per round present. `integrity_check` ok. Journal mode `wal` |
| Short write lock (`exec`, `notify`) | Another process holds `BEGIN IMMEDIATE` for about 1 s while a child writes | The child waits (≥ 0.8 s) and succeeds |
| Prolonged write lock | The lock is held until the child gives up | It fails in 4.5–9 s (the busy timeout), with `"<store> store: taking the write lock failed (SQLite code 5, database is locked)"` and code 5 |
| A later attempt | The lock is released and the same write runs again | It succeeds |

### autolab

| Test | What it checks | Result |
| --- | --- | --- |
| `test_prolonged_store_lock_fails_ticks_without_doubling_attempts` | `exec.sqlite`'s write lock is held past the busy timeout during the tick that would claim, and again during the tick that records the attempt's end | Each tick fails within the bound, naming the exec store and code 5. Nothing is claimed while locked. Afterwards there is exactly one attempt, launched once, with one `attempt.end` and exit 0, and the executor is idle. Job/attempt fencing holds |
| `test_acknowledgment_without_the_database_lock` (step 2) | `notify.sqlite` is locked through the acknowledgment | The notifier fails the pass without reposting and recovers on the next pass by reconciliation |

### Repetition (final code, `78caec2` / `ca8e807`)

| Repetition | Count | Result |
| --- | --- | --- |
| `node --test test/sqlite.test.ts` (4 multi-process contention tests) | 20 runs | 20 × 4/4 pass |
| Open-stress harness (8 processes × 5 opens × 20 rounds; fresh and existing WAL; both stores) | 3,200 opens | 3,200 ok, 0 failures (baseline: 294 failures in the same 3,200) |
| autolab `pytest tests/test_wfnotify.py tests/test_wfexec.py` (combined, where the lock error was first seen) | 5 runs | 5 × 24/24 pass (≈ 3 min 35 s each) |

Before step 4 the combined autolab suite failed on the lock in 1 of 3 runs ([report3](report3.md)); after it, 0 of 5. During development one contention-test run hung. The cause was a race in the test's lock holder (an exit event could be missed), fixed in `78caec2`; the 20 runs above are after that fix.

Finite repetitions do not prove the absence of races. They show that the failure rate measured before is gone at these counts.

## Limits

- The busy timeout (5 s) and open deadline (15 s) bound how long a caller waits. A writer that holds the lock longer makes the waiting operation fail with a named error. The executor and notifier simply retry on their next tick or pass.
- `SQLITE_BUSY_RECOVERY` outside the open path, in an ordinary statement right after another process's recovery, is left to SQLite's own handling. It was not observed in these runs.
