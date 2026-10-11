# p5/ex1 step 1 — baseline

Plan section: [plan.md](plan.md) §1.

## Repositories and instructions

| Repository | State at start (2026-10-11 17:52 JST) |
| --- | --- |
| agdevworld | `32498e9` (request-form fix), one commit ahead of origin: deployed but not pushed. Clean tree |
| agautolab | `daeaada`, even with origin. Clean tree |
| pj-agdev | `cfed399`; the agdevworld gitlink is behind (`2f0873c` recorded, `32498e9` checked out) |
| devdocs | the p5 final [report.md](../report.md) and a `report5.md` addendum, written by the reviewing session at 17:46, uncommitted. Committed unchanged with this step as the parent record. Unrelated untracked files (`agent_guide/p3/ex2`, `workflow_editor/p2`, `p3` braindumps) left alone |

Applicable instructions: the AG standard style (reports per step, English, no absolute local paths in tracked files), localrule (commit and push every change), the agdevworld release lifecycle in the host's `devenv.md`.

## Installed system

| Item | Value |
| --- | --- |
| Editor release | `32498e9` (`service/current`), `previous` `2f0873c`; service and `bin/wfe` agree |
| Executor / notifier | `com.agdev.agautolab-wfexec` running agautolab `daeaada`; notifier thread enabled by `[notify] enabled = true` |
| Execution activity | executor idle; no open attempts; no queued jobs |
| Notifications | targets #29, #273, #274 provisioned; `improve-game/run-002` 0 pending, 0 failed; `run-001` not announced; `inflight.json` absent |
| Long-lived connections | the service (`node`) holds `exec.sqlite` open; nothing holds `notify.sqlite` (the CLI opens and closes it per call, so it has no WAL file between calls) |

## Baseline checks (unchanged code)

| Check | Result |
| --- | --- |
| Editor `npm run check` | type check, 129/129 tests, build: pass |
| autolab `pytest tests/test_wfnotify.py tests/test_wfexec.py` | 10/10 pass, three consecutive runs (≈61 s each). The lock failure seen in the p5 review did not recur in these three runs |

## Defect 1 — a growing retry batch is posted again (reproduced)

A temporary area, the real `wfe` CLI, the real `agautolab.wfnotify` transport, and the fake Zulip of `test_wfnotify.py`. The reproduction is a scratch test built from the existing fixtures:

1. `#2:started` and `#4:question` are delivered normally.
2. The person answers q1. The next post (`ref docs/run-001 #6:answered`) reaches Zulip but its answer is lost; the row is `failed`, `uncertain = 1`.
3. Before the retry, attempt 2 runs to completion and records more events.
4. The retry's `due()` rebuilds the batch: marker `ref docs/run-001 #6:answered #13:reported`.
5. The transport searches for that marker, does not find it, and posts.

Topic markers afterwards:

```text
ref docs/run-001 #2:started
ref docs/run-001 #4:question
ref docs/run-001 #6:answered
ref docs/run-001 #6:answered #13:reported
ref docs/run-001 #15:completed
```

`#6:answered` is posted twice. Observed cause: `due()` re-selects the batch on every call and the marker is derived from the batch, while reconciliation compares the whole marker. Nothing records which events the lost post carried, except `inflight.json`, and that is compared by the same changed id list.

## Defect 2 — SQLite lock at store open (reproduced)

A scratch Node script opens a store with the package's own constructor, reads one row, writes one short row and closes it; it instruments `exec()` to name the failing statement. 8 independent processes start together, each opening 5 times, over 20 rounds (800 opens per case).

| Case | Opens | Failures | Failing statement | Codes |
| --- | --- | --- | --- | --- |
| `ExecStore`, fresh database | 800 | 106 (13 %) | `PRAGMA journal_mode = WAL; PRAGMA busy_timeout = 5000;` | 99 × `SQLITE_BUSY` (5), 7 × `SQLITE_BUSY_RECOVERY` (261) |
| `NotifyStore`, fresh database | 800 | 129 (16 %) | same | 122 × 5, 7 × 261 |
| `ExecStore`, existing WAL database | 800 | 23 (3 %) | same | 13 × 5, 10 × 261 |
| `NotifyStore`, existing WAL database | 800 | 36 (4.5 %) | same | 26 × 5, 10 × 261 |

Every failure is in the first statement, before any busy timeout is in force; no failure occurred in the schema statements, the read or the write. This matches the p5 review's observation (`database is locked` opening `ExecStore`).

What is observed and what is suspected:

- **Observed:** the journal-mode pragma runs with no busy handler, so it fails at once whenever another connection holds the lock it needs. This happens both on first creation and on an existing WAL database.
- **Suspected, not proven:** the 261 codes are opens that meet another process's WAL recovery (the first opener after the last close rebuilds the WAL index). The plain 5 codes are opens that meet a closing connection's checkpoint and WAL removal, or a concurrent journal-mode change. In the tests and on the live host, every `wfe` call is a process that opens and closes both stores, and `notify.sqlite` has no long-lived holder; so these windows recur constantly.
- Not established by this step: whether the busy timeout covers every one of these cases. `SQLITE_BUSY_RECOVERY` is not always retried by SQLite's busy handler. Step 4 tests this rather than assuming that reordering the pragmas is enough.

## Run-header backend (observed)

`improve-game/run-002` (read-only): `run.json` records `executor: {name: autolab, backend: null}`; its three attempts record `claude_code / anthropic/claude-sonnet-5`. The header renders `rec.executor.backend ?? 'unknown'` (`src/runView.ts`), so it says "backend unknown". The attempt list in the same view already shows each attempt's backend from the execution API.

## Starting points confirmed

- `server/notify.ts`: `due()` rebuilds the batch per call; `Delivery` = ids, content, marker, `uncertain`; `render()` writes `unpublished` onto the batch head as a side effect of rendering.
- `agautolab/wfnotify.py`: reconciliation searches the last 100 topic messages for the complete marker; `inflight.json` holds one in-flight send for the whole transport.
- Both store constructors run `PRAGMA journal_mode = WAL` before `PRAGMA busy_timeout`, then `CREATE TABLE IF NOT EXISTS` outside any transaction.
- `src/runView.ts`: header reads the recorded executor backend; attempts carry the actual one.

## Next

Step 2: the durable delivery record and its claim protocol in the editor and the transport.
