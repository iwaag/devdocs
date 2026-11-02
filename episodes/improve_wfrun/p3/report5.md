# improve_wfrun p3 step 5: verify, deploy and measure a small live batch

Date: 2026-11-02. Machine details, selections, deletion logs and measurements are in ignored local records (`pj-agdev/.local/improve-wfrun-p3/`).

## Releases and checks actually run

| Release | agdevworld | Package check | Why |
| --- | --- | --- | --- |
| 1 | `a4a87c1` | 704/704 | backup and recovery |
| 2 | `2951d02` | 712/712 | branch cleanup |

Coordinator and worker `gpupc` on `2951d02`, read back on both; executor unchanged `a9b82f8`. No store schema changed.

- **Replay of every registered workspace on the final release:** 21 workspaces, 500 runs, all usable, before and after the batch (498 before; two new runs since).
- **Registry-scale timings against p1's regression:** after the batch, `wfe intake status` 4 s (p2 3.7–3.9), `wfe notify status` 32 s (p2 34–39), `wfe delegate targets gimmickimg` 1 s. Backup and inspection are separate processes and add nothing to these paths. The executor's notify pass kept running, failures 0.
- **Cost of the new operations:** backup 24–55 s, `wfe branches inventory` 5 s, `wfe branches preview` of 11 runs 5 s. Deleting 8 refs took 11 s.

## Trial project first, `wfrun-trial`, check actor `improve-wfrun-p3-check`

| Trial | Result |
| --- | --- |
| Preview of three retired runs, one worker-backed | all eligible; dry run changed nothing |
| Deleted `stamp/run-001`, `-002`, `-008` | done, each at the object previewed |
| Reads with the branch gone | run view (HTTP 200), `wfe run show`, `wfe run storage`, `wfe projects run --refresh` |
| Receipt replay of `run-008`, through `wfe request` | "Already requested (same receipt)"; no job added by it |
| Recreate `run-001` | `would-recreate`, then at exactly `8494b875` |
| **Correction of `run-008` after its branch was deleted** | restored, dispatched to `gpupc`, branch back on Gitea at `ed8b085`, with the deleted tip as an ancestor; integrated |
| Recreate on `run-008` afterwards | `conflict`, nothing overwritten: intended |
| A new request after cache refresh | got `run-009`: no number reused |

## The live batch

Six real retired runs, eight refs, chosen for variety and fully inventoried:

| Run | Why it is in the batch | Refs |
| --- | --- | --- |
| jevlab experiment/run-002 | plain | 1 |
| lowerpoly lod-chain/run-001 | published output | 1 |
| misc freework/run-001 | nested and shared repositories | 3 |
| playprey onboard/run-002 | **cancelled, unintegrated** | 1 |
| playprey freework/run-001 | worker-backed | 1 |
| gimmickimg freework/run-014 | output adopted by protoprey | 1 |

All eight deleted. After deletion, for each of the six: record read, `restore --dry-run` clean, run view 200, access read with cache refresh. Five outputs verified in full with `wfe share verify`, including gimmickimg's adopted `run-014/o1`. Adoption was checked through that shared reference; protoprey itself was not exercised.

**Recovery on a real run:** `misc freework/run-001`, all three refs recreated at their exact objects; restored, each repository at its saved commit (`0b80372`, `8d9563f`, `741ec20`); retired again; backed up at its new generation; branches deleted again. No correction was dispatched on a real run: that proof is the trial's, as the plan allows. A real run's restoration changes its generation, and the preview correctly required a new backup before allowing the branches to go again.

**Final rehearsal**, on the backup host from the final generation: 25 s to restore, 90 s to rehearse, 10 runs, 12.52 GiB, 18784 comparisons, every check passed, including 9 deleted branches recoverable from what was restored.

## Measurements

**Refs.** 730 → 723 run branches on Gitea. That is −8 in the batch, trial net 0 (−`run-002`, +`run-009`), and +1 from `agops recover/run-069`, described below.

**Allocated bytes per location.** Measured with `du`, each location once, not summed across copies.

