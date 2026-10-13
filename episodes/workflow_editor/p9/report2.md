# p9 step 2 — Execution and records through each run's own copy

Step 2 of [plan.md](plan.md).

| Component | Where | Commit |
| --- | --- | --- |
| Editor | agdevworld `main` | `23f5d3a` |
| Executor | agautolab, branch `p9` | `35ebcd1` |

**Nothing is deployed:**

- The installed editor release is still `783cfcb`.
- agautolab's `main` (which the running executor reads its guide from at
  every claim) is unchanged. The `p9` branch is merged at deployment
  (step 7).
- The worktree for the branch is `pj-agdev/.local/p9-agautolab` (ignored).

## What changed

**One place per run, resolved at every entrance** (`server/runs.ts`):

- These operations resolve the run's place (`runSpace`) before touching a
  file: `readRun`, `runOp`, `adjustRun`, `registerPresentation`,
  `runResponse`, `readRunFile` and `checkRun`.
- A history view (`?rev=`, `/at/<ref>`) resolves its commit in the run's own
  repository.
- `runResponse` says where the run is: `place` is `run-copy` with its branch,
  or `project`.
- Definition comparison (`sources`) reads the project workspace, where
  definitions are edited.

**Listings and lookups.**

- `runRefs`/`listRuns`/`resolveRef`/`findRunByReceipt` take each run once,
  from the project tree and from the copies. A run with a copy is read
  there, so the tree's folder of the same run is a snapshot.
- A copy that cannot be used is listed with its problem, never dropped.
- This covers the dashboard, `wfe status`, the project view and notify
  scanning, since they all use `listRuns`.

**Creation** (`createRunInCopy`):

- A request (browser/CLI `wfe request`) and a Zulip request (intake) first
  check everything that can refuse the request: project, input, devdocs,
  workflow validity, delegates and compilation.
- Only then is the run's copy reserved and made, and the run folder written
  into it, with `storage`.
- The same receipt finds the same run. An interrupted creation is finished
  under the same number.
- `wfe run create` by an IDE agent still writes into the project workspace,
  numbered under the same allocation lock (`allocating`). A run created there
  and a request can no longer take one number twice.

**Execution** (`server/exec.ts`):

- `claim` returns the logical `workspace` and a new `cwd`, the run's copy.
- Guides are read from the copy.
- The brief names the copy, its run branch and the project workspace. The
  project workspace is the person's: local resources are read from there by
  name, and nothing in it is changed.
- A job whose copy cannot be used is dropped with that reason.

**Publication** (`server/publish.ts` `publishRun`, used for runs with a copy):

- Everything uncommitted in the copy is the run's own work, so there is no
  baseline exclusion there.
- Each repository commits on its run branch, and its run branch is recorded
  in the checkpoint step before anything is written.
- The push is exactly `refs/heads/<run branch>`, never HEAD and never a
  shared branch. A retry publishes to the same place, even after the agent
  checked out something else.
- A repository with another branch checked out is not committed there, and
  the checkpoint needs attention.
- Submodules are pushed before the root. The root waits while a submodule's
  run branch is unpublished.
- Commits are made under the run's record lock, so no record write lands
  between staging and committing.
- Publications of one run take turns (`.git/wfe-publication` lock).
- A checkpoint waits only for its own run's attempt.
- Runs without a copy keep the old publication (`publish`), including its
  baseline rules.
- The detached-submodule fallback to a remote default branch no longer
  applies to isolated runs.

**Consumers:**

| Consumer | Change |
| --- | --- |
| Watcher | Observes each run's folder inside its copy, plus its manifest, and skips the tree's snapshot of the same run. |
| Notifications | Build commit links and read the request's first line in the copy. Gitea commit links work because run-branch commits are pushed. |
| Presentations | Resolve the presentation folder in the copy. |
| CLI | `artifactPath`/`projectPath` are relative to the run's copy, so a path the agent gives from its cwd is project-relative. The same applies to `wfe present register`. |
| Run view (Detail, Execution and publication) | States: "Work: this run's own copy of the project, saved to branch … — not on the project's shared branch until it is integrated." |
| agautolab `wfexec.py` | Launches in `claim.cwd`. |

Docs: `docs/runs.md` covers where things live, where an attempt works,
publication (both kinds) and observation. The contract stays in
`docs/storage.md`.

## Verification

**`test/isolation.test.ts` (new).** Two runs of one project each write
`out/same.txt` in their own copy.

- **Files:** each copy has its own content. The project workspace keeps
  the committed text and stays clean, with no run folder in its tree.
- **Remote:** each run's content is on its run branch, `main` is
  unchanged, and run B's branch carries nothing of run A.
- **Code paths:** `listRuns` and `readRunFile` show both runs completed
  with their own content.
- **HTTP:** `GET runs` lists both. `GET run` gives each its `place`
  (`run-copy`, its branch), and `GET …/file` returns each run's own text.
- **CLI from inside A's copy:** `wfe run list` lists both, and
  `wfe run attach run-001 out/same.txt` records a project-relative path.
- **Watcher and notifier:** the watcher emits change events for both runs
  from their copies, and the notifier owes a `completed` for each.

**`test/exec.test.ts`, updated:**

- The agent's cwd is the copy, and the brief names the run branch.
- Checkpoints are on `wfe/<project>/ship/run-001`, and `main` and the
  workspace are untouched.
- The decision checkpoint is on the run branch.
- The publication test was rewritten:
  - the person's files stay out;
  - the steps record the branch;
  - devdocs is pushed first and the root's failed push is retried with the
    same commit after the agent checked out `scratch`;
  - nothing reaches `main`;
  - a later checkpoint with `scratch` checked out needs attention and never
    pushes `scratch`.

**Other fixtures** (catalogrun, conversation, intakestore, requestadjust) now
write the agent's files where the run lives.

| Check | Result |
| --- | --- |
| Editor suite (`node --test`) | **268/268** |
| `tsc` | clean |
| `vite build` | ok |
| `checks/exec.ts` (browser, port 18198) | all passed, including "the accepted record is on the remote, on the run branch (p9)" |
| `checks/freework.ts` | all passed |
| `checks/runInteraction.ts` | all passed |
| agautolab `tests/test_wfexec.py`, `test_wfintake.py`, `test_wfnotify.py`, `test_wfconversation.py` against the p9 editor source (`WFE_PACKAGE`) | 27 passed, 1 skipped, 1 failed (see below) |

**agautolab results:**

- **Updated:** `test_wfexec` reads the copy and the run branch, and asserts
  that `main` does not have the work. `test_wfintake` reads the braindump
  in the copy.
- **Skipped:** `test_wfnotify::test_legacy_state_is_migrated_and_reconciled`.
  It drives the pre-p5/ex1 editor release over the same area, and that
  release cannot see run copies. The migration it rehearses was done on
  2026-10-11. The skip reason says so.
- **Pre-existing failure:**
  `test_wfconversation::test_a_run_conducted_in_zulip_through_an_outage_and_a_lost_answer`
  fails with `AttributeError: 'FakeZulip' object has no attribute
  'channels'` in `wfnotify.py:193`. It fails the same way on agautolab
  `main` against the installed release `783cfcb`. It is a test double
  that lags the transport, unrelated to p9. It is left as it is and
  recorded here.

## Not yet in this step

- A held or waiting run still reserves its workspace (`holder()`), so the
  two runs above ran one after the other. Step 3 removes that.
- Integration of accepted results is step 4.
- The agent guide (wfrun) is updated with step 3, on the `p9` branch.
