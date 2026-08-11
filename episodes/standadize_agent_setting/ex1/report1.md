# Step 1 report — build `pyagag`

## Result

Created the installable `pyagag` package with the import namespace `agag`.
The package uses hatchling, a `src/` layout, uv, pytest, and Python 3.11+.

The shared configuration layer now owns:

- `ag.agent-config.v1` parsing and validation;
- role → profile → harness/model resolution;
- canonical harness and intrinsic-capability registries;
- executable and provider endpoint resolution;
- Anthropic secret references resolved into subprocess environment values;
- app-name-specific wording for the Ollama/OpenCode availability error.

The shared harness layer now owns:

- argv construction for OpenCode, Claude Code, and fake harnesses;
- non-raising `HarnessResult` outcomes;
- environment/provider endpoint injection;
- OpenCode JSONL and Claude JSON output extraction;
- transcript capture, ANSI removal, configurable output-tail length;
- `ag.agent-run.v1` record writing.

Relevant config and harness tests were ported from agautolab/agforge and extended
for fake argv handling, environment injection, configurable tails, and run
records.

## Verification

`uv run pytest -q` in `pyagag`: **19 passed**.

No external model or in-system agent was invoked during this step. The
implementation run was served by the Omni Agent through the Codex harness
(GPT-5 family); no cost or token figures were exposed by the harness.

Omni Agent built the shared package for the project agents — handoff candidate.
