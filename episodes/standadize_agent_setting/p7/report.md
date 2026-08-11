# Phase 7 report — cross-project convergence and cleanup

Date: 2026-08-11.

Ran the cross-project profile convergence and live matrix for the agforge,
agautolab, and agdevworld agents — handoff candidate.

## Outcome

Phase 7 is complete for the three local projects. `ag.agent-config.v1` remains
the contract version, and all adopters now use the same role-neutral profile
vocabulary:

| profile | harness | canonical model |
|---|---|---|
| `local` | `opencode` | `ollama/qwen3.6:35b-a3b-coding-nvfp4` |
| `sonnet` | `claude_code` | `anthropic/claude-sonnet-5` |
| `stub` | `fake` | test-only declared model |

The shared conformance examples use those names, and all three loaders accept
and reject the same fixtures with the same contract error codes. Roles still
own their tools, prompts, permissions, cwd, artifacts, and success judgment.
No shared runtime library, retry policy, or fallback path was added.

## Contract and cleanup changes

- Renamed `local-coder` / `local-front` to `local` and `sonnet-coder` /
  `sonnet-front` to `sonnet` in all three committed `agents.toml` files,
  contract examples, tests, and current project documentation.
- Updated the illustrative profile names in `spec.md`; this is an additive
  clarification of v1, not a schema change.
- Added agforge's missing direct gate over every shared valid and invalid
  contract fixture. agautolab and agdevworld already consumed the fixtures.
- Updated the scratch Nautobot `agautolab-agautolab1` placement so all five
  role overrides select `local`, then regenerated the production inventory.
  Preview and apply both reported one update and no create/delete/conflict.
- Re-ran the Phase-1 legacy-name sweep. Production code and current docs have
  no stale selectors or old profile names. The only legacy selector hits are
  the agautolab absence test; old ignored run archives remain intentionally.
- Checked the three dependency manifests. The direct Anthropic SDK is absent;
  the remaining Python dependencies are used by current project behavior.
  No parser, migration adapter, dependency, or document proved newly dead.
- Retained agforge's harness-specific tool lists and agautolab's role-specific
  OpenCode permission files. They remain project authority, as accepted in
  earlier phases, rather than duplicate contract parsers.

Capabilities in current committed configs are: none for agforge;
`nested_harness` and `service_http` for agautolab; `ui_actions` and
`service_http` for agdevworld.

## Shared conformance and build evidence

| gate | result |
|---|---:|
| `cd pj-agdev/agforge && uv run pytest -q` | **57 passed** |
| `cd pj-agdev/agautolab && uv run pytest -q` | **89 passed** |
| `cd pj-agdev/agdevworld && node --test assistant/tests/*.test.mjs` | **28 passed** |
| `cd pj-clusterintent && uv run --project nctl pytest -q devtests/test_strategy/test_ansible_conformance.py` | **4 passed** |
| `docker compose up --build -d web assistant` | built and started; existing 1.4 MB Vite advisory only |
| project `git diff --check` | passed in every touched repository |

Node 26 does not expand `node --test assistant/tests/` as a directory, so the
actual gate uses the explicit `*.test.mjs` glob. The Ansible gate likewise
uses the repository command matrix's `--project nctl` environment.

## Live matrix

Each successful cell has a normalized record and neighboring raw harness
transcript. `stub` is deliberately test-only and is covered by the deterministic
process-seam tests rather than a live model call.

| project / role | profile | identity | outcome | duration_ms | cost_usd | evidence |
|---|---|---|---|---:|---:|---|
| agforge `generator` | `local` | `opencode` / `ollama/qwen3.6:35b-a3b-coding-nvfp4` | done | 61,565 | 0 | `.local/out/phase7-agforge-local.agent-run.json` |
| agforge `generator` | `sonnet` | `claude_code` / `anthropic/claude-sonnet-5` | done | 100,351 | 0.4804671 | `.local/out/phase7-agforge-sonnet.agent-run.json` |
| agautolab `front` | `local` | `opencode` / `ollama/qwen3.6:35b-a3b-coding-nvfp4` | done | 10,710 | 0 | `.local/phase7/front-local.run.json` |
| agautolab `front` | `sonnet` | `claude_code` / `anthropic/claude-sonnet-5` | done | 2,934 | 0.0611898 | `.local/phase7/front-sonnet.run.json` |
| agdevworld `front` | `local` | `opencode` / `ollama/qwen3.6:35b-a3b-coding-nvfp4` | done | 26,246 | 0 | Docker record `949c0cf7-8f71-46b5-ad63-10e712b7c0d3` |
| agdevworld `front` | `sonnet` | `claude_code` / `anthropic/claude-sonnet-5` | failed loudly: `Not logged in`; HTTP 502, no fallback | 22 | 0 reported | Docker record `e58e570c-8916-4c26-9697-ad50bc5d3ae2` |

