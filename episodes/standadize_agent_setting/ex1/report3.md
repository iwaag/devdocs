# Step 3 report — migrate agforge

## Result

agforge is now a hatchling `src/`-layout package and consumes `pyagag`.

- Added `src/agforge/` modules for agent runs, the request service, transforms,
  and generation mechanics.
- Replaced the PEP 723 service scripts with thin compatibility launchers, so
  `service/serve.sh`, `scripts/generate.sh`, and existing charter tool paths
  continue to work.
- Removed the duplicated `service/agent_config.py` and all `sys.path` glue.
- Preserved agforge-local charter composition, HTTP behavior, result-file
  precedence, Claude tool allowlist, and `AgentRunError` compatibility.
- Reworked the run seam to resolve through pyagag, call its non-raising
  `run_harness`, inspect `HarnessResult.meta["outcome"]`, and translate failures
  at the agforge boundary.
- Added the editable `../../pyagag` uv source, made `uv.lock` tracked, and
  regenerated it.

The latent Anthropic secret bug is fixed: pyagag resolves secret references to
`ResolvedAgent.environment`, and the shared harness merges that environment
into the subprocess. An agforge integration test proves the resolved value
reaches the child process without exposing a real secret.

## Verification

`uv run pytest -q` in agforge: **58 passed**.

The legacy `uv run service/agent_run.py` launcher was also exercised and
returned its expected usage error, proving the launcher reaches the installed
package.

No external model or in-system agent was invoked during this step. The
implementation run was served by the Omni Agent through the Codex harness
(GPT-5 family); no cost or token figures were exposed by the harness.

Omni Agent migrated agforge for the project agents — handoff candidate.
