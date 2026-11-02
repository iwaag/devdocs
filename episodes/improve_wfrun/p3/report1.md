# improve_wfrun p3 step 1: recovery dependencies and the backup set

Date: 2026-11-02. Plan: [plan.md](plan.md). Machine addresses, the raw inventory and scripts are in ignored local records
(`pj-agdev/.local/improve-wfrun-p3/inventory/`).

## Deployed state, read today

| Item | Value |
| --- | --- |
| WFE release | agdevworld `aec2430` on the coordinator and worker `gpupc` (`RELEASE.json`, worker checkout read over SSH) |
| Executor | agautolab `a9b82f8` on both |
| pj-agdev | `d533317` |
| Open attempts | none; 1057 ended |
| Queued jobs | 21, all `agops evidence-fixture/run-002…022` `start`, waiting since 10-27 for a profile `hand` no worker offers. Their runs are recorded completed. Older than p3, not touched, and they keep those runs out of any retirement or cleanup |

## Inventory

| Scope | Count |
| --- | --- |
| Managed copies on the coordinator | 497: 488 `ready`, 9 `retired` |
| Run status | 454 completed, 8 cancelled, 33 idle standing, 2 in progress |
| Copies with more than one repository holding a run branch | 159 |
| Repositories named by those copies | 35: 29 on Gitea, 6 on GitHub |
| Branches on the Gitea repositories | 764, of which **729 `wfe/…`** |
| Most run branches | pj-agops 82, study-argue 77, pj-ponderer 73, pj-front 72, wedo-argue 67 |

A shared or nested repository carries a run branch for each run: study-argue and wedo-argue together hold 144. Counting only project roots would miss about a third of the branches.

## Storage, per location

Allocated bytes by `du`, each location once.

| Location | Size | Failure domain |
| --- | --- | --- |
| Coordinator, all copies including retention and archives | 53.3 GiB | coordinator disk |
| Retention repositories | 1.41 GiB, 6 repositories | coordinator disk |
| Archives of retired runs | 4.51 GiB | coordinator disk |
| Worker archive received (playprey freework/run-001) | 0.28 GiB | coordinator disk |
| Coordinator access caches / gitea cache / control backups | 2.0 GiB / 0.17 GiB / 0.72 GiB | coordinator disk |
| Gitea repositories (container volume) | 3.56 GiB | **coordinator host** |
| Shared area | 8.9 GiB | **coordinator disk** (served from it, not a separate machine) |
| Worker `copies/` | 3.27 GiB in 5 project folders | worker disk |

Two facts shape everything below. Gitea and the share live on the coordinator, so a coordinator loss takes out run branches, retention, archives and shared outputs together: a reference to Gitea or to the live share is no backup. And run branch tips are only a second copy of what retention holds if that is checked per run, not assumed.

Today's check of the nine retired runs: `wfe run restore --dry-run` reports archives matching their recorded checksums and every retained commit reachable, for all nine.

## What recovery of a retired run needs

| Dependency | Where it is | In the backup |
| --- | --- | --- |
| Run folder (events, reports, evidence) | `<copy>/record/` | yes, per run |
| Manifest with lifecycle, generation and archive checksums | `<copy>/copy.json` | yes, per run |
| Archive contents, list, index and exclude files | `<copy>/archive/` | yes, per run, against recorded checksums |
| Worker archive with its bundles | `<copy>/worker-archive/<worker>/` | yes, per run, against the worker's reported checksum |
| Every retained commit, staged-only blobs, unpublished commits | retention repository, under `refs/retained/<wf>/<run>/…` | yes, the whole repository per project |
| Bytes omitted from archives because a verified output has them | shared area `runs/<project>/<wf>/<run>/o<n>/` | yes, copied per file with the output manifest's hashes, because the share has no backup of its own |
| Run numbers and receipts of every run, retired or not | `copy.json`, record folders, project tree | yes, snapshot of `copy.json` and record of all runs, plus legacy names |
| Execution and delivery state | `exec`, `notify`, `intake`, `present`, `delegate`, `conversations` stores | yes, `VACUUM INTO` copies |
| Registry and configuration | area JSON files (checked: no embedded secrets) | yes |
| Software | agdevworld and agautolab revisions, on GitHub | revisions recorded in the manifest |
| Credentials | Gitea and worker token files, SSH key | **deliberately not copied**; named as an external dependency. Reading and restoring need none of them; republishing to Gitea does |

## Decisions

**Destination.** A generation directory on worker `gpupc`, beside the message-file backup that already lives there (same SSH transport, a separate disk in a separate machine). 1.9 TB free. Each generation hard-links unchanged files to the previous one, so repeated generations cost little and are still individually complete.

**The guaranteed scope is retired runs.** Non-retired runs have their manifest and record snapshotted for identity and numbering, and are named in the manifest as `identity-only`: their tree and unpublished work are not in the backup. Gitea, the project workspaces and the executor's own state are not in it either, so this backup is **not disaster recovery for the whole service**, and nothing should present it as such.

**Consistency boundary: one retired run.** A filesystem copy across independently changing stores would not be consistent, and stopping the service for a backup would be unreasonable. What actually changes is narrow, so the boundary follows it:

- A run's manifest and record are copied holding its lifecycle lock and then its record lock, the same order retirement uses.
- Its archive files are immutable until a re-retirement, and that changes `generation`. They are checked against the recorded checksums, at the source and at the destination.
- A retention repository is copied holding the lock retirement fetches under (`<bare>.lock`). Its refs are listed in that same hold. It collects no garbage, so objects are never removed.
- After every transfer, each run's state, generation and archive checksums are read again. A run that changed during capture fails the generation by name instead of being published inconsistent.
- Stores are consistent individually (`VACUUM INTO`), with their own capture times. They are context, not the authority: a retired run's authority is its own record.

**Publication.** Files go to `staging/<id>.partial`, every file's hash is read back at the destination, retained refs are checked there with Git, the staged folder is renamed into `generations/<id>`, and only then does `LATEST` move, by rename. A generation that covers fewer still-retired runs than `LATEST` does not move it.

**Outputs that fail to verify** leave that run `covered: false` with the reason. Branch cleanup requires `covered: true`.

**Not inspected:** the content of the 488 non-retired copies; Gitea's internal state beyond repository sizes and branch lists; six of the 35 URLs are GitHub repositories of pj-clusterintent, not on Gitea, so their branches are not counted above and they stay outside cleanup. Adoption references by consuming projects were not followed: they point at shared output paths, which the backup includes for retired runs only.
