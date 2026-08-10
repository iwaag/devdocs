# Phase 4 plan — agdevworld OpenCode front and tool bridge

Replace agdevworld's direct Ollama chat path with one OpenCode process per
`/api/chat` request, move the multi-round tool loop from the browser into the
agent run, and return UI-only operations as actions for the browser to apply.
This is a destructive phase: the `/api/chat` wire contract, the browser tool
loop, and the direct Ollama code are all rewritten or deleted with no
backward compatibility. Browser and server change together in one phase.

Read first: `devpolicy/contracts/agent/spec.md` (the contract),
`devpolicy/contracts/agent/examples/valid/agdevworld/agents.toml` (a ready
example config for exactly this phase), `../p3/report.md` (harness findings
that apply here, especially the Ollama `/v1` endpoint), and
`agautolab/src/agautolab/harness.py` (working OpenCode argv/extraction code
to port).

Hard constraints (the roadmap's minimum set — everything else is your call):

- no credentials or generated private payloads committed;
- no silent harness/model fallback — fail with the contract's error codes;
- keep raw agent output next to normalized records for failed-run diagnosis;
- do not add `--dangerously-skip-permissions`, `opencode run --auto`, or an
  equivalent unrestricted mode.

Environment facts you will need:

- agdevworld is TypeScript/Vite/Phaser in front, plus a stdlib-only Node
  service `assistant/server.mjs` (639 lines, no framework, entrypoint of the
  assistant container). Compose: `web` (nginx, 8090), `assistant` (8091,
  volume `assistant_records:/records`), `dev` (vite, 5173). nginx proxies
  `/api/` to the assistant with `proxy_read_timeout 300s` (`nginx.conf`).
- The only automated check today is `npm run build` (`tsc && vite build`);
  there are no tests. The assistant's own deps are installed only in the
  container (`assistant/package.json`, currently just `@anthropic-ai/sdk`).
- `/api/chat` today (`server.mjs:399-450`): request
  `{messages, context}`, response `{reply, tool_calls, backend}`. The
  **browser** drives the loop: `src/chatPanel.ts:209-273` re-POSTs after
  executing each tool round, up to `MAX_TOOL_ROUNDS = 16`.
- Four tools are declared server-side (`server.mjs:60-102`) but all execute
  in the browser: `fetch`, `switch_view`, `wait`, `show_image`. There are no
  separate autolab/forge/notes tools — those are HTTP paths reached through
  `fetch` and enumerated in `ROLE_PROMPT` and `assistant/GUIDE.md`.
- Reach boundaries already live in the assistant's HTTP routes, not in the
  tool layer: the finite `AUTOLAB_NODES` map with `unknown_node` 404
  (`server.mjs:514-519`) and the raw-evidence 403 (`EVIDENCE_PATH`,
  `server.mjs:355, 520-525`). Route tool traffic through these same routes
  and the boundaries stay at the tool boundary for free.
- The direct Ollama path to delete: `askOllama` (`server.mjs:168-214`),
  `toOllamaMessages`, env `OLLAMA_URL` / `OLLAMA_MODEL`, and the
  `ASSISTANT_BACKEND` selector. The `claude` backend (`askClaude`, Anthropic
  SDK) is Phase 5's deletion target — leave it out of the new path; it may
  sit unreachable or be deleted early if that turns out cheaper, your call.
- P3-proven harness facts: OpenCode argv is
  `opencode run --format json -m <full canonical model ID>`; the Ollama
  OpenAI-compatible provider on this Mac needs the endpoint suffix `/v1`
  (bare root produced a recorded 404, no fallback); provider endpoints
  travel as `AGENT_PROVIDER_OLLAMA_BASE_URL` referenced from the project's
  OpenCode config, and `OPENCODE_CONFIG` pins the config file. JSONL events
  carry `text`, `error`, and `step_finish` with cost and token counts —
  `agautolab/src/agautolab/harness.py:57-91` is the extraction reference.
- The proven local model is `ollama/qwen3.6:35b-a3b-coding-nvfp4`. Ollama
  serves `127.0.0.1:11434` on the host; from a container that is
  `host.docker.internal` (compose already maps it for the assistant).
- `.gitignore` has `*.local` and `public/cluster/`, but **not** `.local/` —
  add it before creating the overlay.

## Step 1 — config, loader, and run records

Add committed `agdevworld/agents.toml` — start from the contract example:
profiles `local-front` (opencode + the qwen model) and `sonnet-front`
(claude_code + `anthropic/claude-sonnet-5`, dormant until Phase 5), a
test-only `fake` profile, `[capabilities] provides = ["ui_actions"]`, and
`[roles.front] requires = ["ui_actions"]`. Add the git-ignored
`.local/agents.local.toml` with `[local.provider.ollama] base_url`
(remember `/v1`) and the local `opencode` command path.

Write a JS loader (suggested: `assistant/agentConfig.mjs`) with the same
semantics as `agautolab/src/agautolab/agent_config.py`: schema marker,
canonical harness/model validation, profile/role resolution, §6 overlay
scope, capability check, and the stable `E_*` error codes. Copy-and-adapt,
don't build a shared library — deliberate duplication between projects is
the established pattern. Node has no built-in TOML parser; a small
dependency such as `smol-toml` in `assistant/package.json` is fine, as is
vendoring a minimal parser. Validate against
`devpolicy/contracts/agent/examples/` (valid examples resolve, invalid ones
return their indexed codes) — this doubles as agdevworld's conformance test.

Extend `recordRun` (`server.mjs:326-335`) to contract §9: `role`, `profile`,
`harness`, `provider`, canonical `model`, `outcome`, `duration_ms`,
`cost_usd`/`usage`/`num_turns` when reported, `failure` on failure. Delete
the composite `backend_model`. Keep the one-line stdout record and the
`ASSISTANT_RECORDS_DIR` file write; also save each run's raw OpenCode
stdout/JSONL beside the record so a failed run is diagnosable.

There is no test runner yet; `node --test assistant/*.test.mjs` with the
built-in `node:test` is enough. The `fake` harness profile exists so loop
and action-collection logic can be tested without a live model.

## Step 2 — OpenCode process seam

Add the process runner (suggested: `assistant/harness.mjs`): spawn one
`opencode run --format json -m <model>` per chat request, feed the prompt on
stdin or argv, apply a timeout, capture raw output, and extract reply text,
error, cost, and token usage from the JSONL events. Port the shape from
`agautolab/src/agautolab/harness.py` (argv at line 36-54, extraction at
57-91, env injection at 126-130); `agforge/service/agent_run.py:174` has a
lenient reader if events surprise you.

Prompt assembly: the stateless contract survives intact — per request,
compose `ROLE_PROMPT` + screen `context` + `GUIDE.md` + the serialized
browser-owned history into the fresh run's input. OpenCode has no separate
system-prompt argument in this mode; folding everything into the single run
prompt (with clear section markers) is acceptable, as is an
instructions-file mechanism if you find one that works — verify by live
experiment, not assumption.

Give the run a fixed scratch working directory inside the container (the
front role needs no workspace), and a project OpenCode config that grants
only what the front needs — the MCP tools from Step 3, no file editing, no
shell. `agforge/opencode.json` and agautolab's role-owned permission files
are working references for the permission vocabulary.

Timeout advice: the whole request must fit under nginx's
`proxy_read_timeout 300s`, and the multi-step tasks it must survive include
autolab summarize (~15 s per poll) and forge requests (20–105 s). A run
timeout around 240 s with the nginx value raised if evidence demands it is a
reasonable start; raising nginx instead of shrinking the task is fine.

Checkpoint: a plain no-tool chat through `/api/chat` resolves
`front → local-front → opencode → ollama/qwen3.6:35b-a3b-coding-nvfp4`,
answers, and writes a §9 record. P3 saw ~43 s for an OpenCode front turn on
this model — budget your patience accordingly.

## Step 3 — tool service and UI action bridge

Move tool execution into the agent run through a harness-neutral boundary.
The preferred shape is a small MCP service owned by the assistant, because
Phase 5 must hand the same tools to Claude Code — but if live
experimentation shows a simpler reliable route, take it and record why.

Server-capable tools: `fetch` (HTTP against the assistant's own routes —
`/api/autolab/*`, `/api/forge/*`, `/api/note`, `/api/guide` — plus the
static `/cluster/*.json` and sample files), `wait` (bounded; the browser
clamped it to 60 s, keep some bound). Whether you keep one generic `fetch`
tool or split named `autolab`/`forge`/`notes` tools is your judgment; the
model-facing descriptions in `ROLE_PROMPT`/`GUIDE.md` must match whatever
you choose.

Two routing facts to solve deliberately: (1) `/api/*` calls from the tool
service should loop back to the assistant itself (or call its handlers
in-process) so `unknown_node`, `evidence_not_proxied`, and `node_offline`
keep firing exactly as today; (2) `/cluster/*.json` is served by the `web`
nginx container, not the assistant — from the assistant container reach it
via the compose network (e.g. an `ASSISTANT_WEB_URL` default
`http://web:80`), and mind that dev-mode (vite, no docker) differs.

UI-only tools: `switch_view` and `show_image` do not execute server-side.
Their tool implementation appends `{name, arguments}` to a per-run action
buffer and returns an immediate acknowledgement string to the model (e.g.
"the view switch was queued for the browser"). After the run ends, the
buffer is returned with the final answer. Since each chat request owns
exactly one OpenCode process, run-scoping the buffer is easy — a per-run
temp file, an in-process map keyed by run ID passed through the MCP
transport, or a per-run stdio MCP process all work; pick what stays
debuggable. Note the model can no longer observe the *result* of a view
switch mid-run; the browser applies actions after the answer. Rewrite the
tool descriptions and GUIDE.md so the model expects that.

Test the loop deterministically with the `fake` profile (stub harness that
emits scripted tool calls), then live: a real OpenCode run that calls fetch
and switch_view.

## Step 4 — wire contract and browser rewrite

Change `/api/chat` to: request `{messages, context}` where messages are only
`user`/`assistant` prose (the `tool_calls`/`role:"tool"` wire variants die);
response `{reply, actions: [{name, arguments}], ...identity fields}` after
the server-side run completes. Errors keep their current envelope style;
`assistant_offline` becomes whatever the harness failure honestly is.

Browser (`src/chatPanel.ts`): delete the `for (round…)` loop, `runTool`,
`runFetch`, `runWait`, `MAX_TOOL_ROUNDS`, the legacy
`extractAssistantActions` inline-action path, and the tool message types.
`send` becomes one POST; on response, render `reply`, then apply each action
(`switch_view` via `src/main.ts` / `viewSwitcher.ts`, `show_image` via the
existing image-bubble code, which stays). Add a modest client timeout that
matches the server budget (the current 60 s `FETCH_TIMEOUT_MS` is now far
too short for a multi-step run). A "thinking…" indicator is worth the few
lines given runs can take minutes.

Server: delete `askOllama`, `toOllamaMessages`, `OLLAMA_URL`,
`OLLAMA_MODEL`, `ASSISTANT_BACKEND`, and the `BACKENDS` registry; harness
selection now flows only from `agents.toml` role resolution. Update
`ROLE_PROMPT`, `assistant/GUIDE.md`, `README_DEV.md` (its "10 s fetch
timeout" line is already stale), and the boot log. `npm run build` must pass.

## Step 5 — container and deployment wiring

The assistant image must contain the pinned OpenCode runtime and the
generated project OpenCode config. Watch out: the base is `node:26-alpine`
(musl) — if the OpenCode binary distribution doesn't run there, switching
the assistant base image to a glibc node image is a perfectly acceptable
destructive change. Verify `opencode --version` inside the image as a build
or startup check. The Ollama endpoint inside the container is
`http://host.docker.internal:11434/v1`, supplied via the overlay/env, never
committed.

Decide where the overlay lives in the container (bind-mount of
`.local/agents.local.toml`, env-generated file at startup, or compose env
vars feeding a generated overlay — your choice; keep machine facts out of
the image and out of git). `compose.yaml` and `assistant/Dockerfile` change
freely.

Checkpoint: `docker compose up --build -d web assistant` succeeds,
`/healthz` answers, and a containerized chat round-trips through OpenCode.

## Step 6 — live evidence, cleanup, report

Collect the roadmap's phase evidence in a live browser session (dev or
compose, but at least one full compose run):

- ordinary chat answered by OpenCode;
- a multi-step task: e.g. list autolab nodes, start a summarize, poll until
  it resolves — one `/api/chat` request, one recorded run, multiple
  tool calls inside it;
- a view switch and an image presentation applied by the browser from
  returned actions;
- the record trail showing one `assistant.run.v1`-style record per request
  with §9 identity fields, plus raw output beside it;
- confirmation (grep) that `askOllama`, `OLLAMA_URL`, `OLLAMA_MODEL`, and
  `ASSISTANT_BACKEND` are gone from code, compose, and docs.

Also verify the reach boundaries still bite from inside the agent run: an
unknown autolab node and an `evidence/` path fetched by the model must come
back as the same refusals, recorded in the raw transcript.

Write `report.md` in this directory following the P2/P3 shape: outcome,
implementation, verification table with evidence paths, contract findings
(did `ui_actions` and the MCP boundary hold up? anything Phase 5 should know
before pointing Claude Code at the same tools?), and the constraint check.
Keep failed live attempts and their raw output — they are findings, not
embarrassments.
