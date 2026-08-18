# run rework — retest report: channel folder + `main/` commit

Both questions the retest asked are answered **yes, live**:
the `work-` channel inherits the project channel's folder, and the supercoder
now commits `main/` once the developer approves.

## Setup

| Piece | Value |
|---|---|
| Guide change | `4dd892b` on `iwaag/agautolab` `main`, submodule bumped in `pj-agdev` (`a933b08`), both pushed |
| Listener | agstudio `com.agdev.agautolab-zulip` kickstarted (pid 86457) |
| Channel folder | `Projects` (id 1) — created by the **Developer**, since folder creation is admin-only (the bots are role 400) |
| Project | `runsmoke2` via `init_project.py`; channel `pj-runsmoke2` created **with `folder_id=1`** and subscribers 8/9/11 |

Note on ordering: `create_channel(..., folder_id=)` only places a channel at
creation, and pyagag has no call that moves an existing channel into a folder.
So the folder has to exist before the `pj-` channel is created — which is why
this ran on a new project rather than on `runsmoke1`.

## Result

1. **Mission → surfaces.** The superdirector planned R2-1 with sub-works
   R2-2/R2-3, and the handler opened `work-r2-1` with
   `run-task1-r2-1` / `run-task2-r2-1`.
   **`work-r2-1` came out in folder `1`** — the same folder as `pj-runsmoke2` —
   with all three parent subscribers. `ensure_work_channel`'s folder
   inheritance is now proven live, not just in the unit test.
2. **Gate closed.** A post into `run-task2-r2-1` while R2-2 was unstarted got
   `Please complete previous work (R2-2)`; no agent run.
3. **Task 1.** First serving: created `main/README.md`, read `.gitignore`, and
   **stopped to ask for approval** instead of committing — exactly what the new
   guide line asks for. After the approval post, the second serving committed
   (`1535a5e Add placeholder README`), wrote `report.md`, and the handler
   completed R2-2, resolved the topic, and pushed `devlog/…/task-1/`.
4. **Gate open, task 2.** Same shape: `HELLO.md` written, approval asked,
   then `1803f8b Add HELLO.md smoke test greeting`, R2-3 completed,
   `✔ run-task2-r2-1`, `task-2/` pushed.

Final state: `main/` has two real commits and a **clean working tree** — the
gap the first smoke reported is closed. Plane shows R2-2 and R2-3 `completed`,
R2-1 `started`; the devlog clone is pushed; both run topics resolved; the
`work-r2-1` channel stays open (still next phase's decision).

## Two things worth carrying forward

- **`main/` is committed but never pushed** — the clone is `[ahead 2]` of its
  Gitea origin. The guide asks for a commit only, so this is behaving as
  written; whether the work should reach the origin (and whose job that is,
  agent or handler) is an open decision, exactly like the commit itself was.
- **The commit instruction pulls the agent toward the wrong repo.** Told to
  check `.gitignore`, the task-2 supercoder walked up into the *agautolab*
  checkout (`cd …/agautolab && git status`, `git check-ignore`) before finding
  that `main/` is its own repo. It recovered on its own and committed in the
  right place, but the project folder sits inside agautolab's own working tree,
  so a less careful run could commit into the wrong repository. Naming `main/`
  as the repository in the guide line would remove the ambiguity.

## Deus Ex Machina note

The Omni Agent created the `Projects` folder (as the Developer), created
`pj-runsmoke2`, posted the mission, both task triggers and both approvals —
did those for the Developer; handoff candidate for the parts that are not the
developer's own judgement (channel creation especially, since `init_project`
still does not create Zulip channels).
