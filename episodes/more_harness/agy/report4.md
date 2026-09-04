# Report 4 — agautolab `role_run`: every role on the bypass (2026-09-05)

Step 4 of `plan.md`. In **agautolab `f2ec660`**.

## What landed

- `skip_permissions` is now true for `agent.harness in ("claude_code",
  "agy")`, plus gemini's rule as before. `harness_args` returns `None` for
  agy: there is no read-only door to hand `summarizer` — headless agy
  auto-denies reads as well as commands, and `--mode plan` writes a plan
  file into `~/.gemini/…/brain/` instead of answering — so the read-only
  role gets the bypass too, and its `allowed_tools = "Read,Glob,Grep"`
  stays as the statement of what it is expected to reach for. The module
  docstring says so.
- Tests: `test_agy_roles_all_run_on_the_bypass_including_the_readonly_one`
  (coding and summarizer both get `skip_permissions=True`,
  `extra_args=None`) and `test_the_agy_profiles_are_declared_and_resolve`.

agfront's listener needed nothing: the harness defaults to the bypass when
the caller names no `--mode`, and front's listener names none.

## Result

agautolab 216 tests pass (214 before).
