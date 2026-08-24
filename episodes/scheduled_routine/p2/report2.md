# scheduled_routine p2 — Step 2 report: devlog record without git

agautolab commit "devlog record without git: push only when devlog/ is a
clone; bmining declines on main-only"; pj-agdev pin bumped; listener
restarted with `launchctl kickstart -k`.

## Diff summary

`src/agautolab/zulip_listener.py`

- `record_task_in_devlog`: writes `work.md` and `report.md` exactly as
  before, then **returns before the git helper when `devlog/.git` does not
  exist**: `recorded <label>-<title>/task-<N> in devlog locally (not a
  repository)`. The clone case is unchanged (`… and pushed` /
  `… (nothing to commit)`). No `git init`, no fallback repository — the
  MUST NOT line.
- `serve_bmining`: after `init_project`, if `direction/.git` is absent it
  answers `NO_DIRECTION_REPLY` ("This project has no `direction/` repository
  (it was set up main-only) … ask in a `workplan-` topic instead") and runs
  neither the director nor git. Cheaper than a guard inside the run and it
  says why.
- Nothing else touched. The grep for `devlog_directory`,
  `direction_directory`, `commit_all_and_push` shows the only other reaches
  are the superdirector serving (runs in the project folder, reads clones
  as folders — fine with a plain `devlog/` and no `direction/`) and
  `project_archive` (already returns `absent` for a missing Gitea repo).
  `init_project(slug)` from every serving now infers the layout (Step 1),
  so a serving cannot re-clone what the flag left out.

`agent/guides/workplan_superdirector/guide.md`,
`agent/guides/workrun_supercoder/guide.md`: "The folder `direction/`, if
exists …", "`devlog/`, if exists …" — these were already edited in the
working tree before p2 started (the plan says the guides "already say" it);
committed here so the deployed text matches the plan.

`tests/test_zulip_listener.py`

- New `test_a_local_only_devlog_is_written_and_never_pushed`: `devlog/` is
  a plain directory; the report lands in `task-2/report.md`, no `push`
  call is recorded, no `.git` appears, the topic still resolves.
- New `test_bmining_declines_on_a_main_only_project`: no director run, no
  push, the reply names the missing `direction/`.
- `wire_run` / `wire_bmining` now leave what a real full `init_project`
  leaves — a `devlog/.git` / `direction/.git` marker — unless the test made
  the folder main-only first (`project_init.is_main_only`). Without that the
  existing "records it and pushes" test would have read as main-only.

## Test output

```
$ uv run pytest -q
........................................................................ [ 82%]
..............................                                           [100%]
174 passed in 0.80s
```

(Step 1: 16 in `test_project_init.py`; this step: +2 in
`test_zulip_listener.py`; the rest unchanged.)

## Note

`WORK_TIMEOUT_SECONDS` in the listener is **1200**, not the 3600 the plan
states; the p1-style cadence choice in Step 4 uses 1200 s per task as the
figure.
