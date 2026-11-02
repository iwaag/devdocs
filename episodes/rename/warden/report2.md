# Step 2 report: source, definitions and documentation

## Commits

| Repository | Commit | Content |
| --- | --- | --- |
| agdevworld | `d7a3bd1` | Identity transition, `formerly`, migration command, docs, tests |
| pj-agdev | `71d525b` | Observer defaults and tests, historical note on the seed, submodule pin |
| pj-warden (prepared, not yet published) | `51bfb98` | `project.yaml`, guide, devdocs entry, new root README |
| devdocs | this commit | README_DEV Observer section and *Renaming a managed project* |

## WFE

- `shared/run.ts`: `run.rename-project`, `createdAs`, `runUid`.
- UIDs read the creation project in `observe`, `notify`, `delegate`, `reviews`, `conversations`.
- `registry.ts`: `formerly`, `recordedWorkspace`, `holdsProject`. Applied to stored-value lookups only.
- `runs.ts`, `access.ts`: the reader accepts `createdAs` and former names.
- `notify.ts`: a delegation's conversation follows the rename, so an old `target.project` cannot
  provision an old channel.
- `branchCleanup.ts`: uses the recorded branch; `runBackup.ts`: a live reinstate resolves a former
  workspace id; `retention.ts`: a restored tree gets the manifest's URL, not the hashed one.
- `server/renameProject.ts`, `wfe rename-project`: preview, preconditions, per-step journal.
- `docs/storage.md` *Renaming a project*; `docs/intake.md`; fixture names in `test/delegation.test.ts`.
  Named-trial references in `server/exec.ts`, `shared/run.ts` and `docs/runs.md` stay.

## Observer

Defaults and `docs/workflow.md` name warden. `docs/agops/` stays, marked as history and pointing to
the authoritative project. A new test covers a pre-rename recovery run (old uid, current project).

## Project definition

The prepared commit changes only the name, plus a README that states the current recovery role, the
workflows and the relationship to Observer. Workflow definitions needed no change.

## Checks actually run

- WFE `npm run check` on this code: type check, 717/717 tests, both builds.
- Observer `pytest`: 122/122.

Three of the changes above came from rehearsal findings; see report 3.

## Not done here

Nothing is deployed or published yet. `.local/devenv.md` is updated after the live change.
