# improve_wfrun p3 step 4: recoverable run-branch cleanup

Date: 2026-11-02. Code: agdevworld `2951d02` (`server/branchCleanup.ts`, `cli/wfe.ts`, `server/api.ts`, `test/branchCleanup.test.ts`). Contract: `workflow_editor/docs/storage.md` *Run branch cleanup*. Deployed and tried live in step 5.

## What exists now

| Surface | Offers |
| --- | --- |
| CLI | `wfe branches inventory`, `preview [--selection-out]`, `delete --selection [--dry-run]`, `recreate [--dry-run]` |
| HTTP | `GET …/runs/<wf>/<run>/branches`, `POST …/branches/delete`, `POST …/branches/recreate` |
| Record | `branch-cleanup.json` beside `copy.json`, in the backup; `wfe backup rehearse` checks each deleted tip is recoverable |

Both surfaces use one function (`assess`). No scheduler exists, and no run view control was added: this is an operator's action for now.

## Decisions taken during implementation

- **Eligibility is checked on the exact remote tip, three ways.** Fully present in the retention repository, reached by this run's own retained refs (not merely by another run's), and reached by what the backup holds of this run. A tip pushed after retirement fails the first and stays.
- **Coverage is read from the destination's `LATEST` each time**, not from local status, so a cleanup cannot rest on a generation that was removed. Unreachable destination means nothing can go.
- **The compare-and-delete is Git's own**: `git push --force-with-lease=<ref>:<object> <remote> :<ref>`. It is atomic on the server, for Gitea and for file remotes alike, so no window remains between the recheck and the deletion.
- **A nested repository's pin is a dependency of the project, not of the run.** If the parent's default branch pins a commit reachable only through the branch going, the branch stays, because a later collection on Gitea would break cloning the project. This was not in the plan's list by name; it is its "dependent operation requiring the branch". The check fetches into a small cache under `<area>/.local/branch-cleanup/cache/`. A check that cannot be made counts as a reason to stay.
- **Only the configured Gitea.** Six repositories in copies' manifests are on GitHub (pj-clusterintent's submodules). Their rows are `outside` and can never be selected, so runs with them can still have their Gitea branches cleaned.
- **Unknown ownership is reported, not selected.** A `wfe/<project>/…` branch with no copy here, such as one from before p9, is listed by the inventory only.

## What did not need changing

Each was checked by reading the code and by a fixture, because a deleted branch could have broken it:

| Consumer | Why it holds |
| --- | --- |
| Run numbers (`usedRuns`) | also counts copy folders, which stay |
| Receipts (`copyByReceipt`) | read from `copy.json` |
| Record and history reads | the durable record and retention, not the branch |
| Restoration | rebuilt from retention, never from the remote |
| Next publication (`publishRun`) | a plain non-forced push, which simply creates the branch |
| Worker dispatch | publishes first, then the worker fetches what was published |

## Checks actually run

`test/branchCleanup.test.ts`, 8 tests:

| Case | Fixture evidence |
| --- | --- |
| Eligibility | not retired, no backup, older generation than the backup, unsettled worker, tip pushed after retirement |
| Dependencies | a pin reachable only through the branch keeps that branch; the root's can still go |
| Inventory | nested repositories, and a stray branch reported as unknown |
| Recreation | exact objects in both repositories; dry runs change nothing; other runs' branches and main untouched |
| Ref race | moved between recheck and push: the lease refuses and records `changed`; a stale selection on a moved branch leaves it |
| Partial failure | one remote gone: the other proceeds, the failure is journaled with its object, and the same selection continues |
| Identity after deletion | receipt replay, number not reused, record sequence unchanged, restoration at the saved commit, a publication recreating and advancing the branch |
| Cancelled unintegrated | not on main, deleted, recreated at its object; the mapping is in the next backup |

Not covered by a fixture: a held or standing run through `assess` directly (it uses the same `recordReasons` retirement does, which has its own tests), and a worker-backed run, which step 5 tries live.

## Limits

- Recreating a branch after a later publication reports a conflict rather than guessing. That is intended.
- A cleanup record entry is durable as soon as it is written, but reaches the backup at the next generation. If the coordinator were lost in between, the retained refs in the backup still hold the objects; only the record of what was deleted would be missing.
- Bytes are not addressed here: see step 5.
