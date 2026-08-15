# Report 9 — devpolicy, and the nintent decision

Status: **done**. devpolicy is clean; nintent is deliberately untouched and
noted as dead config, per the plan's own recommendation.

## `devpolicy/contracts/agent/spec.md`

- §2 harness table: the `opencode` row is gone.
- §2 prose: "Ollama is not a harness; it is a model provider **reachable
  through OpenCode**" → "a model provider **a harness reaches**".
- §2 vocabulary-change paragraph rewritten. It documented one change (agcode
  added); it now documents both, and gives the migration:

  > `opencode` (`opencode run`, any configured provider) was **removed** once
  > nothing ran on it. […] Its intrinsic capabilities were `agentic_tools` and
  > `workspace_fs`, the same pair `agcode` provides, so a profile that only
  > needed those migrates by changing the harness name and the endpoint
  > spelling: `agcode` posts to `{base_url}/v1/messages`, so a
  > `local.provider.*.base_url` written for an OpenAI-compatible client loses
  > its `/v1` suffix.

  That last clause is the episode's most repeated finding — four separate
  `/v1` sites (three consumer overlays, one Ansible assertion) — written down
  where the next adopter will hit it.
- §3: the native-name example `opencode -m ollama/qwen3.6:...` → `python -m
  agag.agcode --model qwen3.6:...`.
- §4: the example profile and the standard-profile sentence now say `agcode`.
- §6: the overlay example loses `[local.harness.opencode]`, and the provider
  endpoint gains `# bare base URL; agcode appends /v1/messages`.
- §7: the intrinsic-capability sentence.

`devpolicy/agent_records.md`: the backend example is now `agcode +
ollama/<model>`.

`contracts/agent/examples/` were done in Steps 4 and 8 as their consumers
needed them — `valid/*` in report 4, `invalid/*` in report 8. The
`unknown-harness.toml` fixture is untouched: it uses `ollama`, and the point
it makes (a provider is not a harness) is unaffected.

## nintent: left alone, as recommended

`PROFILE_BINDING_NAMES = {"node_agent": ("llm_provider",)}` and
`REFUSED_PROFILE_CONFIG_KEYS = {"node_agent": ("llm_provider_service",)}` in
`nautobot_intent_catalog/models.py` still name the retired profile, and
`tests/factories.py` and `tests/test_batch.py` use it heavily.

**This is dead config and it is harmless.** nintent only *rejects undeclared*
binding names; it never requires the profile to exist. No placement can carry
`deployment_profile: node_agent` any more, so neither map is ever consulted
with that key.

Changing it costs a commit → asking the developer to push → `docker compose
build` → a Nautobot restart (the plugin is installed from GitHub, not mounted;
there is no hot reload), and would gut the binding tests for no behavioural
gain. Left as the plan recommends.

## Two items outside the plan's inventory

The plan's coupling table did not list these, and both turned out to matter.

### `devtests/test_strategy/test_mtls_conformance.py` — was broken

A required conformance gate (`README_DEV.md`'s test-strategy matrix) imported
`cagent_api.opencode_client`, deleted in Step 5. It could not even collect.
Its `_FakeOpenCode` — a session-API imitation with a message counter, a
completion flag and a `is_final_step` distinction — is replaced by a
20-line `_FakeRunner` with `new_session_id`, `identity` and `run`. The gate is
green again: **23 passed**.

### `nodeutils` — collected opencode from every node

`nodeutils_collect.py` had `_node_agent_version()` (running
`opencode --version`), a Linux branch matching `opencode-agent.service`, and a
macOS branch matching `com.clusterintent.opencode.agent`. All three are gone;
the generic user-service probes (ollama, and any hinted process) are
untouched. Its 91 tests pass, the submodule is pushed and the superproject
pointer bumped, and `devenv/observation/refresh-node-observations.sh`
redeployed it — aghub now reports `git -C /opt/nodeutils rev-parse` =
`39ce06a`, the new commit.

**One residue, honestly reported.** Two devices (aghub, agpc) still carry an
`observed_services["node-agent"]` row in Nautobot's *actual* state, naming
`opencode-agent.service`, and a full observation refresh with the new
collector did **not** clear them. The cause is that observation ingest merges
service maps rather than replacing them, so a key that stops being reported is
never removed. It is inert — `nctl drift` has no `node-agent` target because
desired state has none, and `nctl status` is ok — but it is a stale fact in
the actual-state store, and removing it needs an ingest change that is not
this episode's business. Recorded here rather than papered over.

## The final grep

`grep -ril opencode` across every project now returns:

- `devdocs/**` and `pj-*/devdocs/**` and `privateharness/devdocs/**` — the
  historical record, deliberately untouched.
- `devpolicy/contracts/agent/spec.md` — the removal record itself.
- `pyagag/README.md` and `docs/agent-config-v1.md` — the same, from the
  implementation's side.
- `pyagag/tests/test_harness.py` — `test_an_unsupported_harness_is_rejected`,
  which asserts the name is now refused.
- `pj-clusterintent/devenv/launchd/README.md` and
  `com.clusterintent.cagent-api.plist.in` — the upgrade instruction for the
  two removed labels, and the comment explaining why the plist sets `PATH`
  (the retired start scripts did it, and the tools now run in-process).
- `pj-agdev/agdevworld/public/cluster/actual.json` — a git-ignored generated
  snapshot of the actual state above; it will carry the same two stale rows
  until the ingest residue is cleared.

Every one of those is a deliberate record of the removal, not a survival of it.
