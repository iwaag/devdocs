# p9 step 1 — The run storage contract

Step 1 of [plan.md](plan.md). It is implemented and committed in agdevworld
`3be51d9`, with the contract in `workflow_editor/docs/storage.md`. Execution
does not use it yet; step 2 routes execution and records through it.
Nothing is deployed: the installed release is still `783cfcb`.

## State read before implementation

- **Installed CLI.**
  - `bin/wfe version` reports release `783cfcb`.
  - jevlab `experiment/run-001` is as the handover says: held (`run.stop` at
    seq 9, by `iwaag via cli`), `_plan` running, `plan.md` written, q1 taken up.
  - The pj-jevlab workspace is clean on `main` at `b3599ae`, equal to
    `origin/main`. Its run commits are `7343f59` and `b3599ae`.
- **Service state.**
  - `nctl status` is ok.
  - `nctl drift --host agstudio` shows agstudio converged.
  - The workflow editor service and the executor are launchd jobs. Neither
    is a Nautobot placement of its own; `agdevworld-web` and
    `agautolab-agstudio` are.
  - Nothing was changed.

## Inventory: what assumes one project working tree

Every one of these joins the registered workspace root with a run folder,
or treats the workspace as a run's place:

| Where | Assumption |
| --- | --- |
| `server/exec.ts` | `holder()` reserves the workspace from a run's first attempt until it completes or is cancelled. `request()` creates the run in the workspace. `claim()` hands the registered workspace to the executor as cwd. `execution()` reads guides from the workspace root. `brief()` names the workspace root. `publishCheckpoint()` waits for any attempt in the same workspace. `baseline()`/`checkpoint()` work in the workspace. `reconcile()` lists runs per workspace. |
| `server/publish.ts` | Commits on the checked-out branch and pushes it. A detached submodule falls back to its remote default branch. |
| `server/runs.ts` | `readRun`, `runRefs`/`listRuns`, `resolveRef`, `createRun`/`nextRunId`, `findRunByReceipt`, `runOp`, `adjustRun`, `registerPresentation`, `checkDocument`, `checkSubject`, `readRunFile`, `runResponse` (artifacts), `historyRev` all resolve under `ws.root`. |
| `server/api.ts` | Every run route takes the registered workspace (lists, show, file, ops, exec, checkpoints, notify). |
| `server/watch.ts` | Watches `devdocs/runs/*/*/run.json` and `git status` of the workspace root only. |
| `server/notify.ts` | Rebuilds the outbox from `listRuns` per workspace. Reads a run's input file under `ws.root`. |
| `server/intake.ts` | Finds runs by receipt, creates runs and records answers and instructions in the workspace. |
| `server/present.ts` | Presentation folders and their sources resolve under `ws.root`. |
| `server/agdev.ts` | The dashboard lists runs per workspace. |
| `server/devdocs.ts` | History views read commits of the workspace's devdocs owner. |
| `cli/wfe.ts`, `cli/run.ts` | The workspace comes from the cwd (`workspaceAt`). Artifact and project paths are made relative to `ws.root`. |
| agautolab `wfexec.py` | The agent's cwd is `claim.workspace.path`. |
| agautolab `agent/guides/wfrun/guide.md` | Describes the workspace and publication as shared. |

## The contract (`docs/storage.md`)

- **Place.** A run's copy is `<project workspace>/.local/wfe/runs/<wf>/<run>/`.
  It holds `copy.json` (the manifest) and `tree/` (a clone of the project).
  These folders are the host-local index: one manifest per run, so no
  second database can drift from them.
  - Inside the registered workspace, `wfe` run in a copy finds its logical
    workspace by the existing rule.
  - Inside `.local/`, Git and every tree walk of the project ignore it.
- **One resolver.** `runSpace(ws, run)` returns the run's copy: a
  `Workspace` with the same logical id, rooted at `tree/`, that knows its
  project workspace (`shared`).
  - A run without a copy resolves to the project workspace. These are runs
    recorded before p9 that were not migrated, and IDE-created runs.
  - A copy that is still being created, damaged or missing is refused with
    its problem. It is never replaced by the project tree's snapshot of the
    same run.
