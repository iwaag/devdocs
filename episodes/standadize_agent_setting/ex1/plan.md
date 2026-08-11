# Plan: Extract shared agent-setting package `pyagag`

Episode: standadize_agent_setting/ex1
Braindump: [braindump.md](braindump.md)

## Goal

Extract the duplicated agent-setting/harness layer of agforge and agautolab into
the new public repo `pyagag` (already cloned at repo root, currently LICENSE-only),
and rework both projects to consume it. agforge is refactored so its call
structure stays close to agautolab's — some rework cost is accepted.
Success criterion: adding a hypothetical new harness (e.g. gemini cli) would
touch pyagag once, plus one JS edit in agdevworld.

This is a destructive phase. No backward compatibility required. The runtime is
an experimental lab environment: keep prohibitions minimal, use your judgment.

## Non-goals

- agdevworld's JS implementation stays as-is (sibling implementation of the
  same contract). Only document the contract so it can follow later.
- Orchestration (agautolab gateway/loop/gates, agforge request_service/charter)
  stays project-specific.
- Deploying the updated agautolab to the agautolab1 node (gitea push + ansible)
  is out of scope; local correctness is enough.

## Step 1 — Build the package in `pyagag`

Suggested shape (mirror agautolab's conventions: hatchling, src layout, uv,
pytest, py>=3.11):

```
pyagag/
  pyproject.toml          # name "pyagag"; import package name is implementer's choice (e.g. agag)
  src/<pkg>/agent_config.py
  src/<pkg>/harness.py
  tests/
  docs/agent-config-v1.md # language-neutral contract spec (see Step 4)
```

Contents:

- **agent_config**: lift from `agautolab/src/agautolab/agent_config.py` (the
  richer of the two near-identical copies — it has `ResolvedAgent.environment`
  and `_resolve_secret_environment()`). Merge notes:
  - Parameterize the project name currently hardcoded in the ollama base_url
    error message ("required by agautolab OpenCode").
  - Drop agautolab's module-level `PROJECT_ROOT`/`AGENTS_CONFIG` constants;
    take paths as arguments. Convenience wrappers stay in the apps.
- **harness**: unify `agforge/service/agent_run.py` (launch parts) and
  `agautolab/src/agautolab/harness.py`. Merge notes:
  - API style: adopt agautolab's non-raising `HarnessResult` with
    `meta["outcome"]`. agforge wraps it and raises its own error where needed.
  - Take agautolab's richer `run_harness` (add_dirs, extra_args,
    skip_permissions, opencode_config, raw_output, environment injection) as
    the base; make `OUTPUT_TAIL_CHARS` a parameter (agforge uses 800,
    agautolab 2000).
  - Keep agforge's `harness == "fake"` handling in `build_argv` — agautolab's
    copy lacks it.
  - `extract_event_text`, claude JSON unwrap, `identity()`, `ANSI_RE`, and the
    `ag.agent-run.v1` run-record writer all go in; they are byte-identical or
    trivially reconcilable across the two projects.
- **Out of the package** (policy, not mechanism): tool allowlists
  (`CLAUDE_ALLOWED_TOOLS`, `ROLE_ALLOWED_TOOLS`) — pass as arguments; the
  adapter registry; anything HTTP.

Tests: port the relevant existing tests from both projects into pyagag rather
than writing from scratch (agautolab has harness/config coverage; agforge's
tests live in `agforge/tests/`). Delete ported tests from the apps or reduce
them to integration smoke tests — implementer's choice.

## Step 2 — Migrate agautolab (the easy one)

- Delete `src/agautolab/agent_config.py` and `harness.py`; re-import from
  pyagag. Keep `resolve_project_role()`-style convenience wrappers locally.
- Adapters (`adapters/claude_code.py`, `opencode.py`, `fake.py`) are thin
  (~40 lines) over `run_harness` — they should survive with only import changes.
- Dependency wiring for local dev: uv path source, e.g. in pyproject
  `[tool.uv.sources] pyagag = { path = "../../../pyagag", editable = true }`
  (relative path from `pj-agdev/agautolab`; verify — relative paths are fine to
  commit, absolute local paths are not, per devpolicy/styles.md). Switch to
  `git+https://github.com/iwaag/pyagag.git` once the package stabilizes,
  if ever desired.
