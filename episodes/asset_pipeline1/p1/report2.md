# Report 2 — superdirector role and mission-flow rework (agautolab)

Status: **done**. `agautolab` suite green (87 passed, was 78).

The disposable per-topic coding workspace is gone. A mission is now planned by
a `superdirector` run in the persistent project folder, and Plane — not the
filesystem — holds the result from registration onwards.

## The role

`agents.toml` gained `[roles.superdirector]` on the existing `sonnet` profile.
No new profile and no new model: the role exists so that the planning agent
and the coding agent can be pointed at different backends later without
touching a call site. (`agent ≠ model`, `devdocs/README_DEV.md`.)

`role_run.ROLE_ALLOWED_TOOLS` gained `"superdirector": WORKING_ALLOWED_TOOLS`.
It writes `plan.md` and the task split, so it gets the writable grant rather
than `director`'s read-only one. Verified against the live config:

```
superdirector sonnet claude_code anthropic anthropic/claude-sonnet-5
writable: True
```

The `.local/agents.local.toml` overlay puts `front` and `coding` on the local
ollama profile; `superdirector` has no overlay entry, so it resolves to sonnet
on this Mac too — which is the braindump's "一番強いモデル".

## The flow

`handle_front_response()` before → after:

```
upsert_work(new_mission)      run_superdirector(.local/projects/<slug>/)
cancel_sub_works              upsert_work(title=mission, description=plan.md)
run_coding(<N>/coding/)       cancel_sub_works
register_task_files(<N>/…)    register_task_files(.local/projects/<slug>/)
                              clear_planning_files(.local/projects/<slug>/)
```

The reordering is forced, not cosmetic: the parent Work's description is the
superdirector's `plan.md`, which does not exist until it has run.

- `new_mission.md` is copied from the front's generation into the project
  folder — the folder holding `main/`, `direction/` and `devlog/` — and the
  run happens there, with cwd set to it. The `<N>/coding/` sibling directory
  is not created any more.
- Timeout is `SUPERDIRECTOR_TIMEOUT_SECONDS = WORK_TIMEOUT_SECONDS` (1200 s).
  The split it replaced ran on 600 s while seeing one file; this run reads a
  whole project, so it gets the work timeout. `CODING_TIMEOUT_SECONDS` is
  deleted.
- **Parent Work**: title from `new_mission.md`, description from `plan.md` —
  the *whole* file, heading included. Step 6 recovers the parent's description
  and treats it as `plan.md`, so a lossless round-trip is worth more than
  avoiding one duplicated heading in the Plane description field.
- A superdirector run that produced no `plan.md` raises `ListenerError`
  before anything reaches Plane. Half a registration is worse than none: the
  topic gets `failed during response handling: …` and the files stay for a
  retry.

## `[Asset]`

`mission.strip_asset_marker(text) -> (text, bool)` matches `[Asset]`
case-insensitively at the very **end** of a `task[N].md` and removes it. The
marker is addressed to the registration, not to the coding run that reads the
description later, so it never reaches Plane; the label does.

`mission.ensure_issue` grew a `labels` parameter — label *names*, not ids, with
`AUTO` prepended and de-duplicated. Turning a name into an id on first use is
`ensure_label`'s job (process-wide cache, mission.py:199), and the caller that
knows a task is an asset should not have to know how a Plane label is born.
`register_task_files` passes `["asset"]` for a marked file and reports it as
`created sub-work PD-5 "Sprite sheet" [asset]`.

A mention of `[Asset]` in the middle of a task file is prose and is left
alone — pinned by a parametrized test.

## Cleanup

`clear_planning_files()` removes `new_mission.md`, `plan.md` and every
`task[N].md` from the project folder, and runs **only** after
`register_task_files` returns. The project folder is persistent: unlike the
front's `<N>/` generations, no generation number separates one planning round
from the next, so leaving the files there would let a later round re-register
a split the superdirector did not write.

On failure they stay, as specified — a retry then finds the plan instead of
paying sonnet for it a second time.

## Also in this step

`ZulipError` is now imported in `zulip_listener.py`. `main()` has caught it
since before this episode without importing it, so the initial
channel-subscription failure path would have raised `NameError` instead of
logging. Flagged in report1, absorbed here because this step already touches
the module's imports.

## Tests (87 passed, +9)

`tests/test_zulip_listener.py` — `wire_response()` now takes `tmp_path`,
patches `PROJECTS_ROOT`, and its superdirector stub writes what a real run
leaves behind (`plan.md` plus the task split) so the cleanup and the
description are exercised against real files:

- `test_new_mission_plans_in_the_project_folder_and_registers_the_plan` —
  call order is superdirector → upsert → cancel → register, the run's cwd is
  the project folder, and the Work is `("Build it", <whole plan.md>)`
- `test_the_planning_files_are_cleared_once_plane_has_them` — after a
  two-task round the folder holds only `main/`; the front's generation keeps
  its `new_mission.md`
- `test_a_failed_registration_leaves_the_plan_for_a_retry` —
  `new_mission.md`, `plan.md`, `task1.md` all survive a Plane failure
- `test_a_superdirector_that_wrote_no_plan_is_a_failure` — raises, and no
  Plane call is made

`tests/test_mission.py`:

- `test_strip_asset_marker_only_matches_the_tail` — five cases incl. the
  prose one
- `test_register_task_files_labels_an_asset_sub_work` — against `FakePlane`:
  the asset issue carries `[AUTO, asset]`, the plain one `[AUTO]`, and the
  marker is absent from `description_html`

## Known edge, accepted

If a round fails *after* the superdirector ran and the next round's
superdirector writes **fewer** task files, the leftovers from the failed round
are still in the folder and would be registered alongside the new ones. This
is the direct cost of "on failure leave them for retry" and the plan chooses
that trade deliberately; the superdirector also sees those files in its cwd
and can overwrite them. Recorded rather than fixed.
