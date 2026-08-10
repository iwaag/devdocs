# Phase 3 report — agautolab role and runner conversion

Date: 2026-08-11.

Standardized agautolab's five agent roles for the in-system agents — handoff
candidate.

## Outcome

agautolab now resolves `front`, `director`, `mediator`, `coding`, and
`summarizer` through `ag.agent-config.v1`. Direct Ollama chat, environment
backend/model selectors, the Claude binary pointer, and the three duplicate
command resolvers are gone. OpenCode and Claude Code share one process seam
for command resolution, argv/model mapping, timeout behavior, protocol
extraction, raw output retention, and normalized run metadata.

A live mission completed the required front -> mediator -> coding path with a
canonical identity record at every boundary. Both standard harnesses ran:
OpenCode/Ollama served the front, while Claude Code/Anthropic served the
director, mediator, coding, and summarizer roles. The local gateway was
restarted on `:8791` with the new implementation; no agautolab node deployment
or Ansible work was performed because that belongs to Phase 6.

## Implementation

- Added committed `agautolab/agents.toml` with the shared `local-coder`,
  `sonnet-coder`, and test-only `stub` profiles and all five roles. `front`
  requires `nested_harness`, supplied as a declared project capability.
- Added the stdlib-only loader at `src/agautolab/agent_config.py`. It preserves
  the Phase-1 error codes, narrow overlay scope, newest `command_glob` match,
  capability checks, canonical provider/model split, and fail-loud behavior.
- Added the ignored `.local/agents.local.toml` for this Mac's OpenCode path,
  versioned Claude Code glob, and Ollama OpenAI-compatible endpoint. The old
  `.local/.env` selector and `.local/agent/claude_bin` pointer were removed.
- Added `src/agautolab/harness.py`, the common process seam. OpenCode receives
  the full canonical model ID and JSONL extraction; Claude Code receives the
  provider-native model name and JSON-envelope extraction. Both preserve raw
  output and emit the same identity/outcome fields without converting their
  different usage payloads.
- Added role-owned OpenCode permission files for front, read-only
  director/summarizer, mediator, and coding. The shared seam does not grant
  tools. Claude Code continues to receive each role's explicit
  `allowedTools`; the pre-existing coding `skip_permissions` option remains
  available but default-off.
- Added `src/agautolab/role_run.py` for front/director/mediator/summarizer
  one-shots. `agent/session.sh` is now a thin mediator wrapper that writes a
  raw transcript plus `session-NNNN.run.json`. The director CLI accepts
  `--prompt` so a front agent can perform a nested run as one permissioned
  command and record it.
- Added the OpenCode coding adapter and rebuilt the Claude Code adapter on the
  common seam. `job.yaml profile:` overrides the `coding` role profile; when
  absent, the role default applies. Model flags in `adapter_config.args` are
  rejected. Raw coding artifacts are now harness-neutral
  `agent_output.json` / `agent_output.jsonl`, and normalized fields merge into
  `adapter_result.json`.
- Converted gateway front and summarizer runs to the role runner. New window,
  director, session, summary, and coding records use `role`, `profile`,
  `harness`, `provider`, canonical `model`, `outcome`, duration, and reported
  cost/usage/turns. Session readers still tolerate archived pre-contract
  session documents.
- Updated `README.md`, `AGENT_GUIDE.md`, `agent/README.md`, `agent/GUIDE.md`,
  and `agent/CHARTER.md` to describe profiles, local overlays, canonical
  records, and `.local/projects/<name>/direction/`.
- Deleted the obsolete `src/agautolab/binpath.py` and rewrote the old
  backend-shaped tests rather than retaining compatibility branches.

## Verification

| check | result | evidence |
|---|---|---|
| Full deterministic suite | **88 passed** in 4.51 s | `cd agautolab && uv run pytest -q` |
| Contract examples | valid examples resolve; all nine invalid fixtures return their indexed codes | `tests/test_agent_config.py` |
| Both harness protocols and failures | JSONL/JSON extraction, cost/usage, launch failure, timeout, empty output, `is_error`, raw failure transcript | `tests/test_harness.py` |
| All five role selections | each role resolves; overlay role override and job override covered | `tests/test_agent_config.py` |
| Legacy deletion gate | production code/docs contain none of the Phase-1 legacy selectors and paths | `tests/test_legacy_removed.py` |
| Syntax/config hygiene | Python compile, JSON parse, `git diff --check` passed | local verification run |
| OpenCode resolved config | local endpoint and front-only permission document merged; canonical model visible | `opencode debug config` |
| Gateway health | restarted gateway answered `GET /healthz` on `127.0.0.1:8791` | live local service |
| nctl/Nautobot before | `ok: true`; Nautobot reachable/authenticated, one worker, zero pending jobs | operation `01KZP3J95B66QA2M57TN0317P0` |
| nctl/Nautobot after | unchanged healthy state | operation `01KZP4Z01PEFKGY74NR04KYZFY` |