- **Identity.** The run branch is `wfe/<project>/<workflow>/<run>` in every
  repository the run owns. The project id keeps projects apart in shared
  study and wedo repositories.
  - The integration target is the branch the project workspace had checked
    out. For a detached submodule it is its remote's default branch.
  - The manifest keeps host-local facts (URLs, state, the request receipt).
  - `run.json` gets portable `storage` from `run.create`: the branch, and per
    repository the path, base, branch and target. It holds no paths and no
    URLs (`validStorage` in the reducer).
- **Allocation.** Allocation holds the workspace's allocation lock and takes
  one past the highest `run-NNN` that is:
  - recorded in the project tree,
  - held by a copy, or
  - named by a run branch Git knows (local or `origin/…`).

  The folder plus its manifest is the reservation. A request's receipt is
  kept, so the same request finds its run.
- **Creation** is idempotent.
  - The root starts at the committed `HEAD` of the project workspace. The
    person's uncommitted edits are never included.
  - Submodules are cloned from the workspace's checkout, so the run gets
    commits only the workspace has. When the workspace lacks a commit, it
    comes from the submodule's own remote. Each submodule is then pointed at
    its own remote and given the run branch at the base gitlink.
  - The layout is standard (`.git/modules`).
  - Nested submodules are checked out at their gitlinks, detached, and are
    not owned by the run.
  - `.local/` is excluded in the copy. The workspace's top-level `.local`
    files (notes such as `localenv.md`) are copied. Folders are not.
  - A half-finished clone is redone. An existing run branch is kept.
- **Definition source.** The definition a run follows is still read from the
  project workspace, as the person sees it, and frozen into the run folder.
  `createRun` gets `into` (the copy) and `storage`.
- **Reconstruction.** `wfe runs index [--rebuild]` lists the copies. With
  `--rebuild`, a lost manifest is rebuilt from the copy's checked-out run
  branch and its `run.json` storage. Nothing is ever cloned, restarted or
  deleted. A copy whose repository is gone stays reported with its branch.

Versions:

- The manifest is `ag.wfe-run-copy.v1`.
- `run.json` stays `ag.workflow-run.v2`; `storage` is an additive field.
- The migration of existing runs is step 6. There is no second, indefinite
  execution mode.

## Git behaviour checked in fixtures before choosing the layout

Probed by hand first, then fixed in `test/runspace.test.ts`:

- A local `git clone --no-checkout` of the workspace with the run branch
  checked out at `HEAD` excludes the workspace's uncommitted edit.
- `git submodule init` plus a local URL override, then `submodule update`,
  gives the standard layout and gets a commit the workspace has not pushed.
  The real URL is set afterwards.
- A nested submodule needs its own local override. Without one it stays
  uninitialized, which is why it is handled explicitly.
- The copy's `git status` is clean, and the gitlinks match.

## Verification

| Check | Result |
| --- | --- |
| `test/runspace.test.ts` (new, 6 tests) | Two runs get separate roots, manifests and branches, with origin the project remote. The person's uncommitted edit stays out. `localenv.md` is copied and `.local/outputs` is not. `.local/` is ignored in the copy. A file written in one copy is not seen in the other copy or in the workspace. A run created into its copy records `storage`, has nothing in the project tree, and resolves to the copy; a run without a copy resolves to the workspace. The same receipt gets the same run. 5 concurrent reservations get 5 distinct numbers. A record folder in the project tree and a run branch pushed by another clone both hold their numbers. A taken name is refused (409). A half clone and a manifest left at `creating` are finished under the same number, and the copy's files are kept. A fresh read after a "restart" lists the same identity and allocates the next number. devdocs and study submodules are on the run branch at their gitlinks, including a workspace-only study commit, and the workspace's uncommitted study edit is not taken. A nested submodule is detached and not listed. A damaged manifest is refused and reported, then rebuilt with `--rebuild` to the original base, branch and target. A missing copy is reported with its branch, and nothing is recreated. |
| Full suite (`node --test`) | **267/267**: the baseline 261 plus 6. Baseline before p9: 261/261. |
| `tsc` | clean |

## Next

Step 2 routes execution, records, CLI/API, watching, notifications and
presentations through `runSpace`, and moves checkpoint publication onto run
branches.
