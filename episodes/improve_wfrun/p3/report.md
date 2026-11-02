# improve_wfrun p3: final report — verified recovery and measured branch cleanup

Date: 2026-11-02. Plan: [plan.md](plan.md); steps: [report1.md](report1.md) … [report5.md](report5.md).

## Outcome

Retired runs now have a verified backup on another machine, recovery has been rehearsed where the original files do not exist, and run branches can be removed and put back. All of it is deployed. A live worker run was retired, restored and corrected with its worker-only bytes intact. Nine run branches of real and trial runs are gone from Gitea.

**Deleting branches freed essentially no storage.** That is the main measured result: the refs went, the bytes did not, and Gitea is small next to the copies. Cleanup is safe and recoverable, but it is not a way to get disk back.

**This backup is not disaster recovery for the service.** Gitea, workspaces, credentials and the trees of non-retired runs are outside it.

## Per deliverable

| Deliverable | State |
| --- | --- |
| Backup and recovery operations, contract and help | deployed: `wfe backup …`, `GET api/backup`, daily launchd job |
| Verified independent generation and rehearsal | done on the backup host; every check passed, twice |
| Live worker continuation | done: `wfrun-trial stamp/run-008`, also after its branch was deleted |
| Branch inspection and cleanup, contract and help | deployed: `wfe branches …`, `…/runs/<wf>/<run>/branches` |
| Explicit live batch, recovery checks, measurements | done: 6 real runs, 8 refs; figures below |
| Documentation | `docs/storage.md`, `README_DEV.md`, local notes. `docs/workers.md` and `docs/access.md` did not need changes |

## Revisions

| Repository | Revisions |
| --- | --- |
| agdevworld (`workflow_editor/`) | `a4a87c1` backup and recovery · `2951d02` branch cleanup · `33b357d` a docs paragraph only |
| Released WFE | `2951d02` on the coordinator and worker `gpupc` (previous `aec2430`); executor unchanged `a9b82f8` |
| pj-agdev | `e305073` (pin, launchd template) |
| developer/pj-wfrun-trial | trial runs `stamp/run-008`, `-009`; `37f8fa6` a later definition; `853a2f2` trial bytes, see *Problems* |
| devdocs | step reports, this report, `README_DEV.md` |

## Recovery procedure, current release

Full text: `docs/storage.md` *Backup and recovery*.

1. `wfe backup status`, then `wfe backup verify`. A failure names the affected runs.
2. **To read and restore in isolation:** `wfe backup restore --into <empty folder>`, or on the backup host `--from <path>`, with no registry. `wfe backup rehearse --into <folder> --by <name>` runs every check.
3. **After losing pieces of live storage:** stop the executor and service, `wfe backup reinstate --by <name> --dry-run`, then without `--dry-run`. Nothing that exists and is intact is overwritten.
4. **A deleted branch:** `wfe branches recreate <wf>/<run> --workspace <id> --by <name>`.

Measured on the final generation: 25 s to fetch and verify 6.31 GiB, 90 s to rehearse, 12.52 GiB of trees restored.

Rollback to pre-p2 code is **unverified and not a fallback**.

## Measured

| | Before | After |
| --- | --- | --- |
| Run branches on Gitea | 730 | 723 (−8 batch, trial net 0, +1 unrelated recovery run) |
| Gitea, all repositories, deletion alone | 3727932 KB | 3727960 KB |
| Gitea, after ordinary maintenance on 2 repositories | | 3727444 KB |
| Retention repositories | 1463600 KB | 1463600 KB |

Costs of preservation, apart: backup destination 7.09 GiB for 7 generations on the worker's disk, retention 1.40 GiB and archives 4.51 GiB on the coordinator, a 23 MB cache. Nothing extrapolated. `df` is not usable as evidence here: see [report5.md](report5.md).

## Acceptance evidence

"Fixture" is a test; "live" is the deployed system.

| Case | Result | Evidence |
| --- | --- | --- |
| Backup coverage | met | live: 10 of 10 retired runs covered, every file and ref verified at the destination; 490 runs named as identity-only |
| Independent recovery | met | live: on a host without the original paths; Git limited to local files; jobs and attempts unchanged |
| Consistency and failure | met in fixtures | interruption, a changed byte, a restoration during capture, lost coverage: `LATEST` stays. Live captures had no failure |
| Worker continuation | met | live: both files by hash, frozen definition by hash, bundle recovered by hand and readable |
| Cleanup eligibility | met | fixture: not retired, no backup, older generation, unsettled worker, unretained tip, pin. Live: a restored run needed a new backup first. Held and standing runs share retirement's tested check, with no cleanup fixture of their own |
| Ref race and retry | met in fixtures | moved during and before deletion; partial deletion continued. No live failure occurred |
| Recovery after deletion | met | live: exact refs on nested repositories; trial correction republished its branch on the worker |
| Shared storage | met | live: 5 outputs verified including an adopted one; three shared and nested repositories; other runs' refs untouched in fixture. protoprey itself not exercised |
| Identity and reads | met | live: receipt replay, next number not reused, run views and access reads after cache refresh, replay of all 500 runs |
| Capacity | met | above and [report5.md](report5.md) |

## Problems and open items

1. **Worker-only untracked files were published by a correction.** A file that comes back untracked is committed by the next checkpoint, like any untracked file in a copy, and integration carries it on. Retirement does not change that, so I documented it rather than changing code. 200 KB of my trial bytes are on `wfrun-trial` main. A run holding sensitive or large unignored files would publish them too, retired or not; you may want that looked at on its own.
2. **Agent ended without recording nodes, again.** `stamp/run-009`, exit 0, its stamp node pending, after deciding nothing needed doing. Agent behavior, no storage defect; cause still open as a separate investigation. Observer and `agops recover/run-069` handled it properly, and I cancelled the run. Its recovery run counts as the +1 branch.
3. **Defect found and fixed:** restoring a nested repository under a symlinked path failed. Not reachable on the production path.
4. **One backup destination.** Generations hard-link, so they share bytes: they guard against bad captures and logical damage, not against that disk. Tokens are not in the backup, by decision: copying them to another machine seemed yours to decide.
5. **Pre-existing and untouched:** 21 queued jobs of `agops evidence-fixture`, waiting since 10-27 for a profile no worker offers, on completed runs. They keep those runs from retirement. Not mine to drop.
6. **My earlier retirement:** one more real run than p2's, `misc freework/run-001`, 832 KB, because none of the nine had nested repositories. `forge freework/run-005` was tried as a live cancelled run and correctly refused as held.
7. `stamp/run-008` is left restored and `ready`, with its worker copy; retiring it again is one command.

## Follow-up

- Do not widen branch cleanup for bytes. Widen it only for a measured cost of refs, which was not measured here (clone or fetch time).
- More retirement is the lever for bytes, case by case. Discarding reproducible data remains each project's agent's decision, with a reason, never by name in bulk. [report5.md](report5.md) has the reasoning.
- Deduplication, automatic retention and retention pruning: nothing here justifies them yet.
- Whether to back up Gitea and the workspaces, which is what real disaster recovery would need.
- Whether credentials belong in private backup storage, and a second destination.
