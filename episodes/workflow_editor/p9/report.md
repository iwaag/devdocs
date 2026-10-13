# workflow_editor p9 — Isolated runs and non-blocking holds

Final report of [plan.md](plan.md), for [braindump.md](braindump.md). Step
reports: [1](report1.md), [2](report2.md), [3](report3.md), [4](report4.md),
[5](report5.md), [6](report6.md), [7](report7.md).

## Outcome

A person can now hold a run and keep its plan and unfinished work, run
other workflows in the same project, and resume the held run later in its
own copy. Work no longer disappears, and it does not leak into another run.

The incident that started p9 was jevlab `experiment/run-001`: held before
plan agreement, it blocked every other request in its project. It is now
migrated into its own copy and still held, with its plan, and the project
admits requests again.

| Component | Deployed |
| --- | --- |
| agdevworld (workflow editor service, `bin/wfe`) | `513aa3f` |
| agautolab (`wfexec`, wfrun guide) | `fa26c16` |
| pj-agdev superproject | `e51f509` |

All three are on agstudio since 2026-10-14 03:0x JST. They were released
as `2153cf0`, `7c5355c` and then `513aa3f`, each a release switch, with no
open attempt.

## What p9 changed

| Step | Commit | What it changed |
| --- | --- | --- |
| 1 Storage contract | `3be51d9` | <ul><li>Each run the executor performs has its own working copy: `<workspace>/.local/wfe/runs/<wf>/<run>/{copy.json,tree/}`. Its branch is `wfe/<project>/<wf>/<run>` in the root and in every submodule the workspace has checked out.</li><li>The copy is made from the committed project, never from uncommitted edits. Submodules sit at their base gitlinks; nested ones are detached.</li><li>One resolver, `runSpace`. Allocation happens under a lock and counts records, copies and branches.</li><li>Creation is idempotent. `wfe runs index [--rebuild]`.</li><li>`run.json` records portable `storage`: branch, base and target per repository, with no paths.</li></ul> |
| 2 Execution and records | `23f5d3a` | <ul><li>Every operation, the CLI and the API, the watcher, notifications and presentations resolve the run's copy. Lists dedupe snapshots.</li><li>The claim hands `cwd` to `wfexec`.</li><li>Checkpoints commit and push exactly the run branch, which is recorded before writing. There is no shared-branch fallback, and a checkpoint waits only for its own run.</li></ul> |
| 3 Holds that reserve nothing | `be656f1` | <ul><li>`holder()` is removed: only the one execution slot is shared.</li><li>A run without a copy is not executed until it is migrated.</li><li>`because` on stop, resume and cancel. An executing attempt records as its agent. A host-performed run needs `--by` outside an attempt.</li><li>The resume brief names newer commits; they are not merged.</li><li>Simple, Detail and Zulip say what a hold keeps.</li></ul> |
| 4 Integration | `490132c` | <ul><li>An acceptance records its exact revision (trees).</li><li>Integration is separate from saving. It is journaled per repository, submodules before the root. It fast-forwards, or combines in a worktree and checks the result: a conflict or a changed reviewed file stops it, and nothing is forced.</li><li>A per-target lock. Retry after a crash, and partial progress.</li><li>`run.integrate`, `wfe run integrate`. Saved, accepted, partially integrated and integrated are shown apart.</li></ul> |
| 5 Abandonment and reversal | `378f948` | <ul><li>`wfe run reversal`: facts, never an operation. Cancelling keeps the work. Unintegrated work needs no rollback. Integrated work is undone with new commits, with submodule and external effects stated.</li><li>`--predecessor` on requests.</li><li>Copies start from the upstream when the workspace is only behind it.</li></ul> |
| 6 Migration | `116cc58` | <ul><li>`wfe runs migrate [--dry-run]`: an inventory first, then recovery copies and journaled steps (adopt the id, make the copy, place the folder, `run.relocate`).</li><li>Meaning is unchanged: nothing is queued, resumed or announced.</li><li>Checkpoints keep their mode (exec.sqlite v2).</li></ul> |
| 7 Verify and deploy | `2153cf0`, `7c5355c`, `513aa3f` | Verification, deployment, migration, admission probe, a real-agent trial and three deployment findings, all fixed. |

The agent guide (agautolab) covers these facts:

