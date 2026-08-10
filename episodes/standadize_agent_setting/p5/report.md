# Phase 5 report — agdevworld Claude Code parity

Date: 2026-08-11.

Added Claude Code parity to the agdevworld `front` role and proved both
standard harnesses against the same MCP tool boundary — handoff candidate.

## Outcome

agdevworld can now resolve either `local-front` (OpenCode + Ollama) or
`sonnet-front` (Claude Code + Anthropic) through the same
`ag.agent-config.v1` role. Both paths start one fresh process per browser
request, receive the complete browser-owned history, execute the same four MCP
tools, return the same UI-action shapes, and write the same normalized identity
and outcome fields. There is no per-request backend vocabulary and no fallback.

The direct Anthropic Messages API path and `@anthropic-ai/sdk` had already been
removed in Phase 4; the Phase 5 cleanup gate confirms they did not return.

## Implementation

- Added a `claude_code` branch to `assistant/harness.mjs` using
  `claude -p --output-format json --model claude-sonnet-5`.
- Generated a machine-neutral MCP config inside each run's temporary directory,
  selected it with `--mcp-config --strict-mcp-config`, and allowed only
  `mcp__agdevworld__fetch`, `wait`, `switch_view`, and `show_image`.
- Ran Claude from the per-run scratch directory so checkout-scoped user/project
  memory is not inherited. The generated MCP entry uses the current Node
  executable and the resolved project tool-service path; no machine fact is
  committed.
- Added Claude JSON-envelope extraction for `result`, reported duration, turn
  count, cost, and the native usage payload. `is_error`, nonzero exit, empty
  output, and timeouts remain explicit failures; stderr tails are retained in
  failure records.
- Kept the raw Claude JSON envelope beside each normalized record. OpenCode
  continues to retain raw JSONL events in the same neighboring-file convention.
- Fixed the JavaScript contract loader's `command_glob` implementation to
  expand wildcard directory components. This was needed for the documented VS
  Code extension path, not just a wildcard in the final filename.
- Aligned the assistant bound to 300 seconds and nginx to 310 seconds, allowing
  the assistant to record and return its own timeout before the proxy closes.
- Updated `README_DEV.md` with the harness matrix, profile selection, native
  Claude authentication, explicit failure behavior, and container limitation.
- Kept the image OpenCode-only for this phase. Claude Code binary/auth deployment
  remains Phase 6 work; selecting `sonnet-front` in the current container fails
  with `E_UNAVAILABLE` rather than using OpenCode.

## Verification

| check | result | evidence |
|---|---|---|
| Deterministic suite | **24 passed** | `npm test` |
| Claude process seam | native model argv, strict MCP/allowed-tools flags, success/error/garbage extraction, native usage/cost, stderr failure, raw output | `assistant/tests/harness.test.mjs` |
| Command glob | wildcard parent directories resolve and newest executable wins | `assistant/tests/agent-config.test.mjs` |
| Frontend build | TypeScript and Vite build passed | `npm run build` |
| Container build | web and assistant images rebuilt successfully | `docker compose up --build -d web assistant` |
| Application checks | web `/` and assistant `/healthz` returned HTTP 200; final production-style OpenCode request succeeded | record `8d30ebb2-6c68-4cf2-83ca-d4231fa4c97e` |
| Real browser | Chromium submitted a Claude request through Vite, displayed the reply, applied `switch_view`, and loaded `/favicon.svg` | record `7c7a8a0b-627e-4b77-b655-8c89149aeb9e` |
| Container unavailability | selected Claude profile returned HTTP 502 with `E_UNAVAILABLE`; no fallback | record `89f7cff1-187f-4e2e-89d1-87ed7bdeea3e` |
| Legacy deletion | no SDK dependency, `askClaude`, Anthropic URL selector, Messages API call, or legacy `claude` backend remains | repository `rg` gate |
| Diff hygiene | passed | `git diff --check` |

The existing non-failing Vite advisory for the 1.4 MB Phaser-containing main
bundle remains unrelated.

