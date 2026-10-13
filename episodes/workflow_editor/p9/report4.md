# p9 step 4 — Integrating accepted results, separately from saving

Step 4 of [plan.md](plan.md). Committed in agdevworld `490132c`, with the
contract in `workflow_editor/docs/integration.md`. Nothing is deployed.
agautolab needs no change: the executor already calls `wfe exec reconcile`
every minute, and reconcile now also carries out due integrations.

## What changed

### An acceptance binds to an exact revision

- When `accepted` is recorded, the file layer (`prepareOp`) adds
  `revision` to the decision. This covers the person's own decision and the
  one a confirmation answer makes.
  - The revision is the tree of each repository the run owns, as the copy
    holds it then. It is computed in a temporary index, includes uncommitted
    files and leaves ignored files out.
  - Each tree is pinned (`refs/wfe/accepted/<tree>`) so garbage collection
    cannot remove it.
- Integration first makes the final checkpoint. It uses the commits saved
  after every attempt ended, and the run branch must be published as it is
  locally.
- Those commits may differ from the accepted revision only in the run's
  own record folder (administrative) and in the gitlinks of submodules the
  run owns. Any other difference stops it: "the result changed after it was
  accepted …; present it again for agreement".

### Integration (`server/integrate.ts`)

Per repository the run owns, submodules first (devdocs first), then the root.

| Case | What happens |
| --- | --- |
| Unchanged by the run | Left as its target has it (`already`). The root's gitlink for it is not touched. |
| Target not moved | Fast-forward. |
| Target moved | The combined result is built in a `git worktree` of the copy's repository, under the run's copy folder, and removed afterwards. In the root, the owned submodules' gitlinks are set to their integrated commits. |

The combined result is then checked:

- A **conflict** stops with its files.
- A **clean combination that still changes files the run changed** stops as
  `needs-agreement`, because it differs from what was accepted.

Both are agent work: a linked repair run. Nothing has been pushed when they
stop.

**Pushing:**

- Every push is plain, never forced.
- A rejection stops with `attention`.
- Updates of one remote branch take turns on this host, through a lock per
  remote URL and target shared across runs and projects.

**Journal** (exec.sqlite v2, table `integrations`, one row per accepted
decision):

- Each step records target, observed, candidate, method, result and pushed,
  and is written before its push.
- **Retry.** A retry continues from the journal:
  - a done step is checked against the remote again;
  - a push that happened before a crash is found on the target, not
    repeated;
  - a changed candidate is computed anew.
- **States:** `integrated`, `partial` (progress reached a target, then
  stopped), `conflict`, `needs-agreement`, `attention`, `failed`,
  `refused`, `planned` (dry run).
- **Recording.** When every repository is done, `run.integrate` is recorded
  in `run.json` (`integrations[]`, receipt `integration:<id>`), and a
  checkpoint saves it on the run branch.
- **The live record.** The run folder reaches main as a snapshot; the copy
  keeps the live record.

### Queueing and preconditions (`server/exec.ts`)

- An acceptance queues the integration from either place it is recorded:
  - the decision route (`afterDecision`);
  - the end of an attempt, for a confirmation recorded during that attempt.
- The execution host's reconcile carries out queued rows once the run is
  ready:
  - the latest decision is an accepted one with a revision;
  - the run has its own copy;
  - the execution is completed and no attempt is open;
  - the result is not integrated yet.
- A refusal or failure stays on its row, and nothing retries by itself.
- A completed run without acceptance stays saved and unintegrated.
- An acceptance recorded before p9 has no revision, so it is refused with
  "accept the result again to integrate it".

### Surfaces

| Surface | What it offers |
| --- | --- |
| CLI | `wfe run integrate <run> [--dry-run \| --status] --by <name>` |
| HTTP | `GET/POST runs/<wf>/<run>/integration` and `GET …/exec` (`integrations`). The generic ops route does not accept `run.integrate`. |
| Simple (focus card) | <ul><li>"Saved on its branch — not integrated (no accepted result)"</li><li>"Accepted — integration queued"</li><li>"Partially integrated (… done)"</li><li>"Accepted — not integrated: conflict / needs agreement / …"</li><li>"Integrated into main"</li></ul>The wording is in `src/integrationState.ts`, which has no DOM, so it is unit-tested. |
| Detail | <ul><li>"Execution and publication → Integration": the state, each repository's step, "Check integration" (dry run), and "Integrate now" / "Retry integration".</li><li>Run details: "Saved and integrated".</li></ul> |
| Zulip | An `integrated` notice with each repository's target, method and result. |

The docs are `docs/integration.md` (new) and `docs/runs.md` (Acceptance and
operations). The README links both.

## Verification

`test/integration.test.ts` (new, 5 tests) covers every case in the plan's
completion evidence:

| Case | Evidence |
| --- | --- |
| Unchanged target | <ul><li>Before the acceptance: "Saved on its branch — not integrated". After it: "Accepted — integration queued". `main` has not changed.</li><li>`reconcile` reports "integrated", and `run.integrate` records the study as `already` and the root as `fast-forward` from the observed commit.</li><li>`main` now has the change and the run folder snapshot.</li><li>A completed run that was never accepted is not integrated and is refused with "stays saved on its branch, unintegrated".</li><li>A second integration is refused as "already integrated".</li></ul> |
| Advanced target + shared submodule | <ul><li>Meanwhile, the project's main gains `NEWS.md` and the shared study's main gains another project's file.</li><li>The dry run plans `merge`/`merge` and records nothing.</li><li>The study gets both files.</li><li>The root's gitlink is the study's integrated (merge) commit, and the root has both changes.</li></ul> |
| Conflict | Stops as `conflict` with `app.txt`. `main` is unchanged, and the run's work stays on its branch. |
| Clean combination of the same file | Stops as `needs-agreement`, and `main` is unchanged. |
| Result changed after acceptance | <ul><li>The confirmation decision (during the attempt) records a revision with 2 trees.</li><li>The agent then edits `app.txt` and completes. The integration is queued once the run completes.</li><li>`integrateDue` stops it as `needs-agreement`: "the result changed after it was accepted: app.txt". `main` is unchanged.</li></ul> |
| Remote rejection | <ul><li>A pre-receive hook refuses `main`: the state is `partial`, with the study `done` and pushed and the root `attention`. Simple says "Partially integrated (study/study-x done)".</li><li>After the hook is removed, a retry integrates without touching the study again, and the root records the study's commit.</li></ul> |
| Crash after push | <ul><li>`main` already holds the candidate, and the journal says not pushed.</li><li>A retry finds it, does not push again, and keeps the observed commit.</li></ul> |

| Check | Result |
| --- | --- |
| Editor suite | **279/279** |
| `tsc`, `vite build` | clean / ok |
| `checks/exec.ts` | all passed. New: "Accepted — integration queued" in the focus card, accepting changed no shared branch, reconcile integrates it, "Integrated into main", and the accepted work is on `main`. Screenshot: `.local/wfe-checks/screenshots/p9-run-integrated.png`. |
| `checks/simplerUi.ts`, `checks/displayMode.ts`, `checks/runInteraction.ts` | all passed |

## Decisions made within the plan

- **Shared files are conservative.** A clean merge that combines another
  change into a file the run changed is not integrated automatically,
  because the combined file is not the one that was reviewed. The plan
  says: "If the reviewed result changes, present that change for
  agreement."
- **Integration waits for completion.** A result accepted while cleanup was
  still running is integrated once the execution is completed.
- **Repair is agent work.** A conflict or a needed agreement is described in
  the journal and the view. The repair is a new run linked to the original;
  no repair runs are made automatically.
