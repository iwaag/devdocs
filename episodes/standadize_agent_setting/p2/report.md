# Phase 2 report — agforge first adoption

Date: 2026-08-10.

Standardized the agforge runner for the `generator` agent — handoff candidate.

## Outcome

agforge now resolves its `generator` role through `ag.agent-config.v1` and
has no legacy backend selector or arbitrary-command compatibility path. Both
standard harnesses completed the same representative image request locally.
The project-owned charter, result-file behavior, OpenCode permissions, and
Claude Code allowlist remain in place.

## Implementation

- Added committed `agforge/agents.toml` with canonical models, the
  `local-coder`, `sonnet-coder`, and test-only `stub` profiles, and the
  `generator` role.
- Added the stdlib-only Python loader in `service/agent_config.py`. It validates
  the contract error classes, applies the narrow local overlay, checks role
  capabilities, selects the newest `command_glob` match, checks executable
  availability, and never falls back.
- Added the git-ignored `.local/agents.local.toml` for this machine's harness
  paths and Ollama endpoint. Removed the superseded agent/backend and dead
  interpretation/Ollama variables from `.local/.env`.
- Renamed the runner branches to canonical `opencode`, `claude_code`, and
  `fake`. OpenCode receives the full canonical model ID; Claude Code receives
  the provider-native name derived by removing `anthropic/`.
- `opencode.json` retains the existing agforge tool grants and reads its
  provider endpoint from an environment value injected from the resolved
  local overlay. Claude Code retains `CLAUDE_ALLOWED_TOOLS`; neither harness
  uses an unrestricted native-host mode.
- Each request now writes a neighboring `.agent-run.json` record in addition
  to its useful `.agent.jsonl` transcript. Records contain request ID, role,
  profile, harness, provider, canonical model, outcome, duration, and reported
  cost/usage/turns. Failure records include the failure text.
- Updated the developer entry point, capability guide, service logging, and
  local development notes to use profile/harness/model vocabulary only.

## Verification

Deterministic verification:

- `uv run pytest -q`: **45 passed**.
- Coverage includes all nine contract rejection classes used by
  Phase 1, overlay profile precedence, newest command-glob selection,
  per-harness model argv mapping, binary/endpoint `E_UNAVAILABLE` without
  fallback, ordinary service behavior, raw transcript retention, and
  canonical run records.
- `git diff --check`: passed.
- A repository search found none of the deleted variables or legacy
  `backend` record/settings keys outside the test that explicitly proves the
  key is absent.
- OpenCode's resolved configuration showed the overlay endpoint and canonical
  default model, proving the project `opencode.json` environment reference.

Live representative request for both harnesses:

| profile | harness | canonical model | result | duration | turns | reported cost | evidence ID |
|---|---|---|---|---:|---:|---:|---|
| `local-coder` | `opencode` | `ollama/qwen3.6:35b-a3b-coding-nvfp4` | image delivered | 72,578 ms | 5 | $0.0000000 | `4863a1f88dd047568ba1afa66e8d1602` |
| `sonnet-coder` | `claude_code` | `anthropic/claude-sonnet-5` | image delivered | 39,791 ms | 13 | $0.2448357 | `505f3abe7c534b9a8421b7203f4a6abd` |

The local ignored evidence is under `agforge/.local/out/` as
`<evidence-id>.agent.jsonl` and `<evidence-id>.agent-run.json`. Generated
image URLs are intentionally omitted because they are expiring signed URLs.
After the Claude Code run, the local role override was removed, restoring the
committed `local-coder` default.

## Contract findings

No Phase-1 contract change was necessary.

- The unresolved provider-endpoint choice now has a working first-adopter
  answer: agforge injects the overlay value into the child process, and its
  OpenCode config references that value. This avoids modifying global
  OpenCode configuration and makes endpoint/model selection inspectable.
- Provider-prefix handling is sufficient as specified: OpenCode kept
  `ollama/...`; Claude Code successfully used the derived unprefixed native
  model name.
- The `fake` profile plus a local harness command cleanly replaces
  `AGFORGE_AGENT_CMD`; no special fake-harness contract extension was needed.
- Usage payloads differ naturally by harness. OpenCode step token reports are
  aggregated into input/output/reasoning/cache fields; Claude Code's richer
  reported usage object is retained without inventing a cross-harness
  conversion.
- Tool-grant duplication remains project-owned. Live execution did not show a
  reason to move either permission vocabulary into the common contract.

## Remaining observations

- The Claude Code run cost more and used more turns but completed faster in
  this single sample. This is evidence from one request, not a selection rule.
- `command_glob` removes the stale versioned Claude binary path problem for
  this deployment. An absent matching binary produces `E_UNAVAILABLE`.
- No Nautobot/nctl query was run: Phase 2 changed only the local agforge runner
  and made no claim about cluster service placement or deployment state.
