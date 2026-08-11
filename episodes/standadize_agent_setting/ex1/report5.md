# Step 5 report — verification

## Test and build matrix

| project | full tests | focused fake/stub smoke | package build |
|---|---:|---:|---:|
| pyagag | 19 passed | covered by harness/config tests | sdist + wheel passed |
| agautolab | 80 passed | `run_once` converged in 3 fake iterations | sdist + wheel passed |
| agforge | 58 passed | fake HTTP generation request + environment injection passed | sdist + wheel passed |

The agautolab persisted role-run record was also switched to pyagag's shared
`ag.agent-run.v1` writer during final verification.

## Local service state

Because the plan mentioned the local request service, cluster state was checked
through `nctl status --json` before touching it. The result was `ok: true`:
Nautobot was reachable and authenticated with intent-catalog/GraphQL available,
one worker was running, and there were no pending jobs.

Nothing was listening on agforge's local port 8092, and `/healthz` was
unreachable. The optional "restart if running" check therefore did not apply;
no service process was stopped or started.

## Gemini test

A new harness such as `gemini_cli` requires these Python changes, all inside
pyagag:

1. Add the ID to `CANONICAL_HARNESSES`.
2. Add its set to `INTRINSIC_CAPABILITIES`.
3. Add its default executable to `_resolve_command()`.
4. Add any harness/provider compatibility rule in committed-config validation.
5. Add its argv protocol to `build_argv()`.
6. Add its output extraction and select it from `run_harness()`.

No Python edit is required in agautolab or agforge when the generic adapter/run
seams are sufficient. agdevworld remains a sibling implementation and would
need the corresponding edit in its single JavaScript agent-config/harness
implementation. This meets the episode's stated extensibility criterion.

## Agent record

No external model or in-system project agent was invoked during verification.
The work was served by the Omni Agent through the Codex harness (GPT-5 family);
no cost or token figures were exposed by the harness.

Omni Agent verified the shared package and both consumers for the project agents — handoff candidate.
