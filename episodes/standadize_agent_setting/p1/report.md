# Phase 1 report — common contract and conformance examples

Date: 2026-08-10. Executed by the Omni Agent (contract authorship at
workspace level; no in-system agent owns this task, so no handoff note
is due).

## Outcome

`ag.agent-config.v1` is defined and documented at
`devpolicy/contracts/agent/`, with valid and invalid conformance
examples covering all known roles and the failure classes the roadmap
names. No runner was changed; no scripts, runtime data, or
machine-specific values were placed in the contract directory.

## Deliverables

- `devpolicy/contracts/agent/README.md` — orientation, contract-only
  rule, versioning, how projects consume the examples.
- `devpolicy/contracts/agent/spec.md` — the specification: TOML shape;
  canonical harness IDs (`opencode`, `claude_code`, test-only `fake`);
  canonical model IDs (`<provider>/<name>`, declared in `[models]`);
  profiles; roles (all eight known roles named); committed
  `agents.toml` vs git-ignored `agents.local.toml` overlay with a
  strict overlay scope; capability declaration
  (`requires`/`provides`, names in use: `ui_actions`,
  `nested_harness`, `service_http`); ten stable error codes with
  fail-loud/no-silent-fallback semantics; normalized run-metadata
  fields extending `devpolicy/agent_records.md`.
- `devpolicy/contracts/agent/examples/` — 5 valid files (agforge,
  agautolab + overlay, agdevworld shapes) and 9 invalid files, indexed
  with expected error codes in `examples/README.md`. Invalid cases
  cover unknown harness (the `"ollama"`-as-harness collision itself),
  unknown/malformed model, incompatible harness+provider, unknown
  profile, overlay scope violation, embedded secret value, unmet
  capability, and missing schema. `E_UNAVAILABLE` is documented as
  run-time-only, no fixture.
- `legacy-names.md` (this directory) — the explicit list of legacy
  names, env vars, code paths, and stale docs that Phases 2–5 will
  delete, per project, from a fresh survey of all three codebases.

## Grounding

The contract was written against a same-day survey of the actual
selection mechanisms: agforge's `AGFORGE_AGENT_BACKEND=ollama|claude`
(OpenCode vs Claude Code CLI, plus an `AGFORGE_AGENT_CMD` bypass and a
hardcoded claude model), agautolab's `AUTOLAB_WINDOW_BACKEND` (direct
`/api/chat` vs Claude Code) with per-role env vars and job.yaml
`adapter_config.args` model smuggling, and agdevworld's
`ASSISTANT_BACKEND` (direct Ollama vs Anthropic SDK, no harness at
all). Two existing conventions were deliberately kept and generalized:
the versioned-envelope precedent (`autolab.monitor.v1`,
`assistant.run.v1`) and agautolab's adapter registry key
`claude_code`, which was already the canonical spelling. The
`backend_model` composite (`"claude/claude-sonnet-5"`) that both
agautolab and agdevworld record was the closest proto-canonical form;
the contract replaces it with separate `harness`/`provider`/`model`
fields and tells archive readers to expect the old spelling.

## Design decisions made here (revisitable by adopters with evidence)

- **Model declaration is required** (`[models]` table) so "unknown
  model" is checkable without a global model registry; per-model
  harness-interpreted knobs (agdevworld's `effort`/`max_tokens`) live
  on the model entry.
- **The overlay may override role→profile selection** in addition to
  the roadmap's three local-fact categories. Today's live behavior
  (agautolab `.local/.env` flipping the window backend per node)
  showed node-level profile selection is a real need; confining it to
  the overlay keeps it out of committed files.
- **Secret references must end `_file` or `_env`** — a mechanically
  checkable rule instead of prose about not committing secrets.
- **Error codes, not messages, are the conformance surface** — the
  cheapest way to make Python/JavaScript loader drift visible without
  shared runtime machinery.
- **Direct provider API calls have no harness ID** — they are the
  thing being removed, so the contract cannot express them; a project
  still on one simply hasn't adopted the contract yet.

## Unresolved choices left to the first adopter (agforge, Phase 2)

1. **Where the OpenCode provider endpoint really lives.** The live
   Ollama baseURL sits in `~/.config/opencode/opencode.json`, outside
   the repo; the overlay's `[local.provider.ollama].base_url` declares
   it, but OpenCode reads its own config. Decide whether the loader
   generates/patches OpenCode's config, passes flags, or merely
   verifies agreement and fails on mismatch (`E_UNAVAILABLE`).
2. **Provider-prefix stripping per harness.** Spec says loaders derive
   the provider-native spelling from the canonical ID (`claude --model
   claude-sonnet-5` vs `opencode -m ollama/...` which keeps the
   prefix). Confirm this per-harness rule is enough once real argv is
   built, and record the mapping in the loader tests.
3. **Fate of `AGFORGE_AGENT_CMD`.** The recommendation is a
   `fake`-harness profile with a `[local.harness.fake].command`
   overlay key; confirm the fake harness needs a command at all or
   whether project test doubles suffice, then codify in the contract
   examples if the shape changes.
4. **Tool-grant duplication is out of contract scope** — agforge keeps
   `opencode.json` and `CLAUDE_ALLOWED_TOOLS` as two hand-synced
   lists. If Phase 2 finds a safe single source, that is project
   design, not a contract change; note whatever is learned.
5. **Capability granularity.** agforge declares none today. If Phase 2
   finds it needs deployment capabilities (SwarmUI, MinIO reachability)
   expressed, decide whether those are `requires` capabilities or stay
   project-owned runtime checks; the contract's open vocabulary
   accepts either.
6. **Record placement.** agforge currently logs `meta` to
   `service.log` and persists no per-run record file. The normalized
   fields (§9) fix the *fields*; whether agforge grows a record file
   or enriches the log line is the adopter's call under
   `devpolicy/agent_records.md`.

## Constraint check

- No credentials or private payloads committed; secret examples use
  fake placeholder values marked as such.
- No silent-fallback path introduced — the spec forbids it and
  provides `E_UNAVAILABLE` instead.
- Raw-output retention is written into the metadata section
  (normalization adds fields, never replaces transcripts).
- Nothing touches `--dangerously-skip-permissions` or equivalent;
  agautolab's existing charter prohibition surfaced in the survey and
  remains a project-level rule the contract does not weaken.
