# improve_wfrun p2: final report — retiring run copies without losing records or work

Date: 2026-11-02. Plan: [plan.md](plan.md); handout: [handout.md](handout.md); steps: [report1.md](report1.md) … [report5.md](report5.md).

## Outcome

A finished run's record now outlives its working copy, and the copy can be retired and restored explicitly, on the coordinator and on workers. It is deployed: all 490 existing copies were relocated, 9 runs are retired today (7 real, 2 trial), and a correction of a retired run restored it and ran on its frozen definition. It freed about 4.15 GiB on the coordinator and 0.86 GiB on the worker from the batch, measured, which is much less than the copies' own sizes suggest.

The handout is done separately: gimmickimg's `partial-animation` is `ready` and has run once for real.

## Per deliverable

| Deliverable | State |
| --- | --- |
| Durable records, resolvers, relocation | deployed; 490 of 490 relocated, 516 runs replay usable |
| Inventory, retire, restore (CLI, HTTP, run view) | deployed; no scheduler; run branches kept |
| Worker lifecycle | deployed; tried live with a worker-only file and with a real run |
| Measured live batch | 7 real runs; figures below |
| `partial-animation` | repaired by gimmickimg, verified ready, first run integrated |
| Integration diagnostics | deployed; the invalid path shown in fixtures, not yet by a live integration |

## Revisions

| Repository | Revisions |
| --- | --- |
| agdevworld (`workflow_editor/`) | `041c7a7` steps 2–3 · `fb47811` step 4 · `35c01d9`, `29ef411`, `aec2430` fixes from live checks |
| Released WFE | `aec2430` on the coordinator and worker `gpupc` (previous `145c852`); executor unchanged `a9b82f8` |
| pj-agdev | `d533317` (pin) |
| developer/pj-gimmickimg | `fcd191e` the repair (freework/run-019) · `af2fa0f` the first run (partial-animation/run-001) |
| developer/pj-wfrun-trial | trial runs and two trial edits of `stamp.yaml` (`a845fdb`, `2fdbc03`) |
| devdocs | step reports, this report, `README_DEV.md` *Run records and working copies* |

## Measured capacity

| | Coordinator | Worker `gpupc` |
| --- | --- | --- |
| Inventory before p2 (per-copy `du` sum) | 72.4 GB in 487 copies | 4.1 GB in 6 copies |
| The same copies, allocated in aggregate | 56.5 GB | — |
| Batch: allocated change | −4.15 GiB (`df`: −4.02 GiB) | −0.86 GiB |
| Batch: of which kept as retention | 1.41 GiB | — |
| Batch: of which kept as archives | 4.51 GiB (including the worker's 0.28 GiB) | — |

Not extrapolated to other copies. Whether retiring more is worthwhile depends on what those copies hold: Git-heavy copies free their working files and share retention, while copies with large unique data mostly move bytes unless a responsible agent discards some. The inventory is in local records.

## Acceptance evidence

"Fixture" is a test; "live" is the deployed system. Details in [report3.md](report3.md) and [report5.md](report5.md).

| Case | Result | Evidence |
| --- | --- | --- |
| Durable records | met | live: run views, records and files of retired runs |
| Authority | met | fixture: one lock through both paths; live: relocation with writers stopped, replay after |
| Eligibility | met | live: available presentation, standing, queued, held, in progress, each with its reason |
| Local data | met | fixture: every kind, nested submodules; live: untracked and ignored on a trial, a real run restored |
| Worker data | met, one part fixture only | live: worker-only file, `waiting`, removal, stale messages; re-dispatch bringing files back is fixture only |
| Interruption | met in fixtures | not run live: no live operation failed |
| Corruption/unavailability | met in fixtures | truncated, missing, garbage archive; missing retention repository; share away |
| Restoration | met | live and fixture: saved commit, latest record, no job |
| Identity | met in fixtures | replayed receipt finds the retired run; number not reused |
| Shared outputs | met | live: four outputs re-verified after retirement; fixture: restoration from the share; adopters not followed separately |
| Continuation | met | live: correction on a retired run, frozen definition's hash unchanged; fixture: races. Session reuse not compared in detail |
| Handout | met | live: `ready`, actual run, output on main, numbers recomputed; see [report4.md](report4.md) |
| Capacity | met | table above |

## Who did what on the handout

Found by the Omni Agent in p1; repair and run requested by the Omni Agent as `improve-wfrun-p2-check`; implemented and run by gimmickimg's own runs; verified by the Omni Agent. The first run's result was confirmed by that check actor, **not by the developer**: nobody has yet judged how the leaf looks.

## Problems and open items

1. **Agents ending without recording nodes, cause unknown.** `freework/run-019` twice and `stamp/run-005` once did the work, committed and exited with no node recorded; recovery completed them. Part of the first case was a `wfe validate` defect, now fixed. The rest is not explained, and it is not specific to p2's code as far as I can tell: the first case began on the p1 release.
2. **My mistake on a real run**: re-retiring `gimmickimg freework/run-015` before a fix dropped 60 files from its archive list. No bytes lost, repaired and re-verified; it leaves one harmless extra marker file. See [report5.md](report5.md).
3. **Defects found by testing before they could matter**: staged-only bytes not retained, sockets and pipes invisible to Git, and the output check reading a project tree's snapshot.
4. **The retention store is shared.** Damage to one repository affects every retired run of it. Restoration reports that; nothing repairs it. It collects no garbage by itself, and neither it nor the archives are in a backup.
5. **Rollback is code and data** and its procedure, in local notes, is **not rehearsed**. A pre-p2 release cannot read relocated records.
6. **An agent's `git rebase` or `reset` across different snapshots** of its run folder fails with Git's own message. Safe, and the brief has a line on it, but no live agent has met it.
7. An archive stays after restoration until the next retirement replaces it. A worker's unpublished commits come back as bundle files, not refs. An acceptance of a retired run is refused until it is restored.

## Follow-up

- Deciding what else to retire, which is now a measured question, with `wfe runs storage` and `--dry-run`.
- Branch cleanup and further deduplication, deferred by the plan, still to be justified by these numbers.
- A backup position for retention repositories and archives.
- The cause of item 1.
- `wfrun-trial` stays as a fixture; its copies are small.