Both agforge runs received the same representative image request. The local
run delivered an image. The sonnet run kept two diffusion-model text-rendering
failures as evidence, used its existing Pillow grant to compose the requested
labeled diagram, and left a problem note; this was agent tool judgment, not a
harness/model fallback. Both agautolab cells used the same exact-answer front
smoke. The agdevworld image was rebuilt before its HTTP cells and was restored
to the default `local` profile afterward; web and assistant health checks pass.

## Run-record comparability

One record from every project was compared side by side. `role`, `profile`,
`harness`, `provider`, full canonical `model`, `outcome`, and `duration_ms`
have the same meaning everywhere. Successful live backends also reported
`cost_usd`, native `usage`, and `num_turns`. `failure` is present only on the
failed agdevworld Claude cell, as §9 intends. OpenCode usage uses aggregated
input/output/reasoning/cache keys; Claude Code retains its native token/cache
shape. No conversion or invented usage was added.

## Cost and latency observations

Phase 7 added $0.5416569 of reported Claude cost; all Ollama cells reported
$0. The tiny exact-answer agautolab sample was 10.7 s on local OpenCode and
2.9 s on Claude Code. The image task was 61.6 s on local OpenCode and 100.4 s
on Claude Code, with substantially different agent behavior and 8 versus 24
turns. These single samples do not support a selection rule. Together with
the approximately $1.79 reported in Phases 2–6, the episode has roughly
$2.33 of reported Claude observations; subscription/account reporting and
native usage shapes remain harness-specific.

## Nautobot and deployment state

- Pre-change `nctl status` operation `01KZQ62RFSAFVZ2G5MB9CV2H94` and
  post-change operation `01KZQ6MXZSWGJSXXYATFEZ5ENK` both returned `ok: true`:
  Nautobot 3.1.3 was reachable/authenticated, one worker was running, and no
  jobs were pending.
- The initial whole-cluster drift summary was 12 converged, 12 unknown, and 1
  drifting. Most errors were stale observations or unrelated service drift;
  none was changed for this phase.
- Final `nctl drift --host agautolab1` returned both node and compute instance
  converged with zero errors/warnings and showed all five desired profiles as
  `local`. The known desired `192.168.0.130` versus observed/used
  `192.168.0.220` address discrepancy remains visible and untouched.
- The node was not redeployed in Phase 7. Its Gitea deployment branch is still
  `0a5bdef`; GitHub/local is `d55ae13`, one descendant documentation commit
  ahead, plus the uncommitted Phase-7 rename. The local development memo says
  pushes are handed to the user, so no remote was pushed and no Ansible rollout
  was attempted against a branch that does not yet contain profile `local`.

## Remaining exceptions

- agdevworld's container has the pinned Claude binary but no Anthropic
  credential. Its selected `sonnet` run therefore remains a useful explicit
  `E_UNAVAILABLE`-equivalent process failure rather than a green cell.
- Anthropic authentication is available only to native Mac runs; it was not
  copied into an image or node.
- agautolab1 deployment awaits a Gitea update and then the documented Ansible
  rollout. Nautobot desired configuration and generated local inventory already
  use the converged name.
- Harness-specific permission documents and native usage shapes are deliberate
  project/harness concerns, not contract exceptions.

## Best next ENT experiments

1. Failure-farm the agforge text-rendering difference with the same image
   desire, preserving tool choices and failures before considering guidance.
2. Compare runner-side `_file` and `_env` secret-reference resolution on one
   disposable authenticated container/node; improve ergonomics only from the
   observed failure surface.
3. Collect several same-task cost/latency samples before trying profile
   selection heuristics; the current samples vary too much by task and agent
   behavior.
4. After the user updates Gitea, deploy the `local` vocabulary to agautolab1
   and use the next node run as evidence for whether deployment/profile drift
   needs a first-class observation.

## Constraint check

- No credential, presigned URL, or generated private payload was added to Git.
- No silent fallback, retry wrapper, or compatibility alias was introduced.
- Successful and failed raw harness output remains beside normalized records.
- No unrestricted native-host mode or permission bypass was added.
