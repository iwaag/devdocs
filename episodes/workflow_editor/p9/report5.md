# p9 step 5 — Abandonment and reversal without erasing evidence

Step 5 of [plan.md](plan.md). Committed in two repositories:

- agdevworld `378f948`;
- agautolab branch `p9` `fa26c16`: the wfrun guide, to be merged at
  deployment.

Nothing is deployed.

## What changed

### Facts, not an operation: `wfe run reversal <run>` (`GET …/reversal`, `server/reversal.ts`)

- **Cancelled.** A cancelled run keeps its records, its copy and its saved
  branch. Nothing is deleted or reverted.
- **Not integrated.** The facts say: "Nothing of run … is on the project's
  shared branches: its work is saved on its own branch … and in its copy on
  this host. Discarding it needs no rollback; cancelling keeps it (nothing
  is deleted)." They also list the saved heads per repository.
- **Integrated.** The facts list each integration and each repository it
  changed:
  - target, before → after, and method;
  - the commits and files in between;
  - the target's later commits (as last fetched; they are kept);
  - steps for an agent to prepare the reversal as new work in a linked run:
    `git revert --no-commit <before>..<after>` (`-m 1` for a merge), then
    `git checkout HEAD -- <the run's record folder>` so the evidence stays,
    then commit.
- **Submodules.** "… is a repository of its own, possibly shared with other
  projects: its revert is a commit on its main; the root then records that
  new commit. Setting the root's gitlink back to … alone would not undo …'s
  branch."
- **Outside Git.** Effects outside Git are named and never claimed undone:
  deployments, data written elsewhere, a presentation still being served.
- **Runs without their own copy** (before p9): the facts point at their
  commits in the project history.

### Links between runs

- `wfe request … --predecessor <run>`, and `predecessor` in `POST runs`,
  record the run that the new one follows. The brief then says where that
  run's records are, which branch holds its saved work, and whether its
  result is integrated (pointing to `wfe run reversal`). This is how a plan is
  reused, and how a repair or reversal run refers to its origin. The plan
  asks for no restart-from-plan screen in this phase.

### A fix found while testing reversal: where a copy starts

- Integration pushes from the run's copy, so the person's project workspace
  falls behind its remote.
- A new request would then copy the stale workspace and miss results already
  integrated.
- A reservation now fetches the workspace branch's upstream (best effort).
  When the workspace is only behind it, the copy starts at the upstream
  commit (`baseFrom: upstream` in the manifest).
- The workspace's own unpushed commits still win, uncommitted edits are
  still never taken, and the person's workspace is not changed.

### What the person sees

- **Cancel prompt.** "…cancelled, for good. Its records and saved work are
  kept (nothing is deleted or reverted)."
- **Detail → Integration.**
  - Integrated: "Undoing it is new work: a run that names this one as its
    predecessor and reverts the integrated commits with new ones (wfe run
    reversal … lists them); later changes stay, and effects outside Git
    need their own check."
  - Not integrated: "Nothing of it is on the project's shared branches:
    cancelling or abandoning it needs no rollback, and its work stays on its
    own branch."

### Documentation

- **Agent guide.** Abandoning and undoing are facts read with `wfe run
  reversal`. A reversal is made of new commits. Submodules are reverted in
  their own repository. Effects outside Git are checked separately. A
  predecessor's records are reused from where the brief says.
- **`docs/integration.md`** has a new section, *Abandonment and reversal*.
- **`docs/storage.md`** describes the base selection.

## Verification

`test/reversal.test.ts` (new, 2 tests):

| Case | Evidence |
| --- | --- |
| Abandonment | A run held mid-build with `half.txt` and then cancelled keeps its copy (`half.txt` is there) and its saved branch (`half.txt` is on `wfe/demo/ship/run-002`). `main` is unchanged. The facts say `not-integrated` with the wording above, and that effects outside Git are not undone by reverting commits. |
| Reversal of an integration | <ul><li>Run 1 changes `app.txt` and the shared study and is integrated. Later work (`NEWS.md`) lands on `main` from another clone.</li><li>The facts give: the study (submodule, `notes.md`) with its note; the root with `app.txt`, the study gitlink and the run folder, and 1 later commit; the root's revert command, which keeps `devdocs/runs/ship/run-001`.</li><li>Run 2 is requested with `--predecessor ship/run-001`. Its brief says where run 1's records and branch are and that its result is integrated. Its copy starts from the integrated project, the upstream rather than the stale workspace.</li><li>Run 2 follows the facts: it reverts the study in the study, reverts the root range, restores run 1's records, records the study's new commit, then completes and is integrated.</li><li>On `main` afterwards: `app.txt` is `original` again and `NEWS.md` stays. Run 1's snapshot stays, and its live record still names its integration.</li><li>The study's `main` is reverted, and the root records the study's new commit.</li><li>History holds the original commits, the later work and the reversal.</li></ul> |

| Check | Result |
| --- | --- |
| Editor suite | **281/281** |
| `tsc` | clean |

## Out of scope, as the plan says

Automatic branch deletion, collecting working copies, an exhaustive
classification of rollbacks, and a restart-from-plan screen.
