# improve_wfrun p2 step 3: inventory, retirement and restoration

Date: 2026-11-02. Code: agdevworld `041c7a7` (`server/retention.ts`, `server/tarball.ts`, `cli/run.ts`, `cli/wfe.ts`,
`cli/remote.ts`, `server/workerApi.ts`, `src/runSide.ts`, `src/runView.ts`). Contract: `docs/storage.md` *Retirement*. Verified in
fixtures here; deployed trials are in step 5.

## What exists now

| Surface | Offers |
| --- | --- |
| CLI | `wfe runs storage`, `wfe run storage <run>`, `wfe run retire <run> [--dry-run] [--discard <path>=<reason>]`, `wfe run restore <run> [--dry-run]`, `wfe runs retire --selection <file>` |
| HTTP | `GET …/runs/<wf>/<run>/storage`, `POST …/storage/retire|restore` (dry run answers; the operation starts apart) |
| Run view | Simple: whether the work is retained, and Restore. Detail: Retire, reasons, sizes, saved commits, per-worker progress, history, last failure |
| Worker | tasks on the heartbeat, `wfe exec storage-tick`, archive upload, removal, files back on a new dispatch; no executor change |

No scheduler exists. Bulk retirement takes a selection and rechecks each run.

## Decisions taken during implementation

- **Retention repository per project repository**, not per-run bundles: a bundle per copy would repeat each copy's history. The trade-off is a
  shared store, whose damage would affect every retired run of that repository; restoration reports it as unavailable.
- **Archive everything not committed by default.** Nothing is discarded by name; `--discard` needs a reason from the agent asking. Archiving
  `.godot` or `.venv` therefore costs bytes until an agent says otherwise.
- **An acceptance of a retired run is refused** with the instruction to restore, because an acceptance binds to the files. A rejection
  and later record writes are accepted; the next publication after a restoration snapshots them.

## Defects the fixtures found in this code

Both were in my first version, and both would have lost data on a real copy:

1. **Staged-only bytes were not retained.** A blob in the index is reachable from no ref, so the restored index pointed at a missing object. The
   index's own tree is now retained (`refs/wfe/retain-index`).
2. **Git does not report sockets and pipes**, so a FIFO was neither inventoried nor refused. A walk of the tree now finds them and they are classified like anything else.

## Checks actually run

`test/retention.test.ts`, 10 tests, and one in `test/workers.test.ts`, against the plan's acceptance table:

| Case | Fixture evidence |
| --- | --- |
| Durable records | run view, files, `?rev=`, observation, late records on a retired run, over HTTP and through tools |
| Eligibility | not started, queued job, a process in the tree, legacy; presentations and held runs are in code, **not exercised by a fixture** |
| Local data | modified, staged, deleted, untracked, ignored, modes, links in and out of the copy, empty folders, `.local`; nested submodule commit no remote has |
| Worker data | worker-only file and local commit archived first, lost contact holds at `retiring`, stale upload and report refused, file back on re-dispatch |
| Interruption | failed commit retention, leftover staging, crash between commit and release, crash during restoration, abandoned retirement |
| Corruption | truncated and missing archive, missing retention repository, a tar that writes garbage |
| Restoration | saved code despite main advancing, no job, latest record, publication works afterwards, a second retirement |
| Identity | replayed receipt finds the retired run; its number is not reused |
| Continuation | claim during retirement, a job winning at the last check, a job on a retired run requesting restoration, frozen definition kept |

`Shared outputs` is implemented (a verified output's matching bytes are not archived) but has **no fixture**; step 5 must check it or say it did not.
Session replacement after a restoration follows the existing harness code and is not separately tested.

## Limits

- A worker's bundle returns as a file, not as refs.
- An archive stays after restoration until the next retirement replaces it, so a restored run costs its archive too.
- The run view was type-checked and built; it has not been opened yet (step 5).
