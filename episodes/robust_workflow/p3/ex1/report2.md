# Step 2 report — each mission has its own working state

Plan: [plan.md](plan.md) step 2. 2026-09-24 (JST), by the Omni Agent. Code: agautolab `ec0b065`
(written, tested and pushed; deployed in step 4). The same commit holds step 3's integration,
because the two meet in the close-out. [report3](report3.md) describes that part.

## What was built

**`agautolab.missionspace`** (new module) holds a mission's own copy of its project:

| Concern | How |
|---|---|
| Identity | `.local/missions/<slug>/m<mission id>/`, with branch `autolab/m<id>` in every repository. The mission id is the anchor id of its `[selfnote][mission]` note, so nothing is keyed on a name |
| Contents | One git worktree per repository of the project folder, meaning each top-level folder that is a clone. That covers `main`/`direction`/`devlog` and a pattern project's own repositories alike. Everything else, such as README_PROJECT.md or a main-only plain `devlog/`, is a symlink to the project folder's own. A `.mission.json` records the project, the branch and each repository's base commit |
| Creation and recovery | `ensure_view` is idempotent. It keeps an existing worktree as it is (dirty files included). It re-attaches a missing one from its branch, and it attaches a repository added to the project later (and `autolab project init-repo` typed inside a copy attaches the new clone itself). A restart, a callback or the next task therefore resumes the same copy and never makes a second one |
| Base | A new branch starts at the repository's shared branch. When a branch holds nothing of its own, it is re-attached at the newest shared state |
| Publication from the copy | Refused. The copy's own worktree configuration sets `remote.origin.pushurl` to a path that is no repository, so `git push` fails from the copy and still works from the project folder |
| Serialization | `project_lock` (`flock` on `.local/missions/<slug>/.lock`) around creation, release, integration and record commits. The listener's single executor already serializes servings. The lock is for the CLI inside a run and for a retried close-out |
| Release | `release_view` commits anything uncommitted onto the branch, removes the worktrees and the folder, and keeps the branch |

**Routing (`zulip_listener`).** A task serving calls `ensure_view(slug, mission id)`. The supercoder
runs with the copy as its working directory, and its prompt says where it is:
"this mission's own copy of the project (…): each repository folder in it is a worktree on the
branch autolab/m<id>, which only this mission uses. The project folder itself (…) holds only
integrated work; read it, never write it."
- Tests, staging and commits happen in the copy.
- Planning still runs in the project folder, since a plan is about the shared result. Anything
  the planning run writes into a clean repository there is committed right after it as
  `[AUTO] planning notes (<topic>)` and published for the standard repositories. A repository that
  was already dirty is left alone, and the reply says so.
- The devlog record commits exactly its own task folder instead of `git add -A`.

**Cancellation and ending.**
- `cancel.flag` and `replace.flag` release the ended mission's copy. The reply says what was kept
  and on which branch. A cancelled task already refuses to run (`run_target`), so no serving of it
  reaches a close-out and nothing of it is integrated.
- The last task's close-out also releases the copy. Its work is integrated by then, and a re-plan
  that adds a task re-attaches the branch.
- Archiving a project releases every copy first, because moving the project folder would strand
  the worktrees.

**Generated outputs and caches.**
- Ignored files, such as each repository's `.local/`, raw generations and virtualenvs, are not in
  a copy, so one mission's outputs cannot overwrite another's.
- The prompt names the project folder for reading shared inputs. Immutable references stay where
  they were (`agrefs`).
- The generation directories were already per serving.

## Existing dirty shared work (rollout)

A survey of all 16 project folders found:

| Repository | State | Handling |
|---|---|---|
| `mediagen/gentest-actionDatasets` | ` M datasets/cat_idle/meta.json`, last written 2026-08-31 22:48, 43 s before `688d8bc` "fill cat_idle meta.json". Probably that task's, not certainly | **Set aside** with `missionspace.set_aside` onto `autolab/set-aside-20260924` (`7a601af`), and the checkout cleaned. Nobody receives it until a person decides |
| `mediagen/gentest-YuE2`, `studyrealworld/publish`, `studyuspolitics/publish` | One unpushed commit each. Their READMEs say `publish/` is never pushed by an agent | Left alone. autolab does not publish these repositories unless a worker names them (`publish.flag`). The integration step would push `gentest-YuE2`'s old commit only if a worker named it, so it is listed here |
| `mediagen/publish` | Its remote branch is gone | Integrated locally only (no upstream) |
| Every other repository | Clean and level with its remote, including robustp1 at `7c2cb8d` | — |

## Verification (fixtures)

`tests/test_missionspace.py`: 20 cases on real git repositories (scratch bare remotes, no mocks).
The ownership cases:

| Case | Asserts |
|---|---|
| Two missions edit `wordcount.py` | Neither diff holds the other's line. Each commit is on its own branch. The project folder and the remote are untouched |
| A copy found again | Same path, no new worktree (`worktree list` shows the branch once), dirty work intact |
| Checkpoint before agreement | The commit stays on `autolab/m10427`. `git push origin HEAD:main` from the copy fails and the remote still says `base`. The project folder can still push |
| Cancelled with dirty edits | Release commits them to `autolab/m10330`. The next mission's copy, the project folder and `pending_changes` hold none of it |
| Released, then re-planned | Re-attached at its branch with the kept work |
| Repository added later | Attached on the next `ensure_view` |
| Record commit | Takes only its own paths. A stray file stays uncommitted |
| Set-aside | Tracked and untracked changes are on the branch and the checkout is clean. A repeat is a no-op |

Listener fixtures (`test_zulip_listener.py`, mocked realm): the run's working directory is the copy,
and the prompt names it. Cancellation releases the copy and integrates nothing. A planning run's
notes are committed and pushed, while a repository dirty from before is left and reported. The CLI
fixture covers `init-repo` inside a copy: the clone goes into the project folder and joins the copy.

Suite: **295 passed** (was 236).

## Limits noticed

- The mission is the unit. Two tasks of the same mission share the copy, which is intended, since
  they run in sequence.
- A worker can still `cd` into the project folder and write there. The prompt says not to, and the
  folder only receives integrated work, so what it could damage is the shared checkout. The next
  integration refuses on files it would overwrite (step 3) rather than taking them.
- Ignored local data is not shared. A pattern project whose run needs another mission's raw
  outputs reads them from the project folder by path.