## Live parity evidence

Native evidence is stored in the ignored `.local/phase5-records/` directory;
container records are in the durable `assistant_records` volume. Each
successful request has a normalized record and neighboring raw harness output.

| task | Claude Code | OpenCode |
|---|---|---|
| ordinary chat | `d2367559-7145-4851-9443-ed4bbee8fecb` — 2,186 ms, 1 turn, $0.0741204 | `39f0568b-f69e-499f-b8bf-802701fdaacb` — 18,337 ms, 1 turn, $0 |
| fetch → wait 1 s → fetch | `59ed4da0-f8e4-4162-a40b-c82bbdd56386` — 12,249 ms, 5 turns, $0.1283683 | `7dd50b3c-6d0f-49f5-82fb-1839230ff277` — 19,870 ms, 4 turns, $0 |
| switch view + show image | `9cb75ce5-4217-45bc-8695-43565a2342ee` — 5,982 ms, 4 turns, $0.1024548 | `489996b3-3b39-4776-ae5d-4af52498cf71` — 14,783 ms, 2 turns, $0 |

Both poll runs observed the same local state twice: `agstudio` was unreachable
and `agautolab1` returned HTTP 200. Both action runs returned exactly
`["switch_view", "show_image"]`. Claude's live envelopes reported no
permission denials.

## Behavioral findings

- Claude completed this small matrix faster, but used more turns for the two
  tool tasks and reported nonzero subscription-account cost. OpenCode's local
  Ollama provider reported zero cost; no value was inferred.
- Claude's `--output-format json` produces one rich result envelope rather than
  OpenCode's event-by-event JSONL. The normalized record therefore keeps the
  same identity/outcome field names while `usage` intentionally retains each
  harness's native shape.
- The existing MCP service worked unchanged for Claude Code. Environment values
  for the same-origin base URL and per-run action file reached the Claude-spawned
  server, and strict MCP selection did not require a harness-specific prompt.
- The first native attempt failed before launch with
  `E_UNAVAILABLE` (`d5021ae1-6c40-4676-bbc4-1f40c0323354`) because the JS loader
  only expanded the basename portion of `command_glob`. Preserving that failure
  exposed a real cross-project conformance gap; component-wise expansion fixed
  it without adding an alternate path or fallback.

## Contract findings

No Phase-1 contract change was necessary.

- Canonical-to-native mapping was sufficient:
  `anthropic/claude-sonnet-5` remained in records while Claude received
  `claude-sonnet-5`.
- The `ui_actions` capability and four-tool MCP boundary held for a second
  harness without introducing project-specific selection vocabulary.
- `command_glob` already allowed the documented versioned binary path; the
  adopter loader needed correction to implement that existing meaning.
- Normalized identity/outcome fields are directly comparable even though raw
  transcripts and usage payloads remain harness-native as intended.

## Local service state

The pre-work `nctl status --json` operation
`01KZPBGY8E66R9YBYJASFC7XFN` found Nautobot unreachable. At the user's
direction, the documented Nautobot compose stack was started. Its logs then
identified the stopped external `service_scripts-redis-1`; restarting that
existing Redis service allowed the Nautobot web, worker, and scheduler
containers to become healthy.

The final `nctl status --json` operation
`01KZPC50Z9CJ5ZD41YWJD51KYS` returned `ok: true`: Nautobot 3.1.3 was reachable
and authenticated, intent catalog/GraphQL were present, one Celery worker was
running, and no pending jobs were reported. No desired state, Nautobot data,
node deployment, Ansible target, or external service was changed.

## Constraint check

- No credential or generated private payload was added to tracked files.
- Missing binary and authentication/process failures are explicit; no harness
  or model fallback exists.
- Successful and failed raw harness output remains beside normalized records.
- Claude runs use an allowlist and strict MCP config. No permission bypass,
  OpenCode `--auto`, or equivalent unrestricted native-host mode was added.
- Behavioral differences are recorded here as observations; no harness-specific
  guidance was added to force identical behavior.