| Location | Before batch | After deletion | After maintenance |
| --- | --- | --- | --- |
| Gitea, all repositories | 3727932 KB | 3727960 KB | 3727444 KB |
| Access caches | 2103524 KB | 2103624 KB | 2103604 KB |
| Retention repositories | 1463600 KB | 1463600 KB | 1463600 KB |

**Branch deletion freed nothing measurable.** Each touched Gitea repository moved by 4–8 KB, and the total rose by 28 KB with unrelated activity inside the window.

**Ordinary maintenance** (`git gc`, default prune, selected repositories, no open attempts):

| Repository | Before | After |
| --- | --- | --- |
| developer/pj-wfrun-trial | 740 KB | 468 KB |
| trustedagent/pj-jevlab | 31348 KB | 31104 KB |

Seven objects remain unreachable in each, and default expiry keeps them for now. This is packing as much as collection; zero is close to the honest figure.

**Upper bound probe**, on disposable copies inside the Gitea container, `gc --prune=now`, production untouched:

| Repository | Before | After |
| --- | --- | --- |
| developer/pj-misc | 22456 KB | 10708 KB |
| developer/pj-playprey | 362484 KB | 312112 KB |
| developer/wedo-misc | 13820 KB | 12980 KB |
| developer/pj-gimmickimg | 1295984 KB | 1294280 KB |
| trustedagent/pj-lowerpoly | 123132 KB | 122416 KB |

These are **not** savings from deleting branches: packing loose objects is included, and no pre-deletion control was measured. The pj-misc figure especially looks like packing. They bound what a collection could do in these repositories at most.

**Preservation costs, stated apart:**

| Item | Size |
| --- | --- |
| Retention, kept on the coordinator | 1.40 GiB |
| Archives, kept on the coordinator | 4.51 GiB (p2's figure, including 0.28 GiB received from the worker) |
| Backup destination, 7 generations | 7.09 GiB, on another machine |
| Cleanup's reachability cache | 23 MB |

**Limits on attribution.** `df` used moved −4.9 MB around deletion, then **+3.4 GB** around maintenance and the probes. I cannot attribute that from here: the temporary copies were removed inside the container, whose disk image may not shrink, and other work was running. So `df` is not evidence of anything in this batch. Nothing here extrapolates to the registry.

## What else happened on the way

- **An unexplained agent exit, recorded as the plan asks.** `stamp/run-009`: the agent found `stamps/heron.txt` already right, said so, made no node record, and exited 0 with its stamp node pending. The log shows the agent ending its turn normally. It is **agent behavior, not storage**: nothing failed in the record, the checkpoint or the worker. Observer raised it, and `agops recover/run-069` correctly continued the run. I then cancelled the run, stopping that continuation, since its number was all I needed from it.
- `stamp/run-008` agent attempts recorded nodes normally in all three attempts.
- Worker-only files on trial main: see [report3.md](report3.md) *Findings*.

## Not verified live

- `wfe backup reinstate` (fixture; no live damage, as the plan requires).
- A real run's correction after branch deletion (trial project only).
- Ref races and partial deletion (fixtures, as the plan requires); no live deletion failed, so retry has not run live.
- Session reuse across restoration: the trial workflow is `fresh`.
- Whether fewer refs make clones, fetches or Gitea faster: not measured.

## Recommendation from the remaining inventory

- **Branch cleanup is not a storage tool.** Gitea holds 3.56 GiB against 53.3 GiB of coordinator copies. Do not widen it for bytes. Widen it only if a measured operational cost of 723 refs appears, such as clone or fetch time.
- **Bytes are in the copies.** More retirement is the lever, decided case by case with `wfe runs storage` and `--dry-run`. Runs with large unique untracked data move bytes rather than free them unless the responsible agent discards some.
- **Discarding reproducible data** belongs to each project's agent, per run, with `--discard <path>=<reason>`. Nothing here authorizes removing `.godot`, environments or generated files by name.
- **Deduplication:** not justified yet. Retention is 1.40 GiB and p3's added runs barely moved it, though they were small; look again when it, not the copies, is what grows.
