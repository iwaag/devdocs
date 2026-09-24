# Step 1 report — contamination reproduced, isolation unit and contract

Plan: [plan.md](plan.md) step 1. 2026-09-24 (JST), by the Omni Agent.

## Where autolab works today (agautolab `13f242b`)

| Concern | Today | Where |
|---|---|---|
| Workspace selection | Every planning and task run has its working directory at the project folder `.local/projects/<slug>/`, where `main/`, `direction/` and `devlog/` (or a pattern project's own repositories) are ordinary clones. Each serving has a private generation directory for its chatlog, report and flags, but the source tree is shared by every mission of the project | `zulip_listener.project_directory`, `serve`, `serve_run` |
| Worker environment | The supercoder runs on claude_code with the permission bypass, `git` in its grant, and the guide line "commit changes in the repository folders you edited after getting the developer's approval". Nothing stops a commit or a push from its working directory | `agents.toml`, `workrun_supercoder/guide.md`, `role_run` |
| Task close-out | Runs only when the worker wrote `report.md` **and** the serving's input holds a requester's post (p3 guard `0421c95`). It then posts the result, marks the task `completed`, runs `push_main` (every commit the shared `main/` has that Gitea does not), `record_task_in_devlog` (`git add -A` in the shared devlog, commit, push), and starts the next task | `serve_run`, `push_main_repository`, `commit_all_and_push` |
| Mission acceptance | `agentchat accept` / `accept.flag` / the completion door write `[state] accepted` on each finished task and `done` on the mission, with evidence. It refuses while a task is open | pyagag `agag.acceptance` |
| Cancellation | `cancel.flag` cancels the tasks, marks the mission cancelled and archives its work channel. Nothing touches the files. A cancelled task's topic refuses to run (`run_target` raises) | `handle_superdirector_response` |
| Serving concurrency | One executor thread per listener, so autolab's servings are serial. Missions interleave *between* servings: E∥F, G∥H and I∥J∥K in p3 alternated in one tree | pyagag `agag.listen` |
| Services | All six listeners, the relay and the autolab gateway are running under launchd. `nctl drift`: converged=46, every agent `polling`, `autolab-agautolab1` `stale` as before (the VM runs the gateway only, no listener and no missions) | launchd, nctl |

## Contamination reproduced

**Live (p3 trials).**
- E (m10311, `--upper`) ∥ F (`--lower`) in `robustp1/main/wordcount.py`. E's worker found F's uncommitted edits before it started (#10360): "A commit of the tree as it stands would include both". The requester had to decide the commit scope (#10372). The worker then removed F's lines, committed `a326124` and put them back. For that it used a fixed temporary path, `/tmp/lowerbk`, which any other run could also use.
- F was cancelled. Its `--lower` edits stayed in `main/`. G's plan (m10427) had to say "first discard the uncommitted `--lower` changes left by a cancelled mission" (#10432).
- G's task 1 committed `9801e75` in its first serving, and the pre-guard close-out pushed it. H and K also committed before anybody agreed. The p3 guard now stops the *task* from closing, but the commit stays in the shared `main/`. The next close-out of **any** mission of the project then publishes it, because `push_main` pushes `origin/main..HEAD`.

**Fixture on the current code** (autolab's own `push_main_repository` and `commit_all_and_push` on scratch repositories. The scratch script is not committed):

| Case | Result |
|---|---|
| Two missions edit `wordcount.py` | E's `git diff` contains F's line (`True`) |
| F cancelled with dirty edits, G starts | G's worker sees ` M wordcount.py` |
| H checkpoints before agreement, then K's close-out publishes | "K's close-out carried 2 commits". origin/main now has `K accepted change` and `H checkpoint (not accepted)` |
| Devlog record | The `[AUTO] task 1 report` commit contains `m1-task-1/report.md` **and** `stray-note-from-another-mission.md` |

## What is shared

| Shared thing | Contaminates by | Kind |
|---|---|---|
| `main/` working tree and index | Another mission's uncommitted edits land in a worker's diff and in its `git commit -a` / `add -A` (E∥F, cancelled F → G) | Source |
| `main/`'s local branch | A commit made before agreement (G, H, K) is published by any later close-out, whichever mission it belongs to | Commits and publication |
| `direction/` | Same shape as `main/`. Workers and the planning run both write decisions there | Source (direction repository) |
| `devlog/` | `record_task_in_devlog` stages everything dirty (`add -A`), so a stray file enters another task's record commit | Records (devlog repository) |
| Pattern projects' extra repositories (`gentest-*`, `localize`, `publish`) | The same as `main/`. `mediagen/gentest-actionDatasets` has an uncommitted `datasets/cat_idle/meta.json` today, and `gentest-YuE2` is one commit ahead of its remote | Source and generated outputs |
| Ignored `.local/` inside a repository (raw generations, venvs) | Shared by every mission, never committed | Generated outputs and caches |
| Fixed temporary paths the worker chooses (`/tmp/lowerbk`) | Possible collision; not observed | Worker scratch |
| Generation directories (`.local/topics/<channel>/<topic>/<N>/`) | Not shared: one per serving | — |

All other repositories are clean and level with their remotes today (survey of the 16 project folders).

## The isolation unit: the mission

- **One working copy per mission** (`.local/missions/<slug>/m<id>/`), holding a git worktree of every repository in the project folder on branch `autolab/m<id>`. Each worktree starts from that repository's shared branch when the copy is first made. The copy is created and found again by the mission's id, so a restart or a repeated serving resumes it. Sequential tasks share it; independent missions never share files or an index. It lives outside the project folder, so a planning run that reads the project never sees another mission's unfinished work.
- **Refs are shared** (worktrees share one repository), so the copy's own configuration refuses pushes. What a worker commits stays on its mission branch.
- **The shared project folder holds integrated work only.** Planning still runs there, because planning is about the shared result. Anything a planning run writes into a repository there is committed as that plan's notes right after the run, so the folder stays clean for integration.
- Ignored local data (`.local/`, raw outputs, caches) is not carried into a mission's copy. The shared folder's path is in the prompt for reading it.

Why not per task: tasks of one mission build on each other and are reviewed in sequence. A per-task tree would need an integration between every task, and the requester would review work split into those parts. Why not per project with locking: the executor already serializes runs, yet E∥F still collided. The hazard is state left *between* runs, not two runs at once.

## The contract

| Outcome | Meaning | Who and when | Recorded as |
|---|---|---|---|
| **checkpointed** | Saved on the mission's branch or in its copy. Implies nothing about agreement | The worker, whenever it likes. The listener notes the copy's state after each serving that changed it | Commits on `autolab/m<id>`; `[selfnote][change] checkpoint …` in the task topic |
| **accepted** | The requester agreed that the task is done, and the content they agreed to is bound to exact commits | The task's close-out, on the requester's post in the serving's input (p3's guard) | `[selfnote][change] accepted <repo>=<commit> … #<evidence post>` |
| **integrated** | Those commits are in the project's shared branch | The listener, under a per-project lock, right after `accepted` | `[selfnote][change] integrated …` |
| **published** | The shared branch is pushed to its remote | The listener, for the repositories autolab created and publishes (`main`, `direction`, `devlog`) and any other the worker names in `publish.flag` | `[selfnote][change] published …` |
| **complete** (task) | Result posted, `[state] completed`. Only after the change is integrated and published | Close-out, last | `## Result` + `[state] completed` |
| **done** (mission) | The requester's recorded acceptance | `agentchat accept` (unchanged). It already refuses while a task is open, and a task is open until its change is integrated | `[selfnote][acceptance]` + `[state] done` |

**The decisions that authorize integration and publication already exist.** They are kept, and no approval round is added:
- The **task agreement** (the requester's post, checked in code since p3) authorized "commit" and the close-out's push of `main` and the devlog. It now authorizes the integration and publication of that task's change, bound to the commits that exist at the close-out.
- **Mission acceptance** still records the whole mission as done. It adds no integration of its own, because every task's change is already integrated when the task closes.
- A retry of an authorized, unchanged result needs no new approval: integration is recognized by commit ancestry, so a repeat does nothing.

**What returns for a decision:**
- The shared branch moved after the mission began, and the other change touched the same files. This covers both a textual conflict and a clean merge of overlapping files. The task stays open, and the worker updates its copy onto the shared branch so the requester can review the combined result.
- A shared checkout has uncommitted changes that nobody owns and that the integration would overwrite.

Changes to disjoint files are merged automatically, since each side was reviewed and neither touches the other.

**Cancellation and ending.**
- A cancelled or replaced mission's tasks already refuse to run. Integration re-reads the task and the mission first, so nothing of a cancelled mission is ever integrated.
- When a mission ends (its last task closed, cancelled or replaced), its copy is released: any uncommitted state is committed to its branch and the worktrees are removed. The branch stays, so no work is erased, and a later re-plan re-attaches it.

**Rollout.** Uncommitted changes that nobody can attribute in a shared checkout are set aside on a branch `autolab/set-aside-<date>` and the checkout is cleaned. They are never handed to the next mission. Unpushed commits in repositories autolab does not publish (`gentest-YuE2`, two `publish/`) are left alone and listed.

## Open to step 2–3 (not blockers)

- Pattern projects' extra repositories are published only when the worker names them (`publish.flag`). Until now a worker pushed them itself, as README_PROJECT.md says. It can no longer push from its copy, and the shared checkout only receives the change after its serving ends.
- The bmining director still works in the shared `direction/` clone and commits and pushes it itself. It runs alone in the executor and leaves the clone clean, so it is not a mission path and is left as is.
