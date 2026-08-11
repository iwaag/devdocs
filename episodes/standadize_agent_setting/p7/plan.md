# Phase 7 plan — cross-project convergence and cleanup

Roadmap: `../roadmap.md` §Phase 7. Contract: `devpolicy/contracts/agent/`
(`ag.agent-config.v1`). This is the closing phase: run the same named
profiles across agforge, agautolab, and agdevworld, delete what the matrix
proves dead, and write the final report.

The environment is experimental. Rebuilds, renames, and deleted files are
fine; backward compatibility is explicitly not required. Structure, order
within a step, and recovery from failed experiments are the implementer's
judgment. Write `report<step>.md` per step (or fold small steps together)
and a final `report.md`.

## Hard constraints (the only ones)

- No credentials or private payloads in Git.
- No silent harness/model fallback — unavailable selections must fail
  loudly (`E_UNAVAILABLE`).
- Keep enough raw output to diagnose failed runs.
- No `--dangerously-skip-permissions` / `opencode run --auto` equivalents.
- agautolab deploys from the agstudio Gitea remote, never run
  skip-permissions jobs on this Mac (see `pj-agdev/.local/devenv.md`).

## Known state going in (verified 2026-08-11)

- Legacy sweep is essentially done: repo-wide grep for the
  `p1/legacy-names.md` list hits only episode archives and
  `agautolab/tests/test_legacy_removed.py`. Step 3 is a verification and
  straggler hunt, not a big excavation.
- **Profile-name divergence** (the main convergence gap): agforge and
  agautolab define `local-coder` / `sonnet-coder` / `stub`; agdevworld
  defines `local-front` / `sonnet-front` / `stub`. Same harness+model
  pairs, different names. Roadmap requires "a profile means the same
  harness and canonical model everywhere".
- **agforge conformance gap**: `agforge/tests/test_agent_config.py` uses
  inline TOML, not the shared `devpolicy/contracts/agent/examples/`.
  agautolab (`tests/test_agent_config.py`) and agdevworld
  (`assistant/tests/agent-config.test.mjs`) already read the shared
  examples directly.
- **Anthropic auth exists only on this Mac.** agautolab1 and the
  agdevworld container have the Claude binary but no credential; selecting
  a `sonnet-*` profile there fails `E_UNAVAILABLE` (proven in p6). Decide
  in step 4 whether to provision (Ansible already has an optional
  controller-local key-file var) or record the exception.
- Local services are started by hand: agforge service `:8092`, agautolab
  gateway `:8791`, agdevworld web `:8090` / assistant `:8091`. Ollama
  OpenAI endpoint needs the `/v1` suffix. Cluster state questions →
  `nctl status` / `nctl drift` from `pj-clusterintent`.
- agautolab p6 commits (`7c82e57`, `0a5bdef`) are on Gitea only, not
  GitHub — sync opportunity during cleanup.
- Prior cost/latency samples live in `p2`–`p6` reports (Claude total so
  far ≈ $1.79; Ollama runs all $0). Reuse them in the final report; the
  matrix adds one more comparable sample per cell.

## Step 1 — converge the profile vocabulary

Pick one shared profile name set and apply it in all three `agents.toml`
files, overlays, Ansible deployment vars
(`pj-clusterintent/ansible_agdev/vars/`), and the contract examples.

Suggestion: role-neutral names (`local`, `sonnet`, `stub`) sidestep the
coder/front split, but any single vocabulary is fine — the acceptance test
is "same name → same harness + canonical model in every project". Update
the agdevworld entrypoint-generated overlay and the agautolab
`agents.local.toml` template if names appear there. If a project genuinely
needs an extra project-only profile, keep it but list it as an exception
in the final report.

Evidence: each project's own tests pass; grep shows one profile
vocabulary across the three `agents.toml` files and deployment vars.

## Step 2 — one shared conformance gate

Wire agforge's loader tests to `devpolicy/contracts/agent/examples/` the
way agautolab and agdevworld already do (path resolution relative to the
workspace root; copy their approach). Keep or drop the inline-TOML cases
at your discretion — the shared examples are the gate.

