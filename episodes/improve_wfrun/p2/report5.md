# improve_wfrun p2 step 5: verify, deploy and retire a measured live batch

Date: 2026-11-02. Machine addresses, logs and recovery details are in ignored local records (`pj-agdev/.local/improve-wfrun-p2/`).

## Releases

| Release | agdevworld | Why |
| --- | --- | --- |
| 1 | `fb47811` | durable records, lifecycle, definition reports; the relocation cutover |
| 2 | `35c01d9` | Simple mode did not show a retired copy at all (seen in the deployed page) |
| 3 | `29ef411` | a restored copy lacked files kept as published outputs (seen in a live restoration) |
| 4 | `aec2430` | a re-retirement forgot published files that had not come back (seen on a live run) |

Coordinator and worker `gpupc` both on `aec2430` (commit read back on the worker, its unit active); executor unchanged `a9b82f8`. pj-agdev pin `d533317`.

## Checks actually run

- Package check in each release build: final **695/695** (673 before p2).
- Replay check of every registered workspace with the release before activation: 21 workspaces, all usable. Again after relocation, and again on the final release: 516 runs, all usable.
- After relocation, directly on disk: 490 of 490 copies have the link, reach their `run.json`, and Git shows nothing of the folder.
- Registry-scale timings (the p1 regression): `wfe intake status` 3.7–3.9 s (p1 3.2 s), `wfe notify status` 34–39 s (p1 32 s), `wfe delegate targets gimmickimg` about 1 s. The machine was loaded by test runs during some of these. The executor's passes kept running; the only recurring failures are the two null-byte deliveries p1 already listed.
- Normal new runs on the final-but-one release: `stamp/run-006` on `studio` and `stamp/run-007` on `gpupc`, both integrated.
- Deployed run view of a retired run, in a headless browser at desktop and phone width: the statement and Restore in Simple, details in Detail, no page errors, no horizontal overflow.

## Relocation

`wfe runs relocate`, executor and service stopped: 490 moved, none blocked, 35 s; recovery copies 64 MB. Allocated bytes under the copies unchanged in practice (59.81 → 59.85 GB by `du -c`).

## Trials on `wfrun-trial`, check actor `improve-wfrun-p2-check`

| Trial | Result |
| --- | --- |
| Coordinator: `stamp/run-001` with an untracked and an ignored file | retired; run view and record read; restored; both files and the commit back; retired again |
| Worker: `stamp/run-004`, with a file only on the worker | `waiting` until the worker reported, its archive received, retired, worker copy removed, restored on the coordinator |
| Correction: `stamp/run-005` retired, default branch's `stamp.yaml` then changed, `wfe run return` | the job waited, the claim started the restoration, the correction ran, `stamps/ibis-corrected.txt` reached main. Frozen definition's hash unchanged (`8b75e8d9…`), without the later text |
| Stale messages on retired `stamp/run-002` | upload refused (409); a report of an older generation not accepted |
| Eligibility on real runs | presentation available, standing conductor, queued job, held and in progress: each kept, with its reason |

## The measured live batch

Seven real finished runs, chosen for variety, selection file local. All retired; none failed.

| Run | Copy by its own `du` | Archive | Note |
| --- | --- | --- | --- |
| gimmickimg freework/run-014 | 3.33 GB | 0 | 60 files are its published output |
| gimmickimg freework/run-015 | 3.33 GB | small | see *My mistake* below |
| lowerpoly lod-chain/run-001 | 233 MB | 32 MB | 167 files are its published output |
| agops recover/run-036 | 5 MB | 0 | clean |
| jevlab experiment/run-002 | 239 MB | 16 KB | |
| playprey onboard/run-002 | 6.42 GB | **4.51 GB** | 11970 files kept, 3091 as published output; a cancelled run with large Blender and `.godot` data |
| playprey freework/run-001 | 654 MB here, 922 MB on `gpupc` | 302 MB (worker's) | worker copy removed |

Allocated bytes, measured around the batch (some unrelated activity falls inside the window; `df` figures include it):

| Location | Before | After | Change |
| --- | --- | --- | --- |
| Coordinator, all copies (`du -c`, including retention and archives) | 58.91 GiB | 54.76 GiB | **−4.15 GiB** |
| Coordinator, filesystem used (`df`) | | | −4.02 GiB |
| Worker, `copies/` | 4.12 GiB | 3.27 GiB | **−0.86 GiB** |

Preservation cost included in those: retention repositories **1.41 GiB** (gimmickimg 1.21 GiB for its two runs), archives **4.51 GiB** (mostly playprey), worker archive included.

What this says, and nothing more: 13.2 GiB by per-copy `du` gave about 5 GiB back in total. Most gimmickimg bytes were shared hard links, and the largest run's data is really unique and was kept. The later retirements of a project should cost less retention, since its history is already stored, but that was **not measured**, and nothing here is extrapolated to the other copies. Disk use stays 52%.

Reads and outputs after the batch: run views of retired playprey and lowerpoly runs answered, four outputs re-verified in full against the shared area (`gimmickimg run-014 o1`, `playprey onboard/run-002 o2`, `lowerpoly lod-chain/run-001 o1`, `playprey freework/run-001 o1`). Adoption references by a consuming project were not followed separately; they use those shared references, which do not change.

## My mistake on a real run

To verify restoration I restored and re-retired `gimmickimg freework/run-015`. The first time, restoration did not yet bring back files kept as published outputs, so the re-retirement saw 60 fewer files and its archive list dropped them. The next restoration then showed 0 of 0, which is how I found it. No bytes were lost: they were on the shared area, verified. I restored the run, copied the output back to its original source path (taken from the attempt's `wfe run publish` command), verified 60 of 60 by hash, and retired it again; a further restore showed 61 of 61, and it is retired now. The 61st file is `.wfe-output.json`, the share's own marker that my copy brought along, which was not in the original folder: harmless, and still there. Release 4 keeps such files listed so a re-retirement cannot forget them.

## Not verified

- A live re-dispatch of a restored worker run bringing its files back (fixture only).
- Crash injection on the live system (fixtures only). The live retirements had no failures, so the retry paths have not run there.
- Session reuse across a restoration was not compared in detail; the correction's attempt recorded its session normally.
- The rollback procedure in the local notes was **not rehearsed**.
