# agent_mindmap — Phase 2 plan: entrances and guides

AI-generated (Omni Agent, 2026-08-09), from `../roadmap.md` Phase 2.
Experimental environment: no backward compatibility required, minimal
prohibitions, maximal implementer discretion. Everything below that is
not a prohibition is a hint, not a requirement.

**Goal**: autolab has a conversational window; all four entrances
(agforge, assistant, cagent, autolab) answer "what can you do / what
does it cost" in some form; backends are selectable and runs recorded
per `devpolicy/agent_records.md`.

## Prohibitions (complete list)

1. Don't modify executor.
2. Secrets and cluster payloads stay out of git (existing rules).
3. Existing irreversible-destruction confirmations (nctl
   `--allow-destroy` class) stay.

Everything else — API shapes, file layouts, endpoint removal — is the
implementer's choice.

## Steps

### 1. autolab conversational window

Add one desire-accepting free-text route to `agautolab/agent/gateway.py`
(e.g. `POST /window {"text": ...}`). Behavior in words:

- job/progress questions → answer from the same job state the typed GETs
  already expose (the jobs/status helpers are all in the same file —
  reuse them, don't re-read the dirs).
- development requests → reply that `POST /mission` with bearer auth is
  the door.
- capability/cost questions → answer from the guide file (step 3).

Placeholder quality is fine. Terrain:

- `gateway.py` is a stdlib-only single file (~900 lines); staying
  dependency-free is nice, not required. Routes and auth policy are
  documented in its module docstring; only `POST /mission` requires the
  bearer token (`.local/agent/gateway_token`). An unauthenticated
  window is consistent with the current read side; it spends money like
  the summarize route, which is already accepted unauthenticated with a
  one-at-a-time guard — copy that guard if desired.

### 2. Window backend

Small local model via ollama by default, switchable per Agent ≠ Model.
Copy the `AGFORGE_AGENT_BACKEND` pattern —
`agforge/service/agent_run.py:local_env()/agent_backend()` (process env
first, then `.local/.env`, default `ollama`) — rather than inventing a
new mechanism or a shared library. Record each window run per
`devpolicy/agent_records.md` (id, backend, outcome, cost/time when
reported, failure text in the agent's own words); the job `evidence/`
dirs or a sibling under `.local/` are both fine locations.

### 3. Guide files (capability cards)

Give each of agforge, assistant, cagent, autolab a small plain-text
capability card answering "what can you do / what does it cost"
(tentative numbers and "unknown" fine — absence of the form is not,
per `devpolicy/policy.md` Entrance Guide).

Precedent to copy: cagent serves `src/cagent_api/static/llms.txt` and
re-reads it from disk per request (`server.py:_load_llms_txt`) — plain
file, no restart needed. Its content is also a good model for tone:
what it is, endpoints, timing. Suggested (not required) placements:
next to each service's code, committed, one page each.

### 4. Wire guides into existing entrances

- **agforge**: its request service answers guide questions at its
  normal window (service runs on agstudio `:8092`, `service/serve.sh`).
- **assistant**: `agdevworld/assistant/server.mjs` `handleChat` — the
  guide can simply be appended to / referenced from `ROLE_PROMPT`
  context, or served like cagent's llms.txt; implementer's choice.
- **cagent**: via its existing `llms.txt` / opencode `AGENTS.md`
  instructions mechanism — likely just editing text files.
- **autolab**: the step-1 window reads the card.

### 5. Backend selectability sweep (Agent ≠ Model on what exists)

Process/config-level switching is enough; per-request switching
optional. Same `AGFORGE_AGENT_BACKEND` pattern everywhere; each run
recorded per Phase 1 policy.

- **assistant**: implement the commented "engine-agnostic seam"
  (`server.mjs` top comment; today `handleChat` is the only code that
  knows ollama, config via `OLLAMA_URL`/`OLLAMA_MODEL` env). Ollama
  stays default; a strong backend becomes selectable (e.g.
  `ASSISTANT_BACKEND=ollama|claude`). Upgrading the engine is optional;
  only the switch + guide Q&A form is required.
- **cagent**: model currently hardcoded in
  `cagent/opencode/config.json.template` (`"openai/gpt-5.6-luna"`,
  applied by `opencode/start.sh`). Expose as explicit, documented
  config; per-request switching not required.
- **agforge**: already conforms (`AGFORGE_AGENT_BACKEND`,
  `AGFORGE_OPENCODE_MODEL`) — verify one run per backend, document,
  done.
- **autolab delegate**: already conforms via adapters
  (`job.yaml adapter:` + `adapter_result.json`) — nothing to do.

## Known terrain / gotchas (read before touching agautolab1)

- **Deploy path** for agautolab1 is agstudio Gitea + Ansible — the exact
  two commands are in `pj-agdev/.local/devenv.md` ("Updating an autolab
  node"). The node may run a stale checkout (it has before — that's why
  its `/jobs` still demanded auth on 2026-08-08): **push to gitea
  first, then run the playbook**. Ansible is the only channel; plain ssh
  is refused.
- `agautolab1.local` has resolved to a different address than Nautobot's
  desired one (0.220 vs 0.130) — if the node misbehaves, check actual
  state via nctl/nautobot before debugging code.
- **macOS quirk**: a bare `node`/local process may get `EHOSTUNREACH` to
  LAN hosts while curl succeeds (local-network privacy). Test the
  assistant through its Docker container (`localhost:8091`), never via
  bare `node assistant/server.mjs`.
- The agstudio gateway and agforge service are hand-started native
  processes (start commands in `pj-agdev/.local/devenv.md`); jobs
  vanish on restart — restarting them mid-test is cheap and safe.
- cagent's server doesn't implement `HEAD`; probe with plain GET.
- Long waits: keep command waits under ~1 minute; a stuck deploy or
  probe gets killed and diagnosed, not waited out.

## Out of scope / future work

- Completion *notification* (vs polling) — note as future work if
  skipped, per roadmap.
- Comparing/exploiting run records; auth redesign; director (Phase 3).

## Acceptance (from roadmap)

- Asking each of the four entrances "what can you do?" and "what does N
  cost?" returns a sensible sentence (placeholder ok).
- Asking autolab's window about a real job returns real state; asking
  it to build something returns the `/mission` redirect.
- Sweep: each agent's docs name the backend switch, and flipping it on
  one sample run demonstrably changes the backend recorded in that
  run's record.

Write the phase report to `p2/report.md` (ENT).
