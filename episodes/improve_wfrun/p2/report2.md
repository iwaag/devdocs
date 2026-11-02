# improve_wfrun p2 step 2: separate records and migrate safely

Date: 2026-11-02. Code: agdevworld `041c7a7` (committed together with step 3, because the manifest type and the worker
routes are shared), `docs/storage.md`, `docs/workers.md`, `docs/runs.md`. Live relocation happens in step 5; this
step is verified in fixtures only.

## What changed

- **Durable record**: new copies keep the run folder in `record/` beside `tree/`, and the tree's documented path is a
  relative link (`server/runrecord.ts`). Worker copies keep a real folder.
- **Resolvers** (`server/runspace.ts`): `runSpace` reads and writes without materializing anything and falls back to the record
  root for a run with no usable tree; `treeSpace` is the working tree or a refusal (`run-retired`, `run-copy`).
  Publication, alignment, worker files and integration use `treeSpace` or refuse.
- **Snapshots**: `publishRun` and `workingTree` stage the durable folder by plumbing (`stageRecord`), honouring the owning
  repository's ignore rules. `alignRecordCopy` moves its repository with the link set aside (`withoutLink`).
- **Lock**: unchanged code. `lock.ts` already keys on the real path, so both spellings are one lock, and it no longer depends on the tree.
- **Worker files**: uploads extract through the link with `tar -P` (names confined first), downloads read the record directly.
- **Relocation** (`wfe runs relocate`, `server/relocate.ts`): dry run, recovery copies (stores, settings, refs, each
  manifest), journal, per-run steps that continue after a crash; refused while the executor reports.
- **Agent guidance**: one sentence in the run-folder line of the brief; the link's failure mode is a refusal.

## Checks actually run

- `test/runrecord.test.ts`, 8 tests: one set of bytes and one lock through both spellings; the snapshot in directory and submodule
  devdocs; the agent's `git add -A`; Git refusing between snapshots; a replaced link folded in (without `run.json`); a missing
  tree still reported as missing with the record readable; Git at a record root failing; relocation with a
  blocked open attempt, a refused live executor, and a crash between staging and swap.
- Full suite at the step 2–3 code: 691/691. An earlier run of the existing suite against the link design
  failed 14 tests, all from real-path checks (`checkSubject`, `childFile`, `context`) and worker tar transfers; those are fixed in the code, not in the tests.

## Limits

- A crash at each relocation step was not injected one by one; the swap crash is. The other steps are covered by their idempotent checks and
  by reading them.
- An agent's `git rebase` or `reset` across different snapshots fails with Git's own message. That is safe but not explained by Git; the brief
  has one sentence on it, and no live agent has met it yet.
