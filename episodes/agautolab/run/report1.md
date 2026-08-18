# run rework — Step 1 report: the `supercoder` role

Done. `supercoder` exists as a role and resolves to the `sonnet` profile.

## What changed

| File | Change |
|---|---|
| `agautolab/agents.toml` | new `[roles.supercoder]`, `profile = "sonnet"`, `requires = []` |
| `agautolab/src/agautolab/role_run.py` | `ROLE_ALLOWED_TOOLS["supercoder"] = WORKING_ALLOWED_TOOLS` |
| `agautolab/tests/test_role_run.py` | the role resolves and carries the working grant |

## Why a role, not a profile

The braindump says "agent profile に supercoder を追加" but structurally
this is a **role** on the existing `sonnet` profile, exactly like
`superdirector`: profiles name a harness+model pair, roles name a job.
Same profile as `coding` today; the separate role is what lets the two
diverge later without touching call sites.

## Why the allowlist entry matters

`build_argv` omits `--allowedTools` entirely when a role has no entry, and
a non-interactive claude_code run then waits for a permission answer until
the timeout. `supercoder` writes code in the project folder and `report.md`
in the serving workspace, so it gets `WORKING_ALLOWED_TOOLS` — the same set
`coding` and `superdirector` use.

## Verification

`uv run pytest -q` — 147 passed (146 before, +1).

`coding` and `director` are now unused-but-kept, per the plan.
