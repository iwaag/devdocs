# Phase 4 report — agdevworld OpenCode front and tool bridge

Date: 2026-08-11.

Standardized the agdevworld `front` agent and moved its tool loop into one
fresh OpenCode run per request — handoff candidate.

## Outcome

agdevworld now resolves `front` through `ag.agent-config.v1` and starts one
OpenCode process for every `POST /api/chat`. The browser still owns and sends
the complete conversation history, screen context, and current guide, but it
no longer performs model tool rounds. The OpenCode run completes its own MCP
loop and returns final prose plus collected UI actions in one HTTP response.

The direct Ollama `/api/chat` implementation, direct Anthropic Messages path,
legacy backend/model selectors, composite `backend_model` record, and browser
inline-action/tool-loop compatibility code are gone. Phase 5 therefore only
needs to add the `claude_code` process branch against the same config and MCP
boundary; there is no API-native Claude compatibility path to preserve.

The production-style web and assistant containers were rebuilt and remain up
on `:8090` and `:8091`. A live OpenCode run exercised ordinary chat, a
three-call fetch/wait/fetch sequence, view switching, and image presentation.

## Implementation

- Added committed `agdevworld/agents.toml` with canonical `local-front`,
  `sonnet-front`, and test-only `stub` profiles. `front` requires the
  deployment-provided `ui_actions` capability.
- Added the stdlib-only JavaScript contract loader in
  `assistant/agent-config.mjs`. It validates the Phase-1 error classes,
  overlay scope, capabilities, command/command-glob availability, canonical
  provider/model identity, and fails without harness/model fallback.
- Added ignored native and compose overlays. They supply only the OpenCode
  executable and Ollama OpenAI-compatible `/v1` endpoint; no machine endpoint
  or credential entered committed agent configuration.
- Added `assistant/harness.mjs`. It creates a fresh process, supplies the full
  browser history as prompt input, bounds wall time, retains raw JSONL, extracts
  OpenCode text/cost/usage/turns, and returns normalized run metadata.
- Added `assistant/tool-service.mjs`, a dependency-free stdio MCP service with
  `fetch`, `wait`, `switch_view`, and `show_image`. `fetch` stays on the
  configured agdevworld origin, supports only GET/POST, has a 60-second bound,
  and clips a response at 1 MB. `wait` caps one call at 60 seconds.
- Server-capable tools execute during the OpenCode process. UI-only tools
  append typed actions to a per-run temporary channel; the server reads those
  actions after process exit and removes the temporary directory.
- Existing HTTP reach boundaries remain authoritative: the autolab node map
  is finite and raw `/evidence/` paths are still refused by the passthrough.
  They were not converted into prompt prohibitions.
- Simplified `src/chatPanel.ts` to one `/api/chat` call. It records only user
  and final assistant messages, displays the reply, then applies returned
  `switch_view` and `show_image` actions. The 16-round browser loop and legacy
  inline JSON action parser were deleted.
- Added project OpenCode configuration with only the agdevworld MCP tools
  enabled for this role; built-in filesystem, shell, and web tools are denied.
  The provider declares the canonical local model explicitly, so the container
  does not depend on a developer's global OpenCode config.
- The assistant image installs the locally proven OpenCode `1.18.10`, copies
  project config, and mounts the ignored compose overlay. Final deployment
  pinning/reconciliation remains Phase 6 work.
- Run records now contain `role`, `profile`, `harness`, `provider`, canonical
  `model`, `outcome`, `duration_ms`, and reported cost/usage/turns. Each has a
  neighboring `<id>.agent.jsonl` raw transcript in the durable records volume.
- Updated `README_DEV.md`, `assistant/GUIDE.md`, compose configuration, and
  assistant package metadata. The unused direct Anthropic SDK dependency was
  removed.

## Verification

| check | result | evidence |
|---|---|---|
| Deterministic suite | **19 passed** | `npm test` |
| Contract conformance | valid agdevworld fixture resolves; all nine invalid/overlay fixtures return indexed error codes | `assistant/tests/agent-config.test.mjs` |
| Process seam | JSONL extraction, full browser history, normalized fake run, raw transcript, timeout=`aborted` | `assistant/tests/harness.test.mjs` |
| MCP boundary | tool catalog, same-origin refusal, raw HTTP response, stdio handshake, and UI-action collection | `assistant/tests/tool-service.test.mjs` |
| Frontend build | TypeScript and Vite production build passed | `npm run build` |
| Container build | web and assistant images built; assistant contains OpenCode `1.18.10` | `docker compose up --build -d web assistant` |
| Application checks | web `/` and assistant `/healthz` returned HTTP 200; both containers remained up | local compose checks |
| Browser render | Chromium loaded the production page at 1280x800 with cluster canvas and chat entrance visible | local `/tmp/agdevworld-phase4.png` (not committed) |
| Legacy deletion | no old selector env vars, direct-provider functions, `backend_model`, or inline action parser found | repository `rg` gate |
| Diff hygiene | passed | `git diff --check` |

