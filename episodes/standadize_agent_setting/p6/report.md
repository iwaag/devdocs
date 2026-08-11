# Phase 6 report — deployment and local environment reconciliation

Date: 2026-08-11.

Reconciled the standardized agent configuration with the agdevworld image and
the agautolab node deployment, including live intent and node rollout — handoff
candidate.

Did the agautolab1 harness deployment and profile-selected verification for the
agautolab agent — handoff candidate.

## Outcome

Phase 6 is complete. A clean agdevworld checkout no longer needs an ignored
compose overlay: its assistant image contains pinned OpenCode 1.18.10 and
Claude Code 2.1.226, and its entrypoint generates the §6 local overlay from
deployment environment values. The image does not contain host Claude login
state or a credential.

The agautolab deployment now installs both CLIs in user scope, keeps OpenCode
CLI installation separate from the optional `opencode serve` daemon, renders
the project overlay from Nautobot placement config, and removes the legacy
window backend environment and `claude_bin` pointer path. agautolab1 completed
a real `local-coder` front run using only the deployed checkout and overlay.

## Implementation

### agdevworld

- Pinned `opencode-ai@1.18.10` and
  `@anthropic-ai/claude-code@2.1.226` in the assistant image.
- Added an entrypoint and tested JavaScript renderer for
  `/app/.local/agents.local.toml`. It emits only harness commands, the Ollama
  `/v1` endpoint, an Anthropic environment-variable reference, and an optional
  front profile override.
- Removed the `.local/agents.compose.toml` bind mount. Compose now supplies
  deployment facts with clean-checkout defaults and optionally passes
  `ANTHROPIC_API_KEY`.
- Added `.local` to `.dockerignore`; neither ignored host overlays nor secrets
  enter either image build context.
- Updated the harness matrix and container authentication documentation.

### agautolab and Ansible

- Added `opencode_agent_serve`; `false` installs the verified CLI without a
  global OpenCode config, service unit, handler restart, or `:4096` probe.
  Existing node-agent placements retain the default daemon behavior.
- Added a user-scoped Claude Code role pinned to 2.1.226. It installs through
  the node's existing user npm prefix and links the executable into
  `~/.local/bin`.
- `setup_autolab_node.yml` now installs both harnesses before the project role.
- Added deployment-profile variables for the Ollama `/v1` endpoint and all five
  role profile overrides. The autolab placement in Nautobot now selects
  `local-coder` for `front`, `director`, `mediator`, `coding`, and
  `summarizer` and supplies
  `http://agstudio.home.arpa:11434/v1`.
- Added the `agents.local.toml` template and optional controller-local
  Anthropic key-file provisioning. The overlay holds only the target path;
  agautolab resolves that reference into the child process environment.
- Removed `autolab_node_window_backend`, `autolab_node_window_model`,
  `autolab_node_ollama_url`, `autolab_node_claude_bin`, all three legacy
  gateway environment lines, and the `claude_bin` file task.
- Made agautolab's four role-specific OpenCode configs self-contained by
  declaring the provider/model and reading the deployment endpoint from
  `AGENT_PROVIDER_OLLAMA_BASE_URL`. This was learned from the first clean-node
  run; relying on a developer global config had hidden the missing fact.
- Updated the Ansible deployment guide, agautolab deployment pointer, and the
  workspace developer command.

The agautolab runtime changes needed by the target were committed locally as
`7c82e57` and `0a5bdef` and pushed only to the named agstudio Gitea deployment
remote. The GitHub remote was not pushed.

## Verification

