# Step 3 report — generic Ansible deployment

Completed on 2026-08-22.

Published `ansible_agdev` commit `ed85f76` with:

- deployment profile `agag_agent` → inventory group `agag_agents`;
- playbook `playbooks/agent/setup_agag_agent.yml`;
- generic `roles/agag_agent`;
- separated `roles/autolab_gateway`;
- a thin `autolab_node` composition role that invokes `agag_agent` for the
  shared checkout/runtime/overlay/listener behavior and `autolab_gateway` only
  for the gateway-specific surface.

The placement-visible profile variables are `repo_url`, `repo_version`,
`repo_dest`, `agent`, `instance_name`, `provider_ollama_base_url`, and
`role_profiles_toml`. The role also accepts controller-only sources through
`AGAG_AGENT_ZULIP_CREDENTIALS_SOURCE`,
`AGAG_AGENT_PLANE_CREDENTIALS_SOURCE`, and
`AGAG_AGENT_ANTHROPIC_API_KEY_SOURCE`; these are not inventory or Nautobot
fields. An Ansible-native `agag_agent_role_profiles` mapping remains available
to the autolab wrapper. The production schema currently supports scalar/list
types but not objects, hence the placement-facing TOML string.

The role enforces a GitHub HTTPS source, runs locked `uv sync`, writes ignored
instance/overlay/credential files, installs `agag-<agent>.service` with
`python -m <agent>.listener`, starts it as a systemd user service, and posts
the intro when the checkout revision changes. Service actions are skipped in
Ansible check mode so a not-yet-created unit can still be reviewed safely.

The old `autolab-zulip.service` template was removed entirely. A future
autolab listener managed through the wrapper uses the current
`python -m agautolab.listener` convention supplied by `agag_agent`; the
gateway is now isolated in its own role.

Verification:

```text
setup_agag_agent.yml syntax check: passed
setup_autolab_node.yml syntax check: passed
deployment profile contract: 14 profiles validated
nctl profile/control-loop tests: 8 passed
agag agent check+diff on agautolab1: failed=0, unreachable=0
autolab wrapper check+diff on agautolab1: failed=0, unreachable=0
```

The new-checkout dry run first exposed normal Ansible check-mode limitations:
command modules validate `chdir` although the preceding git task does not
actually create it, and systemd cannot see a unit only simulated by a template
task. The final check used an existing remote checkout solely to exercise the
role without mutation and skips start/restart in check mode. The real placement
uses a dedicated agecho workspace in later steps.