The existing Vite warning about the 1.4 MB Phaser-containing main chunk remains
non-failing and unrelated to this phase.

## Live OpenCode evidence

Records are in the ignored Docker `assistant_records` volume. Every listed
request produced one normalized record and one raw OpenCode JSONL transcript.

| task | result | duration / turns | record ID |
|---|---|---:|---|
| ordinary chat after final rebuild | exact `phase4-final-ok`, no actions | final record succeeded | `acb9eace-a5a3-48df-aaff-f633b75dce93` |
| fetch -> wait 1 s -> fetch | both `/api/autolab/nodes` results compared; no reachability change | 19,271 ms / 3 | `7db7f5f8-4922-4b84-8d60-cee9d7d8fa29` |
| UI view + image | `switch_view(autolab)`, `show_image(/favicon.svg)`, final `phase4-actions-ok` | 35,382 ms / 2 | `97c12c5a-0df4-41fd-986a-7bf5179fabf9` |

The multi-step raw transcript contains, in order,
`agdevworld_fetch`, `agdevworld_wait`, and `agdevworld_fetch`, including both
raw node responses. The UI transcript contains completed
`agdevworld_switch_view` and `agdevworld_show_image` calls, while its run record
contains `actions: ["switch_view", "show_image"]`. OpenCode reported zero cost
for all local Ollama runs; no dollar value was invented.

The first container attempt failed loudly because the project OpenCode config
had provider options but no model declaration. Its error transcript and failed
record remain under ID `4efb3663-0a4d-4cf4-a346-088cdbb1f21e`. Adding the model
to the project config made the image independent of the host's global config;
no alternative harness or model was attempted.

## Browser and tool-loop findings

- The local model used MCP reliably for both server and UI operations. Tool
  names were exposed as OpenCode's `agdevworld_<tool>` prefix, while the
  harness-neutral service itself retains the plain four-tool vocabulary.
- Intermediate OpenCode text events are retained and concatenated. In the
  fetch/poll run the model emitted “Now I'll make the third fetch...” before
  the last call, so that sentence appeared before its final comparison. This
  is useful raw behavior evidence, not a reason to hide intermediate output or
  add harness-specific prompt rules.
- Action runs varied from roughly 35 to 101 seconds with the same prompt/model.
  The browser now waits for one complete HTTP response rather than imposing a
  per-round browser timeout, so this latency does not recreate the old
  browser-owned loop.
- MCP had to be configured in the project image along with an explicit model
  declaration. A provider endpoint alone worked on the Mac only because the
  global OpenCode config happened to declare that model; the clean container
  exposed the missing project fact immediately.

## Contract findings

No Phase-1 contract change was necessary.

- `ui_actions` accurately describes the external browser channel without
  putting action schemas or browser behavior into the common agent contract.
- Canonical `ollama/qwen3.6:35b-a3b-coding-nvfp4` passes unchanged to OpenCode.
- A fresh process per request preserves the stateless browser-history contract
  while still permitting multi-round reasoning inside the harness.
- `sonnet-front` can be declared now but fails clearly in Phase 4 because the
  Claude Code runner is not implemented; no silent OpenCode substitution exists.

## Local service state

Per the local-environment instruction, `nctl status --json` was checked before
and after service verification. Operations `01KZP98D74MHMTJNQ7DQNXYP0Q` and
`01KZP9V4T0MCYTCS0MAE64XV17` both reported Nautobot unreachable with
“Server disconnected without sending a response.” `docker ps` showed the
Nautobot web container restarting/health-starting and its worker/scheduler in
restart loops during this window. This is unrelated pre-existing local service
state; consequently no drift result was available.

No Nautobot data, desired state, node deployment, Ansible target, or external
service was changed. agdevworld's own `web` and `assistant` containers were
healthy after their rebuild.

## Constraint check

- No credential or generated private payload was added to tracked files.
- Harness/model availability fails loudly; there is no fallback.
- Raw successful and failed OpenCode output remains beside normalized records.
- No `--auto`, unrestricted native-host mode, or permission bypass was added.
- Reach and resource limits are enforced by HTTP/MCP boundaries, while prompt,
  UI choice, polling judgment, and final wording remain agent-owned.
