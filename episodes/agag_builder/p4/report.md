# agag_builder p4 final report

Completed on 2026-08-22. A generated agag agent now runs permanently on a node
other than the controller, with desired-state placement, Ansible-managed local
configuration, cagent-driven application, Front communication, and observed
liveness.

## Published implementation

- `agecho` `c6e393e`: public GitHub source plus generated channel description.
- `pyagag` `558d4dd`: `agag provision --instance` and `--out` for provisioning
  a remote instance from the controller.
- `ansible_agdev` `ed85f76`: `agag_agent` profile/role/playbook, thin
  `autolab_node` composition, separate `autolab_gateway`.
- `ansible_agdev` `4798885`: intro once per deployed revision, with a marker
  written only after the post succeeds.
- `pj-clusterintent` `d496e6d`, `1797aa3`: cagent Tool Giving and the published
  Ansible gitlinks.

## `agag_agent` role contract

Placement-visible variables:

- `agag_agent_repo_url`
- `agag_agent_repo_version`
- `agag_agent_repo_dest`
- `agag_agent_agent`
- `agag_agent_instance_name`
- `agag_agent_provider_ollama_base_url`
- `agag_agent_role_profiles_toml`

Controller/runtime variables:

- `agag_agent_zulip_credentials_source`
- `agag_agent_plane_credentials_source`
- `agag_agent_anthropic_api_key_source`
- `agag_agent_claude_code_command`
- `agag_agent_uv_command`
- `agag_agent_role_profiles` (Ansible-native mapping)
- `agag_agent_listener_enabled`
- `agag_agent_service_path`

The role refuses a non-GitHub source, checks out the selected revision, runs
locked dependency synchronization, creates ignored local state, writes
`instance.toml` and the deployment overlay, copies optional controller-only
credentials, installs `agag-<agent>.service`, and starts
`python -m <agent>.listener` in the declared workspace. The service uses a
systemd user unit and the target user has linger enabled. The intro revision
marker makes an interrupted post retryable and a completed post idempotent.

The production contract does not accept object-valued placement variables, so
the desired profile exposes optional role overrides as `role_profiles_toml`.
The autolab wrapper can use the native mapping directly.

## Desired state

Declared records:

- service `agecho`, active;
- placement `agecho-agautolab1` on node `agautolab1`, active,
  `nctl_managed`, profile `agag_agent`, schema `1`;
- GitHub source `https://github.com/iwaag/agecho.git`, revision `main`;
- Python agent and instance `agecho` / `agecho-agautolab1`;
- a dedicated target-user workspace;
- workspace `agecho-agautolab1`, present on that node with the expected GitHub
  identity;
- agent `agecho-agautolab1`, bound to the workspace and placement, Zulip user
  19, expected in `agents` and its own channel.

The service/placement and workspace/agent partial batches each previewed two
creates with no conflicts, then committed atomically. Re-preview of the first
batch was fully unchanged. No secret or controller credential path is stored
in Nautobot or generated inventory.

## cagent transcript

Human-door request:

```text
agecho-agautolab1 is declared on agautolab1; render and apply it. Use the
controller-local credential source for that instance, and report the playbook
changes.
```

First successful application:

```text
ok=17 changed=4 unreachable=0 failed=0 skipped=4
changed: Zulip credential, systemd unit, enable/start, restart
```

Because an earlier git check had already materialized the checkout, this run
exposed the insufficient `git.changed` intro condition. After the revision
marker fix, one local-backend run ended with an empty response before taking
action; a retry through the same human door completed:

```text
ok=20 changed=2 unreachable=0 failed=0 skipped=4
changed: intro post, intro revision marker
```

cagent rendered the inventory and executed the setup playbook itself. Direct
checks afterwards found the unit active/enabled, the expected module entry,
linger enabled, a fresh status file, and the remote intro at revision
`c6e393e`.

## Front exchange

Developer asked Front to greet `agecho-agautolab1` and relay the reply. Front
opened the remote instance's `hello` topic and posted `Hello!`. The instance
replied that this was a simple greeting exchange with no work request. Front
relayed that response to its home topic and marked the task complete. The
instance's mention of Front caused one callback serving at home; it correctly
recognized that the exchange was already complete and did nothing further.

## Observed state

Adding DesiredWorkspace and DesiredAgent was necessary for nodeutils to know
which workspace contained `agag-status.json`; the placement alone carries a
workspace path for Ansible but does not create that probe declaration.

Evidence after a scoped observation refresh:

```text
agent Zulip registration: active; exact expected channels; no errors
workspace: present; identity matched; fresh; no gaps
agent status: present; readable; last_error null; poll fresh
agag_agent idempotent play: ok=19 changed=0 failed=0
nctl host scope: converged
```

## What became of `autolab_node`

The shared deployment behavior is no longer duplicated in the autolab role.
`autolab_node` is now a small composition: it maps legacy placement variables
into `agag_agent`, then invokes `autolab_gateway` for the gateway-only client,
state, unit, and health probe. The retired `autolab-zulip.service` template and
its `agautolab.zulip_listener` entry are gone. Enabling a managed autolab
listener now uses the generic unit and current `agautolab.listener` entry.

## A second placement

Placing another agent, for example forge on a GPU node, still needs:

1. a public GitHub repository and pushed revision;
2. controller-side `agag provision --instance --out` for its dedicated bot;
3. desired service, placement, workspace, and agent records;
4. the target's provider endpoint and any role-profile TOML overrides;
5. a cagent render/apply request with that instance's ignored credential source;
6. a scoped observation refresh and a real Front exchange.

Per-agent Plane identity remains separate work. The current production profile
also represents one scalar `agag_agent` placement per host; multiple different
agag agents on one host would need a placement-list inventory contract or
separate profile groups. macOS/launchd targets remain out of scope.
