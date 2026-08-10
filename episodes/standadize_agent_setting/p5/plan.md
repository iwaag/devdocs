# Phase 5 plan — agdevworld Claude Code parity

Add the `claude_code` process branch to the agdevworld front role behind the
same `agents.toml` config and MCP tool boundary that Phase 4 built for
OpenCode. Destructive phase, no backward compatibility — but note the main
deletion already happened: Phase 4 removed the direct Anthropic Messages path,
the `claude` backend selector, and the `@anthropic-ai/sdk` dependency. Phase 5
is mostly *addition plus proof*, closed by a grep gate confirming the deletion
held.

Read first: `../p4/report.md` (what exists and why),
`../p3/report.md` §"Contract findings" and §"Permission and prompt findings"
(the live Claude Code facts), `agautolab/src/agautolab/harness.py` (working
Claude argv and JSON extraction to port), and `agdevworld/assistant/harness.mjs`
(the file you will extend — `buildArgv` currently throws
"not implemented by Phase 4" for anything but `opencode`/`fake`).

Hard constraints (the roadmap's minimum set — everything else is your call):

- no credentials or generated private payloads committed;
- no silent harness/model fallback — an unavailable selected harness fails
  with its contract code or honest run error;
- keep raw agent output next to normalized records;
- no `--dangerously-skip-permissions` or equivalent unrestricted mode.

Environment facts you will need:

- `agents.toml` already declares everything: profile `sonnet-front`
  (`claude_code` + `anthropic/claude-sonnet-5`) and the model entry with
  `effort`/`max_tokens`. Those model options are advisory — agautolab's
  harness never wired them into argv; ignoring them here too is fine.
- The loader (`assistant/agent-config.mjs`) already validates `claude_code`,
  rejects non-`anthropic/` models for it, resolves the command (default
  `claude`, overlay `command`/`command_glob`), and raises `E_UNAVAILABLE`
  when the binary is missing. The native model name is the canonical ID with
  the provider prefix stripped: `anthropic/claude-sonnet-5` →
  `claude-sonnet-5` (records keep the canonical form; see
  `agautolab/src/agautolab/agent_config.py:47`).
- P3-proven Claude facts: argv is
  `claude -p --output-format json --model <native model>` with the prompt on
  stdin; output is one JSON envelope with `result`, `total_cost_usd`,
  `usage`, `num_turns`, `is_error`, `subtype`, `duration_ms` —
  `harness.py:100-113` (`_extract_claude`) is the extraction to port. Don't
  reshape the `usage` payload to match OpenCode's; both projects record each
  harness's own counts. P3 sonnet runs took 2.6–84 s and cost $0.03–0.57.
- This Mac's binary lives behind the glob agautolab already uses — copy the
  overlay line from `agautolab/.local/agents.local.toml`:
  `command_glob = "/Users/eiji/.vscode-server/extensions/anthropic.claude-code-*-darwin-arm64/resources/native-binary/claude"`.
  Host auth is the subscription login in `~/.claude/.credentials.json`; no
  API key env is needed for native runs. The assistant *container* has
  neither the binary nor credentials — see Step 3.
- MCP is the new territory: P3's Claude roles used built-in tools, never MCP.
  Claude Code takes `--mcp-config <file>` (add `--strict-mcp-config` so the
  developer's global MCP servers stay out) and exposes tools as
  `mcp__agdevworld__<tool>`, so
  `--allowedTools "mcp__agdevworld__fetch,mcp__agdevworld__wait,mcp__agdevworld__switch_view,mcp__agdevworld__show_image"`.
  Claude-spawned MCP servers inherit the parent env, so
  `AGDEVWORLD_TOOL_BASE_URL` and the per-run `AGDEVWORLD_ACTIONS_FILE` that
  `harness.mjs` already sets should reach `tool-service.mjs` unchanged —
  verify live, not by assumption. The service's plain JSON-RPC-over-stdio
  (protocol `2025-03-26`, unknown methods → `-32601`, id-less notifications
  ignored) worked for OpenCode; if Claude's client trips on it, fix the
  service, that's what harness-neutral means.
- Known trap (recorded agautolab failure mode): a headless `claude` loads
  user/project memory and settings keyed by its cwd, and this developer's
  harness-level project instructions have previously leaked into in-system
  agents and killed runs on permission denials. `harness.mjs` currently runs
  the child in the repo root. The front role needs no workspace — a scratch
  cwd (as the container naturally provides) or a deliberate check of what a
  native run actually inherits is worth doing before blaming the model for a
  weird refusal.
- Profile switching for live tests is contract-legal overlay vocabulary, not
  a new knob: `[roles.front] profile = "sonnet-front"` in the ignored
  `.local/agents.local.toml` (§6 allows a role override changing only
  `profile`). The roadmap explicitly wants *no* project-specific
  configuration vocabulary — resist adding a per-request harness parameter.
- Timeout mismatch to resolve while you're there: the server defaults to
  `AGDEVWORLD_AGENT_TIMEOUT_MS=900_000` but nginx cuts the client at
  `proxy_read_timeout 300s`. Align them in whichever direction the evidence
  supports; sonnet is fast enough that 300 s is comfortable.

## Step 1 — claude_code branch in the process seam

Extend `assistant/harness.mjs`: a `claude_code` branch in `buildArgv`
(argv + MCP/allowedTools flags, native model mapping, no skip-permissions
path at all), and a Claude JSON-envelope extractor next to
`extractEventText`, ported from `_extract_claude`. `is_error: true` or a
nonzero exit is a failed run with the envelope kept as the raw transcript;
an empty `result` keeps the existing "agent produced no text" behavior.

Decide where the MCP config lives — a committed
`assistant/claude-mcp.json` mirroring `opencode.json`'s server block is the
obvious shape; a generated temp file also works. Machine facts stay out of
it (the service reads env, so it should contain none).

Extend the tests in `assistant/tests/harness.test.mjs` with `node:test`
coverage of the new argv, envelope extraction (success, `is_error`, garbage
output), and cost/usage mapping. A stub executable that prints a canned
envelope keeps this deterministic; the `fake` harness profile still covers
the server loop.

## Step 2 — native live smoke on the Mac

Add the claude glob to `.local/agents.local.toml`, flip
`[roles.front] profile = "sonnet-front"` there, run the assistant natively
(`node assistant/server.mjs` with `AGDEVWORLD_TOOL_BASE_URL` pointing at a
running web container or dev server), and drive `/api/chat`:

- one plain no-tool chat — proves resolution
  `front → sonnet-front → claude_code → anthropic/claude-sonnet-5` and a §9
  record with the envelope's cost/usage;
- one tool run that must call `fetch` and `switch_view` — proves the MCP
  handshake, tool naming, env inheritance to the tool service, and action
  collection through the same per-run file.

Expect the first attempt to fail somewhere (MCP flag spelling, tool-name
prefix, permission denial, inherited settings). Keep those transcripts —
they are the phase's findings. Fix at the harness/config layer, not with
prompt rules.

## Step 3 — availability and auth made explicit

The roadmap requires binary *and authentication* availability to be
explicit, with clear failure and no fallback.

- Binary: already explicit — missing command is `E_UNAVAILABLE` at resolve
  time and surfaces as the 502 `assistant_offline` detail. Demonstrate it
  once (point the overlay at a nonexistent path, keep the record).
- Auth: a present binary with no credentials fails at run time; make sure
  that failure's stderr tail lands in the record rather than vanishing, and
  document in `README_DEV.md` what each environment needs (host: `~/.claude`
  login; container: `ANTHROPIC_API_KEY` or a mounted credential — supplied
  via compose env/secret reference per contract §"local.secrets", never
  committed).
- Container: your judgment. Installing `@anthropic-ai/claude-code` in the
  assistant image (same pinning style as OpenCode `1.18.10`) plus a
  compose-supplied key gives full containerized parity now; declaring
  claude_code native-only until Phase 6's deployment reconciliation is also
  acceptable *if* the limitation is stated in README_DEV.md and the selected
  harness still fails loudly in the container. Record which you chose and
  why in the report.

## Step 4 — parity evidence

With both profiles available, run the same representative task set Phase 4
recorded, once per harness (OpenCode via `local-front`, Claude via
`sonnet-front`, switched only by the overlay role override):

- ordinary chat;
- the multi-step fetch → wait → fetch autolab-nodes comparison;
- view switch plus image presentation applied by the browser.

Evidence to keep: one record + raw transcript per request; both harnesses'
records carrying the same normalized identity/outcome field names with their
own honest usage payloads; at least the Claude leg exercised from a real
browser session. Diff the behavior, don't sand it down — turn counts,
latency, cost, tool-calling style, and prose differences go in the report as
findings, not into harness-specific prompt branches.

Per the local-environment instruction, snapshot `nctl status --json` before
and after live service work and note (only) unrelated drift — P4 found
Nautobot itself down; that precedent is fine to record again if it persists.

## Step 5 — cleanup gate and report

- Grep gate: no `@anthropic-ai/sdk`, `askClaude`, `ANTHROPIC_URL`-style
  selector envs, or Messages-API code anywhere in agdevworld code, compose,
  or docs (today the only legitimate `anthropic` hits are `agents.toml` and
  the loader's provider tables — keep it that way).
- `npm test` and `npm run build` pass; if you touched the image, the compose
  build and `/healthz` check pass too.
- Update `README_DEV.md` (harness matrix, auth requirements, timeout story)
  and `assistant/GUIDE.md` only if tool vocabulary changed (it shouldn't).
- Write `report.md` in this directory in the P3/P4 shape: outcome,
  implementation, verification table with record IDs, contract findings
  (did the MCP boundary hold for a second harness — the roadmap's core
  question), behavioral-difference findings, and the constraint check.
