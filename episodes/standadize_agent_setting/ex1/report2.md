# Step 2 report — migrate agautolab

## Result

agautolab now consumes `pyagag` through the `agag` import namespace.

- Removed the duplicated `src/agautolab/agent_config.py` and `harness.py`.
- Added the app-local `agent_settings.py` wrapper for repository paths and
  `resolve_project_role()`.
- Updated role runners, adapters, gateway, and tests to import shared config
  and harness primitives from `agag`.
- Kept adapter policy and the registry inside agautolab.
- Added an editable uv path source and regenerated `uv.lock`.
- Removed the unit tests ported to pyagag while retaining app-level config and
  run-loop integration coverage.

The plan's suggested `../../../pyagag` path was verified and found to point one
directory too high. The committed dependency uses the correct sibling path,
`../../pyagag`.

## Verification

`uv run pytest -q` in agautolab: **80 passed**.

The suite includes the fake-profile `run_once` integration path; the explicit
final smoke is repeated in Step 5.

No external model or in-system agent was invoked during this step. The
implementation run was served by the Omni Agent through the Codex harness
(GPT-5 family); no cost or token figures were exposed by the harness.

Omni Agent migrated agautolab for the project agents — handoff candidate.
