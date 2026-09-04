# Report 4 — agautolab `role_run`: a real read-only door this time (2026-09-05)

Step 4 of `plan.md`. In **agautolab `5a604d5`**.

## What landed

- `codex_args(role)`: `["-s", "read-only"]` for `READONLY_ROLES`, `[]`
  otherwise; `harness_args` dispatches on `"codex"`.
- `skip_permissions` is true for codex **except** the read-only roles —
  the same shape as gemini's rule, because the bypass would turn
  `-s read-only` into `danger-full-access` in the harness. So `summarizer`
  runs in a sandbox where reads and shell commands work and writes fail
  with "operation not permitted" (the model reports why, as report1's
  probe showed); every other role gets full access, because
  `workspace-write` can neither commit (`.git` is protected) nor reach
  Zulip. The module docstring says so, next to agy's "no read-only door".
- Tests: `test_codex_readonly_role_gets_the_read_only_sandbox_and_the_rest_the_bypass`
  (`coding` → `skip_permissions=True`, `extra_args=[]`; `summarizer` →
  `False`, `["-s", "read-only"]`) and
  `test_the_codex_profiles_are_declared_and_resolve`.

agfront's listener needed nothing: the harness defaults to full access
when the caller names no sandbox, and front's listener names none.

## Result

agautolab 218 tests pass (216 before).