- Regenerate `uv.lock`; `uv run pytest` must pass.

## Step 3 — Migrate agforge (the refactor)

agforge today is *not* a package: `service/*.py` are PEP 723 inline-metadata
scripts run via `uv run service/agent_run.py`, with flat imports glued by
`sys.path.insert` at `service/request_service.py:48`. Accepted decision: pay
the cost and restructure to mirror agautolab.

- Convert to a real hatchling src-layout package (`src/agforge/`), moving the
  logic out of `service/`. Keep thin launcher scripts (or `[project.scripts]`
  entries) so `serve.sh` / `scripts/generate.sh` still work. Drop the PEP 723
  blocks and the `sys.path.insert` glue.
- Align the call structure with agautolab where it's cheap: resolve role via
  pyagag's config, call `run_harness`, get a `HarnessResult`, branch on
  `meta["outcome"]`. Whether to also adopt an adapter-registry layer like
  agautolab's is the implementer's call — for agforge's single "generator"
  role it may be overkill; a direct call is fine as long as the seam
  (config → resolve → run_harness → result) is the same shape.
- `compose_charter()`, `transform.py`, `request_service.py`, and
  `CLAUDE_ALLOWED_TOOLS` stay agforge-local; pass the allowlist into
  `run_harness` as an argument.
- **Fix the latent bug found during planning**: agforge validates the secret
  keys (`anthropic_api_key_env|_file`) but never consumes them. With pyagag's
  agautolab-derived config this comes for free — make sure agforge actually
  injects `ResolvedAgent.environment` into the subprocess env.
- Same uv path-source wiring as Step 2 (path is `../../pyagag` from
  `pj-agdev/agforge`); regenerate `uv.lock`; tests pass.

## Step 4 — Document the contract

Write `docs/agent-config-v1.md` in pyagag: the `ag.agent-config.v1` schema
(committed file vs `.local` overlay split, role→profile→harness resolution,
model-id format, canonical harnesses, the stable error codes `E_SCHEMA` …
`E_UNAVAILABLE`), and the `ag.agent-run.v1` record shape. Source of truth for
the spec is the code that already exists in three places — agdevworld's
`assistant/agent-config.mjs` reimplements the same error codes, which is the
proof this is a language-neutral contract, not a Python API. Note in the doc
that the JS implementation is a sibling, not a consumer.

## Step 5 — Verify

- `uv run pytest` green in pyagag, agautolab, agforge.
- Smoke test with the `stub`/`fake` profile in both apps (no external model
  needed): agautolab `run_once` path, agforge generation path.
- Optional live check: the agforge request service normally runs natively on
  this Mac at :8092 (`agforge/service/serve.sh`); if it's running, restart it
  on the new layout and exercise one request.
- The "gemini test": write down (in the report, no need to implement) the exact
  pyagag touch-points a new harness would need — `CANONICAL_HARNESSES`,
  `INTRINSIC_CAPABILITIES`, default-command map in `_resolve_command`,
  harness→provider compatibility check, `build_argv`, output extraction. If
  that list stays inside pyagag, the episode's desire is met.

Write `report.md` in this episode folder when done (strongly recommended by
AG style). Record which backend served any agentic runs, per README_DEV.

## Hints and gotchas collected during planning

- The two `agent_config.py` copies diff to almost nothing — treat the config
  layer as a lift, not a rewrite. The harness layer is where real merge
  decisions live (error style, parameter richness, tail size, fake handling).
- `agforge/service/agent_run.py:262-281` carries an in-source comment admitting
  it copied agautolab's claude cost-capture pattern — behavior there is meant
  to be identical; unify without fear.
- Both implementations set `AGENT_PROVIDER_{PROVIDER}_BASE_URL` and
  `NO_COLOR=1` identically — keep that in pyagag.
- `dircommon` submodule was a candidate home but stays unused; pyagag
  supersedes it. Leave dircommon alone.
- uv editable path deps + `uv.lock`: lockfiles record the relative path, so
  fresh clones need the sibling checkout at the same relative location. That's
  acceptable for this workspace layout; mention it in pyagag's README.
- Committing pyagag: it's a public GitHub repo — usual care with secrets, and
  no absolute local paths in committed files (styles.md). Nothing in the
  lifted code contains secrets today; keep it that way.