- the run's own copy, and saving to the run branch only;
- holds;
- recording as itself with `--because`, citing the jevlab incident;
- resumption from the base;
- reversal as new work;
- reuse of a predecessor.

## Definition of done

- [x] **A stopped or waiting run no longer reserves the project.** `holder()`
  is removed. Tests cover an answer wait, a hold, a running process, a
  failed save and a restart. A real agent ran B in the same project while A
  was held. The jevlab admission probe passes the old hold.
- [x] **Its plan, records, tracked changes and retained local files stay
  available.** The copy is kept through hold, cancellation and restart,
  with ignored files preserved (`holds.test.ts`). The migrated jevlab run
  keeps its plan, answer and hold.
- [x] **Another run works independently, and the first resumes in its own
  copy.** The same path was edited in two runs (`isolation.test.ts`). In the
  real trial, A resumed in its copy after B was integrated, and asked for
  agreement again.
- [x] **Checkpoints cannot accidentally publish unfinished work to shared
  branches.** `publishRun` pushes only `refs/heads/<run branch>`, recorded
  per step. Another branch checked out needs attention. Old checkpoints keep
  their mode.
- [x] **Accepted integration is identifiable, recoverable and separate from
  saving.** There is a journal per repository, `run.integrate`, a binding to
  the accepted revision, retry and crash recovery (`integration.test.ts`). A
  real integration ran on the deployed release.
- [x] **Existing runs are migrated without cancelling, resuming or erasing
  them.** In `migrate.test.ts` and in production (jevlab): the meaning is
  deep-equal, nothing is queued, and no notification is repeated.
- [x] **CLI, UI, executor and notifications agree** on run identity,
  location, lifecycle and publication state. The same `runSpace` serves all
  of them. `place` is in the API response. The executor's `cwd` is the copy.
  Notification links and request text come from the copy.

## Verification summary

- **Editor:** 285/285 tests (from 261 before p9). New suites: `runspace`,
  `isolation`, `holds`, `integration`, `reversal`, `migrate`, plus a case in
  `present`.
- **Browser and rehearsal checks:** 14 pass. Seven older checks fail the
  same way on the pre-p9 source (a dev server is needed, or the check is
  stale; see report7).
- **agautolab:** 363 passed, 1 skipped (a legacy transport test the
  architecture makes obsolete), 3 not caused by p9 (one test double is
  pre-existing; two need ignored local files).
- **Production:** migration inventory and migration, a second migration as
  a no-op, the admission probe, and the browser in Simple and Detail.
- **Real agent:** a private trial on the deployed release with four
  sessions, about $0.65.

## Remaining limitations and decisions to know

- **Project workspaces fall behind their remotes** once results are
  integrated, because integration pushes from the run's copy. New copies
  start from the upstream, and resume briefs count both. The person's own
  workspace is never pulled for them: whoever works in it directly pulls
  first.
- **Integration is conservative about shared files.** A clean merge that
  changes a file the run also changed waits for agreement (`needs-agreement`),
  and so does a conflict. Repair is a new run linked with `--predecessor`.
  The browser's request form has no field for the predecessor yet; the CLI
  and the API do.
- **Integration waits for completion.** A result accepted while cleanup is
  still running is integrated after the run completes.
- **Copies and run branches are kept indefinitely.** Deleting them, and
  collecting old copies, are outside p9.
- **History is not rewritten.** The jevlab run's old hold entry still says
  `by iwaag via cli`; from now on an agent records holds as itself.
- **IDE runs** (`wfe run create` by an IDE agent) still work in the project
  workspace. They are not executed by the host.
- **Not seen live.** The Zulip `integrated` notice has not appeared in
  production, because no production run has been integrated yet. It is
  covered by the selection tests.
- **Browser checks ran in Chromium.** The changed screens are text and
  buttons, with no media, but Safari was not checked.

## For the Developer

- **jevlab `experiment/run-001`** is held, in its own copy. The run view's
  focus card resumes it.
- **Requests can go to jevlab now**, with or without resuming the
  experiment.
- **Browser entry:** `http://agstudio.local:8093/wfe/`.
- **Commands:**
  - `bin/wfe runs index --workspace <id>`;
  - `bin/wfe run integrate <run> --status|--dry-run`;
  - `bin/wfe run reversal <run>`.