Then run all four gates and record the commands + results:
agforge pytest, agautolab pytest (incl. `test_legacy_removed.py`),
agdevworld `node --test assistant/tests/`, and
`pj-clusterintent/devtests/test_strategy/test_ansible_conformance.py`.
If step 1's renames changed example files, all three loaders must agree on
the same accept/reject + error-code decisions (spec §8 is the conformance
surface).

## Step 3 — cleanup sweep

During and after the matrix, delete what is actually dead:

- re-grep the `p1/legacy-names.md` list plus anything new the matrix
  exposes; the only allowed survivors are episode archives and the
  legacy-absence test;
- duplicate parsers / migration-only adapters that stopped earning their
  keep (note: agforge's two hand-synced tool lists and the per-role
  OpenCode permission files were *deliberately accepted* in p6 — leave
  them unless you find a clearly better shape while touching them anyway);
- obsolete dependencies (`package.json`, `pyproject.toml` — the Anthropic
  SDK should already be gone; verify);
- dead documentation still describing legacy backend names (README_DEV,
  deployment docs, devenv notes).

Keep raw failure records that teach something. `.local/` archives with old
`backend_model` spellings stay — spec §9 readers tolerate both.

## Step 4 — the live matrix

Goal: for every profile available in an environment, one recorded run per
project proving profile → harness → canonical model identity end to end.
Suggested matrix (adjust names to step 1's outcome):

| Project / role | local (opencode+ollama) | sonnet (claude_code+anthropic) | stub/fake |
|---|---|---|---|
| agforge `generator` (this Mac) | run | run | test-only |
| agautolab mission front→mediator→coding (this Mac and/or agautolab1) | run | run where auth exists | test-only |
| agdevworld `front` via `/api/chat` (container) | run | see below | test-only |

Practical notes:

- Reuse each project's proven representative tasks from p2–p5 (agforge
  image request; agautolab mission with recorded backend identity at each
  boundary; agdevworld chat + fetch/wait + UI-action tasks).
- `sonnet` on agautolab1 / in the agdevworld container: either provision
  the credential through the existing deployment path (Ansible optional
  key file; container env) and run it, or record the loud
  `E_UNAVAILABLE` failure as the exception. Both are acceptable outcomes;
  silent fallback is not.
- If you deploy to agautolab1: push to Gitea, then the Ansible playbook
  (`README_DEV.md` §Ansible Commands), and check `nctl status` / `nctl
  drift` before and after; the known agautolab1 IP intent discrepancy
  (desired .130 vs actual .220) is pre-existing — call it out, don't fix
  it here.
- Do not add retries or fallback to make cells green. A raw failure with
  its record is a valid cell.

Evidence: per-cell `.agent-run.json` (or equivalent) records; a short
matrix table in the step report with role, profile, harness, provider,
model, outcome, duration_ms, cost_usd per cell.

## Step 5 — record comparability check

Take one record per project from step 4 and confirm the normalized fields
(spec §9: role, profile, harness, provider, model, outcome, duration_ms,
cost_usd, usage, num_turns, failure) are present and mean the same thing.
A tiny throwaway script in the scratchpad or a manual side-by-side table
is enough — do not build a shared runtime library for this. Any field
drift found: fix the emitter or record it as a contract erratum, whichever
is cheaper and honest.

## Step 6 — final report

`p7/report.md` and this episode's close-out, covering (roadmap evidence
list):

- the resulting contract as it actually stands (spec version, profile
  vocabulary, capabilities in use), with deltas made during this phase;
- remaining exceptions (expected: Anthropic auth coverage, agautolab1 IP
  drift, deliberately kept duplications, any project-only profiles);
- cost/latency observations: matrix numbers plus the p2–p6 samples,
  presented as observations, not selection rules;
- best next ENT experiments — candidates seen so far: Failure Farming on
  harness behavioral differences (p3/p5 findings), secret-reference
  resolution ergonomics (runner-side vs contractual, p6 note),
  profile-selection heuristics once more cost data exists;
- Gitea↔GitHub sync status for agautolab.

Keep it concise; link raw evidence instead of inlining it.