| check | result | evidence |
|---|---|---|
| agdevworld deterministic suite | **26 passed** | `npm test` |
| agdevworld build | passed; existing 1.4 MB Vite chunk advisory only | `npm run build` and compose image build |
| clean-checkout compose path | passed; no local bind mount in rendered compose | generated overlay at container start |
| image harness pins | OpenCode **1.18.10**, Claude Code **2.1.226** | commands executed inside assistant container |
| agstudio services | web `/` HTTP 200; assistant `/healthz` healthy | documented compose workflow on `:8090`/`:8091` |
| container OpenCode run | done, exact `phase6-container-opencode-ok` | record `7852a7c2-7d94-4cb7-9cfa-a86a9166b42e` plus raw JSONL |
| container Claude without key | explicit HTTP 502, `Not logged in`; no fallback | failed record `448fba87-86f5-419d-961e-a39203cbd405` plus raw envelope |
| agautolab suite | **89 passed** | full `uv run pytest -q` |
| Ansible syntax | passed | production-inventory syntax check |
| Ansible conformance | **4 passed** | `test_ansible_conformance.py` |
| legacy deployment grep | no matches in `ansible_agdev` | four removed legacy names/path |
| live deploy | passed; final rerun `changed=0` | production inventory, `--limit agautolab1` |
| node harness pins | OpenCode **1.18.10**, Claude Code **2.1.226** | Ansible ad-hoc version checks |
| node gateway | `/healthz` returned `{"ok": true}` | deployed cleaned user unit |
| node profile-selected run | done, exact `phase6-node-ok`, 20,333 ms, 1 turn | `window/run-0006` and `run-0006.agent.jsonl` |
| diff hygiene | passed | each touched repository's `git diff --check` |

The first node run, `window/run-0005`, preserved a raw OpenCode error event.
The selected CLI and `/v1` endpoint were available, but the role-specific
OpenCode file lacked the provider declaration that the developer Mac supplied
globally. After making the project configs self-contained and redeploying, the
same selected profile succeeded. No alternate harness or model was tried.

## Intent and drift evidence

- Pre-deployment `nctl status`: operation
  `01KZPD7X3TW52DM7KG8X203PNP`; Nautobot, intent GraphQL, and one worker were
  healthy.
- Applying `.local/phase6-agent-deployment.yaml` previewed one update, then
  committed one update with no creates, deletes, or conflicts.
- Production render generation:
  `2da0be2f-ac90-431d-9f34-e06edbc5af6b`. The generated host vars contain the
  endpoint and all five role selections.
- Post-deployment `nctl status`: operation
  `01KZPDNNMGCFM5GBQ64CE3SKZJ`; infrastructure checks remained healthy.
- Final `nctl drift --host agautolab1` reports both the node and compute
  instance converged, zero errors/warnings, and both active placements applied.

The previously known address discrepancy remains visible but was not changed:
Nautobot's desired primary IP is `192.168.0.130`, while the observed and used
connection address is `192.168.0.220`. nctl nevertheless classifies the current
node and compute targets as converged; this phase did not silently rewrite
network intent.

## Contract findings

No Phase 1 contract change was necessary. Specification §6 survived its first
two generated deployment overlays:

- command paths, provider endpoints, secret references, and role profile
  overrides were sufficient for both a container and a remote node;
- strict overlay-scope validation accepted both generated forms unchanged;
- the `_file`/`_env` convention kept secret values out of templates,
  placement intent, inventories, images, and reports;
- no profile/model/harness definition had to move into a deployment overlay.

The only awkward point was operational rather than contractual: a secret
reference must be resolved by the project runner before spawning the harness.
agautolab now does so for Anthropic key-file/environment references;
agdevworld passes the referenced environment variable directly through.

## Deployment findings

- CLI installation and daemon ownership were genuinely separate concerns.
  `opencode_agent_serve: false` made the autolab node usable without creating
  an unrelated long-running server or global model configuration.
- Clean-node evidence found an accidental dependency on a developer's global
  OpenCode provider declaration. Keeping per-role permissions separate is
  compatible with repeating the small provider block in project-owned configs.
- No Anthropic credential was provisioned on the node or in compose during
  this phase. Claude binary availability was proven; selecting Claude without
  authentication failed loudly and produced normalized/raw evidence.

## Constraint check

- No credential or generated private payload was committed or baked into an
  image. Secret-bearing local files were not printed.
- There is no silent harness/model fallback.
- Successful and failed raw harness output remains next to normalized records.
- No `--dangerously-skip-permissions`, OpenCode `--auto`, or equivalent
  unrestricted native-host mode was added.
- The OpenCode daemon remains enabled only for its existing node-agent
  placements; agautolab receives CLI-only installation.