### Live mission boundary

Mission run 14 (`.local/agent/gateway/run-0014.log`, exit `0`) produced:

| boundary | role/profile | harness/model | result | duration | reported cost | record |
|---|---|---|---|---:|---:|---|
| entrance | `front` / `local-coder` | `opencode` / `ollama/qwen3.6:35b-a3b-coding-nvfp4` | mission accepted | 42,996 ms | $0.000000 | `.local/agent/window/run-0058.json` |
| mediator | `mediator` / `sonnet-coder` | `claude_code` / `anthropic/claude-sonnet-5` | mission completed | 84,055 ms | $0.5673744 | `.local/agent/sessions/session-0030.run.json` |
| iteration | `coding` / `sonnet-coder` | `claude_code` / `anthropic/claude-sonnet-5` | converged, gate 1/1 | 7,360 ms | $0.0881255 | `.local/jobs/p3-profile-smoke/evidence/iter-0001/adapter_result.json` |

The coding job's local experiment definition is
`.local/jobs/p3-profile-smoke/job.yaml`; it uses `profile: sonnet-coder` and
contains no command or model spelling. `target/phase3.txt` contains
`phase3-ok`, `state.json` is `converged`, and the gate
`grep -qx phase3-ok phase3.txt` exited 0.

### Director and summarizer smoke

- The front's recorded nested director consultation succeeded at
  `.local/agent/window/run-0059.json`; the nested Claude Code identity is at
  `.local/agent/director/p3-front-director.json` (2,594 ms, 1 turn,
  $0.0286940), with raw output beside it.
- The coding iteration summary completed through the `summarizer` role at
  `.local/jobs/p3-profile-smoke/summaries/iter-0001.cost.json` (11,790 ms,
  3 turns, $0.0742559). The cached prose and raw transcript remain beside the
  record.

## Contract findings

No Phase-1 contract change was necessary.

- The Phase-2 provider-endpoint pattern worked here too: the local overlay
  value is injected into the child and project OpenCode config references the
  environment value. For Ollama's OpenAI-compatible provider this machine
  requires the endpoint suffix `/v1`; using the bare Ollama root produced a
  recorded 404 rather than fallback.
- Canonical-to-native model mapping remained sufficient: OpenCode used
  `ollama/qwen3.6:35b-a3b-coding-nvfp4`; Claude Code used
  `claude-sonnet-5` while records kept `anthropic/claude-sonnet-5`.
- A job-level profile override can change both harness and model without
  smuggling either through adapter arguments. Keeping adapter-local tool and
  directory knobs outside the common contract remains appropriate.
- Archived session readers need the same old-record tolerance described by
  contract section 9. New writers use only canonical fields.

## Permission and prompt findings

These are observations from the live attempts, not new broad rules.

- OpenCode and Claude Code express the same role authority differently.
  OpenCode needed project JSON permission patterns; Claude Code needed
  `allowedTools` arguments. Keeping both at the caller/role boundary allowed
  the read-only director and summarizer to avoid inheriting mediator/coding
  tools.
- The first front-directed director attempt invoked the nested runner without
  delivering stdin. Claude Code failed loudly with "Input must be provided"
  and recorded `.local/agent/director/p3-smoke.json`; the surrounding complex
  front request later hit its 300 s timeout (`window/run-0057.json`). Adding a
  `--prompt` form made the nested action one explicit permissioned command;
  the front-directed retry succeeded in `window/run-0059.json`.
- The first OpenCode attempt used the bare Ollama URL and preserved the 404
  event in `window/run-0056.agent.jsonl`. After changing only the ignored
  overlay to `/v1`, the same selected harness/model succeeded. No alternative
  harness or model was tried automatically.
- The successful OpenCode front runs used more turns and reported much larger
  input counts than the one-turn Claude director runs, and one front reply
  repeated status prose. These are isolated behavioral observations, not a
  selection policy or a reason for harness-specific prompt rules.

## Constraint check

- No credentials or generated private payloads were added to tracked files;
  machine paths and endpoints remain in ignored local configuration.
- No silent harness/model fallback exists. Availability and selection errors
  retain their contract codes or harness failure words.
- Raw stdout/transcripts remain next to normalized records, including the
  failed live attempts.
- No new unrestricted native-host mode or auto-approval flag was added.
  OpenCode command grants are explicit; the existing default-off
  `skip_permissions` coding option was not enabled.
- `nctl status` was checked before and after the local service work. Nautobot
  remained healthy, and no cluster desired state, node deployment, or
  unrelated service placement was changed.

Ran the direct `p3-smoke-2` director rehearsal for the front agent — handoff
candidate.
