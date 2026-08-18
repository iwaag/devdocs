# run rework — retest with a channel folder

The `run` episode's live smoke ran in a realm with **zero channel folders**,
so `ensure_work_channel` inherited `folder_id = None` — the inheritance itself
was only ever proven by a unit test. This retest runs the same end-to-end
smoke on a fresh project whose `pj-` channel **sits in a real folder**, and
picks up the guide change that closes the gap the first smoke reported
(nothing ever committed `main/`).

## What changed since the first smoke

`agent/guides/run_supercoder/guide.md` now ends with: commit changes in
`main/` after getting the developer's approval, minding `.gitignore`. So the
supercoder — not the handler — owns the source commit, gated on the same
approval that gates `report.md`.

Deployment note: a folder can only be set **at channel creation**
(`create_channel(..., folder_id=)`); pyagag has no call that moves an existing
channel into a folder. Hence a new project rather than reusing `runsmoke1`.

## Steps

1. Roll out the guide change: commit, push to GitHub, bump the `pj-agdev`
   submodule, restart the agstudio listener.
2. Realm setup: create channel folder `Projects` (admin-only, so as the
   Developer), then create `pj-runsmoke2` **with that folder** and the same
   three subscribers as the first smoke (Developer, autolab bot, Omni).
3. `init_project.py runsmoke2`.
4. Post the two-task mission into `pj-runsmoke2/mission-hello-file`.
   **Check:** `work-r2-1` is created in folder `Projects`, with the parent's
   subscribers, and one `run-task<N>-r2-1` topic per task.
5. **Gate closed:** post into `run-task2-…` first → `Please complete previous
   work`, no agent run.
6. **Task 1:** post the go-ahead, let the supercoder work, then approve.
   **Check:** `report.md` in the serving workspace, Sub-Work commented and
   completed, topic resolved, devlog `task-1/` pushed — and, new this time,
   **`main/` carries a commit** with a sane `.gitignore` and no junk.
7. **Gate open:** re-post into `run-task2-…` → task 2 runs and closes the same
   way, `main/` committed again.

## Out of scope

Everything the first episode deferred stays deferred: how a finished `work-`
channel gets closed, the re-plan-after-done path, and whether the handler
should push `main/` to its Gitea origin (the guide asks for a commit only).
