# improve_wfrun p3 step 3: the live worker lifecycle

Date: 2026-11-02. Deployed WFE `a4a87c1`, executor `a9b82f8`, project `wfrun-trial`, worker `gpupc`, run `stamp/run-008`, check actor `improve-wfrun-p3-check`. Nothing here was attributed to the Developer. Commands and outputs are in local records (`pj-agdev/.local/improve-wfrun-p3/worker-trial/`).

## What happened, in order

| Step | Evidence |
| --- | --- |
| Run requested, performed on `gpupc`, integrated | `stamp/run-008`, seq 6. The agent found `stamps/plover.txt` already on main and verified it instead of writing it |
| Worker-only data, written by hand on the worker | `worker-only.dat`, `worker-only.bin` (200 KB), and side branch `p3-side` at `f7d884d`, in no remote |
| Retirement asked before the integration ended | refused: `integration 503 is queued` |
| Retirement again | `waiting`: the worker had not reported, the coordinator's tree stayed. Then `retired` |
| Worker archive | 200974 bytes, checksum accepted, holding both files and `.local/wfe-worker-bundles/root.bundle` |
| Worker copy | `remove-pending`, then `removed`; gone from the worker's disk |
| Backup | generation `20261102T054824Z-9c68` covered it |
| Restored on the coordinator | `ready`; branch at its saved `e618784` |
| Definition on main changed (`37f8fa6`), then `wfe run return` | job 1083, claimed by `gpupc` |
| Correction ran on the same worker | `40b12c8` adds `stamps/plover-p3.txt`; integrated to main |

## Verified on the worker after re-dispatch

| Item | Before retirement | After re-dispatch |
| --- | --- | --- |
| `worker-only.dat` sha256 | `6c66cd19…` | same |
| `worker-only.bin` sha256 | `1adc60d0…` | same |
| Saved code | branch from `69a9b54`, `e618784` on the coordinator | dispatch `c58132f` on top of `e618784` |
| Frozen definition sha256 | `3b5a5bb9…` | same, without the later text on main |
| `p3-side` | a ref at `f7d884d` | **not a ref**: `.local/wfe-worker-bundles/root.bundle` |

**Worker-only commits return as bundle files, not refs, and are not automatically reinstated.** Proof that they are recoverable: `git bundle verify` passes in the restored copy, and fetching `refs/heads/p3-side` from the bundle gave `f7d884d` with `side.txt` readable. That commit is in the bundle alone: the coordinator's retention repository does not have it (`cat-file` exits non-zero). Backup holds the bundle because it holds the worker archive.

The bundle has a prerequisite (`69a9b54`), so it reads only in a repository holding that commit. A restored copy does.

## Latest central record and synchronization

`run.json` on the coordinator: seq 19, completed, on node `gpupc`, correction `c1` taken up, `correction.receive` by the check actor. The agent's recorded outcome gives the same hash for `worker-only.dat` and says the bundle exists. Main has `stamps/plover-p3.txt` (`309d8ba`). Both attempts ended on `gpupc`, integrations 503 and 504 `integrated`, no job open. `stamp.yaml` on main has the later text; the run's frozen copy does not.

## Session

The correction's attempt recorded its session under the workflow's `fresh` policy: a new session each attempt, and no replacement to report. A reuse across restoration was therefore not exercised live, which is the same gap as p2 and belongs to a workflow with a `reuse` policy.

## Fixtures rather than live

Delayed messages stay in `test/workers.test.ts`: a stale upload and a report of an older generation are refused. They were not injected into production.

## Agent behavior

The agent did the work and recorded its nodes in both attempts. No unexplained exit without node records occurred in this trial.

## Findings

**Worker-only untracked files were published by the correction.** Coming back into the new tree as untracked files, `worker-only.dat` and `worker-only.bin` were committed by the checkpoint after the attempt (`853a2f2`) and reached the trial project's main with the integration. They should not have surprised me: any untracked file in a copy is treated that way at a checkpoint, retired or not, and I had written these by hand after the last one. So retirement does not change the outcome, and I did not change the code. It is written into `docs/storage.md` *Retirement*. Consequence here: 200 KB of trial bytes on `wfrun-trial` main, left there. I noticed this only in step 5, when a later run's tree contained them; the check above of "hashes the same" did not reveal it.

**No defect on the retirement, archive, removal, restore or dispatch path.** Worker-backed runs may enter branch cleanup, subject to its own check that their remote tip is retained: this path proves their local data only.
