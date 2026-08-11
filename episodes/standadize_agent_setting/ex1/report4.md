# Step 4 report — document the contract

## Result

Added `pyagag/docs/agent-config-v1.md` as the package-local, self-contained
language-neutral contract reference. It documents:

- committed `agents.toml` versus git-ignored local overlay ownership;
- role → profile → harness/model resolution and precedence;
- the canonical model-ID format and v1 harness vocabulary;
- intrinsic and deployment capabilities;
- secret references and subprocess environment behavior;
- all stable `E_SCHEMA` through `E_UNAVAILABLE` error classes;
- the non-raising `HarnessResult` seam;
- the `ag.agent-run.v1` required and optional fields.

The document explicitly identifies agdevworld's
`assistant/agent-config.mjs` as a sibling JavaScript implementation of the
contract, not a consumer of the Python package. The pyagag README links the
new reference.

No external model or in-system agent was invoked during this step. The
documentation run was served by the Omni Agent through the Codex harness
(GPT-5 family); no cost or token figures were exposed by the harness.

Omni Agent documented the shared contract for the project agents — handoff candidate.
