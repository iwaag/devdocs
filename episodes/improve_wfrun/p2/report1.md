# improve_wfrun p2 step 1: inventory and the migration boundary

Date: 2026-11-02. Code read at WFE `145c852` (installed release), executor `a9b82f8`. Detailed per-copy inventories
are in ignored local records (`pj-agdev/.local/improve-wfrun-p2/inventory/`); this report holds only what is portable.

## Inventory (read-only, complete scans of both hosts)

| Measure | Coordinator | Worker `gpupc` |
| --- | --- | --- |
| Copies | 487 (21 workspaces scanned, none inaccessible) | 6 |
| Sum of per-copy `du` | 72.4 GB | 4.1 GB |
| Allocated in aggregate (hard links counted once) | **56.5 GB** | not measured apart from `du` |
| `.git` by per-copy `du` | 23.3 GB | 0.9 GB |
| Ignored files found | 15.5 GB (top 50 ignored paths per repository, so a lower bound) | 1.9 GB |
| Record folders | 0.34 GB | 0.00 GB |
| Nested repositories | 401 | 0 |

Coordinator copy states: 444 completed, 33 idle, 8 cancelled, 2 in progress; 49 standing conductors. Finished and
not standing: 436 copies (70.7 GB by per-copy `du`). 50 of those have uncommitted or unpushed data, mostly the 21
`agops/evidence-fixture` runs whose jobs are still queued because their profile is not offered. All 1042 attempts have
ended; 21 jobs are queued; 7 integrations are in non-final or refused states.

Largest by project (per-copy `du`): gimmickimg 18 copies 32.7 GB, playprey 15 copies 24.4 GB, protoprey 2.7 GB, jevlab
2.7 GB, lowerpoly 2.1 GB, forge 1.9 GB. Largest copies: playprey onboard run-001 / 002 / 004 (6.3, 6.0, 3.7 GB), then
gimmickimg freework run-013…018 (≈3.1 GB each, 1.3 GB `.git`).

Findings that change how savings must be measured:

- **Per-copy `du` overstates savings.** gimmickimg's copies sum to 32.7 GB but occupy 22.9 GB: local clones hard-link
  Git objects. Retiring one copy frees only what nothing else links. The problems document's 75 GB is the per-copy kind
  of number. p2 reports aggregate allocation before and after, on each storage location.
- **Ignored bytes are not disposable by name.** The largest ignored data is `.local/` (5.3 GB), `.godot/` (3.2 GB),
  `.venv` (1.3 GB), and Blender/GLB work files (`01_import.blend` and similar, about 2 GB) that exist nowhere else.
- **Worker copies** hold no manifest; they are found by path. All 6 are finished, clean and pushed, with 1.9 GB ignored.
- Published-source copies are still few (ponderer conduct/run-069 and three wfrun-trial runs). p1's concern about copies
  growing through pinned submodules is real in code (`planPublished` initializes every pin) but is not yet visible in
  measurements: no archsage copy has been made since p1. It is not extrapolated here.

## Trace of what depends on the tree

| Area | Today | Needed |
| --- | --- | --- |
| Resolver (`runSpace`) | one resolver; refuses a missing tree | split: record/file reads vs. a working tree |
| Record lock (`lock.ts`) | key = realpath of the folder + `run.json` | stable once the folder resolves into durable storage |
| `readRun`, `runOp`, shares, delegations, reviews, reminders | `join(space.root, runFolder)` | work unchanged against a root that holds the same relative path |
| Publication (`publishRun`), acceptance (`workingTree`) | `git add -A` of the run folder | intentional snapshot of the durable folder |
| Worker sync (`alignRecordCopy`, `PUT/GET files`, `manifestOf`) | `reset --hard`, tar into the tree | must reach the durable folder; stale uploads refused |
| Allocation, receipts | copy folders, manifests, branches | already independent of the tree when the folder and manifest stay |
| Presentations (`present.ts`), reviews (`review_work`), external processes | read the tree | become eligibility reasons |
| Claims, integration, reversal | need code | use the working-tree resolver; a retired run restores first |
| Backup (`wfe-release.sh`, p9 migrate) | area stores and JSON | relocation adds its own journal and backup |

## Experiment behind the chosen path

In a scratch repository (Git 2.50): the run folder as a symlink out of the tree, excluded in `.git/info/exclude`, with its
index entries marked skip-worktree. `git status` is clean; `git add -A` leaves snapshot entries alone and does not capture
the link; `stash` and `merge` keep the link; `git add <file in folder>` fails with "beyond a symbolic link"; and
`reset --hard` / `checkout` across a different snapshot **refuse** ("not uptodate", "would be overwritten") rather than
overwrite anything. The failure mode is a refusal, never a second authority. `alignRecordCopy`'s `reset --hard` therefore
needs an explicit step, and agents get one line of guidance.

## Storage contract (implemented in step 2 and 3; final text goes to `docs/storage.md`)

- **One durable authority**: `<copy folder>/record/devdocs/runs/<wf>/<run>/`, beside `tree/`, is the run's folder. The tree's
  documented run path is a relative symlink to it, so agents write the same bytes at the same path. No second writable
  place exists, and `lock.ts` resolves both spellings to one key. Worker copies keep a real folder, as they already
  are a non-authoritative snapshot synchronized by upload.
- **Record root as a workspace**: the record root has the project-relative layout, so a retired run reads through a
  `Workspace` rooted there, with no reader rewrite. Evidence outside the run folder is copied to the same relative path.
  `record/.git` is an invalid gitfile, so Git run there fails instead of discovering the person's workspace above it.
- **Resolvers**: `runSpace` = where a run's files are read and written, never materializing anything; `treeSpace` = a working tree
  or a refusal naming the state and the action.
- **Lifecycle** in `copy.json` (the existing per-run manifest, no new index): `ready`, `retiring`, `retired`, `restoring`, with
  who, inventory, verification, failure details, per-host progress and a generation. Independent of run status; no run event.
  A missing tree without a lifecycle stays reported as missing.
- **Commits**: a per-project bare retention repository (`<workspace>/.local/wfe/retention/`) with one ref per retained commit.
  It is managed retention rather than per-run bundles because a bundle per copy would repeat the 1.3 GB history of each
  gimmickimg copy and cancel the saving.
- **Non-committed bytes**: a per-run archive beside the record (tar with modes and symlinks, a per-file manifest with
  hashes, index files). Each item is classified: durable record, verified published output, archive, reconstructible, or
  discardable by an explicit reason from the responsible agent. Anything unreadable stays unclassified and keeps the copy.
- **Order**: staging → verify → commit `retired` → release the tree; each step retryable. Restoration: staging from retention
  and archive → verify → link the record → atomic rename → `ready`. It queues nothing.
- **Workers**: pending lifecycle tasks are picked up on the worker's heartbeat; worker-only bytes upload to the coordinator's
  archive. An unreachable worker holds retirement at `retiring` (or `retired` with removal pending) and is shown so.

## Migration map and recovery

1. Release the code (it reads both forms). 2. `wfe runs relocate --dry-run`. 3. Back up the affected stores and manifests;
stop executor and service; no attempt may be open. 4. Per run, under its lock, journalled: copy folder to `record.staging`, verify,
rename to `record`, swap the tree folder for the link, mark the index, stamp the manifest. 5. Restart; run `run check` over all
workspaces. Deletion is a separate later command.

Recovery: an interrupted relocation continues from the journal; a finished one is undone by the reverse swap (`--undo` is
not built; the procedure is in the local notes). A pre-p2 release cannot read relocated records, so this is not a code-only rollback.
