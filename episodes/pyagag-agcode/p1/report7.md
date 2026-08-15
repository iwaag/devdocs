# Report 7 — nctl and clusterintent state

Status: **done**. nctl suite green (1315 passed). `nctl status` ok, `nctl
drift` clean of node-agent, rendered inventory regenerated.

## The `nctl agent` subsystem is gone

Deleted: `agent.py` (SSH-tunnelled access to node-local agents), `agent_api.py`
(the session client), `agent_render.py`, the `agent_app` typer tree and its
six commands (`status`, `attach`, `sessions`, `run`, `send`, `abort`),
`AgentConfig` and `Config.resolved_agent_identity_file()`, the `[agent]`
section of `example.nctl.toml`, and `tests/test_agent.py`,
`tests/test_agent_api.py`, `tests/test_cli_agent.py`.

It existed only to tunnel to a loopback `opencode serve` and to run
`opencode attach`. Nothing reaches it now — every agent in this system has its
own conversational entrance.

**`nctl.toml` needed the same edit.** `Config` is a `StrictModel`, so a live
`nctl.toml` carrying `[agent]` now fails to load — every nctl command errors
until it is removed. Removed from this machine's file; anyone else's copy
needs the same one-line deletion. That is the destructive phase working as
specified (no back-compat shim), but it is a real break worth naming.

## Service dependencies

`PROFILE_BINDING_VARIABLES` is now `{}` — its only entry was
`("node_agent", "llm_provider") → "nintent_opencode_ollama_url"`, and that
variable is out of `production/contract.py`'s `_BASE_HOST_VARIABLES` too.

The binding machinery stays, as instructed. The suites that exercise it
(resolution, drift, relations) declare their own binding in a new
`tests/conftest.py`:

```python
TEST_BINDING_PROFILE = "llm_consumer"
TEST_BINDING_NAME = "llm_provider"
TEST_BINDING_VARIABLE = "nintent_llm_provider_url"
```

installed by an autouse `monkeypatch.setitem`. That is the plan's
"test-local mapping" option, preferred over leaving a neutral production
entry: a declared production binding nothing consumes is exactly the dead
config Step 9 recommends noting rather than creating.

**One extra removal, worth flagging.** `_endpoint_url()` appended `/v1` to
every resolved binding URL, unconditionally. That suffix existed for the one
consumer that wanted OpenAI-compatible paths. With the map empty it changes
nothing today, but leaving it would have handed the *next* binding a URL with
a path it never asked for — the same class of trap as the `/v1` overlays in
reports 2–4 and the ansible assertion in report 6. It now returns the bare
`protocol://host:port`.

## The comment sweep

`reconcile/classify.py:99` (`setup_opencode.yml` in the binding-repair
comment), `observation.py` (the `node_agent`/`llm_provider` example),
`reconcile/profiles.py` (the OpenCode-config rationale for allowing a
`~`-relative `config_file`), `README_DEV.md` ("its OpenCode turn"), and
`agentdocs/brainforge/README.md` (which listed `nctl agent` as an execution
tool) are all rewritten. `grep -ril opencode nctl/` is empty.

Test fixtures that used `node_agent` / `node-agent` as arbitrary profile and
service names — in the drift, relations, prune, planner, export and
observation suites — are renamed to `llm_consumer` / `llm-consumer`. They were
never testing the node agent; they were testing generic machinery with its
name on it.

## Desired state

**Refreshed the operator input first**, as report 6 warned:
`nctl desired export -o .local/desired-state.yaml`. The checked-out file was
behind Nautobot and would have repointed agautolab1's deployment source at the
Gitea repository deleted on 2026-08-14. It is now a faithful copy of live
state (90 → 81 operations after the removal below).

Removal, applied as a hand-written batch (bindings, then placements, then the
services):

```
dry_run:   {'create': 0, 'update': 0, 'delete': 9, 'unchanged': 0, 'conflict': 0}
committed: {'create': 0, 'update': 0, 'delete': 9, 'unchanged': 0, 'conflict': 0}
```

- service `node-agent` + placements `node-agent-{aghub,agpc,agstudio}` +
  their three `llm_provider` bindings.
- service `cagent-opencode` + placement `cagent-opencode-agstudio` (whose
  `process_pattern` was `opencode serve --hostname 127\.0\.0\.1 --port 4097`).

Two notes on the mechanics, for the next person hand-writing one of these:

- `op: delete` needs an explicit `values: {}`. Omitting it is a bare
  `HTTP 400`; only `--json` shows the reason
  (`operations[0] must contain op, kind, key, values`). Worth a line in
  `docs/desired-partial-batch.md`, which documents only `op: upsert`.
- **`nctl prune` does not apply here.** The plan says to follow with it, but
  `nctl prune HOST` deletes a fully-retired *guest's* Actual and Desired
  records — it is node-scoped, not service-scoped. There is no orphan to
  collect: the batch deleted the rows directly, and `nctl drift` confirms it.

## The deployment profile

`vars/deployment_profiles.yml` lost the `node_agent` profile and its
`deployment_profile_reconciliation` entry (the one whose action was the
deleted `setup_opencode.yml`, with the `llm_provider` binding slot pointing at
`~/.config/opencode/opencode.json`). Removed **after** the desired-state
deletion — the reverse order would have left placements referring to an
undeclared profile.

`deployment_profile_digest` is computed from the file, so it simply changed.

## Verification

```
nctl render production --out inventories/generated
  placements: 27 → 23, active: 22 → 19
  grep 'nintent_opencode|node_agent|node-agent|cagent-opencode' production.yml → empty

nctl status   → ok: True
nctl drift    → summary: converged=28 unknown=5   (was 29/6)
                node-agent/opencode targets in the JSON envelope: 0
nctl pytest   → 1315 passed
ansible-playbook --syntax-check setup_autolab_node.yml → ok
```

The one remaining `nctl drift` warning is `pj-voxel3dprint:
workspace_observation_stale`, unrelated to this episode and present before it.

## Deus Ex Machina note

Did the desired-state retirement of `node-agent` and `cagent-opencode` for
agent `cagent` — handoff candidate. cagent's window can read drift but cannot
write desired state, so this one could only have been handed to its
authenticated door.
