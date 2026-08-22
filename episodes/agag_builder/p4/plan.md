# agag_builder p4 plan — an agag agent on another node, placed by intent

Goal: a generated agag agent runs on a node other than agstudio, as a
permanent service, with its `.local/` files (instance name, Zulip bot
credentials, harness overlay) put there by Ansible from desired state — and
cagent is the one that applies it when asked. After this, "put agent X on
node Y" is a desired-state change plus one request to cagent.

Success criteria:

1. A deployment profile `agag_agent` exists in nintent/nctl
   (`ansible_agdev/vars/deployment_profiles.yml`), an Ansible role of the
   same name, and `playbooks/agent/setup_agag_agent.yml`. The role is the
   generalization of `roles/autolab_node`: repo from GitHub, `uv sync`,
   `.local/instance.toml`, `.local/zulip.env`, `.local/agents.local.toml`,
   a systemd user unit running `python -m <agent>.listener`.
2. `agecho-agautolab1` is a live instance on the agautolab1 VM: own bot,
   own channel, intro posted from that node, listener under systemd, and
   `agag-status.json` observed by nodeutils (so Nautobot shows it alive).
3. The placement was declared with `nctl desired apply`, rendered with
   `nctl render production`, and **applied by cagent** on a request through
   its human door; cagent reports what the playbook changed.
4. Front greets `agecho-agautolab1` and relays the reply, exactly as p1/p3.
5. `roles/autolab_node` either becomes a thin wrapper over `agag_agent` or
   is deleted, and the agautolab1 unit no longer points at the retired
   `agautolab.zulip_listener` entry.

Decisions already made (discussion after p3):

- **Direction is desired → node.** Agents do not push their paths to cagent;
  the placement's `config` carries what the role needs, and the role derives
  paths from `workspace/.local/` by convention (p1–p3 made every agent's
  `.local/` identical in shape).
- **Bot creation stays with `agag provision`**, run on the controller
  (agstudio, where `provisioner.env` is). The role only copies the resulting
  `zulip.env` from `pj-agdev/.local/zulip/<instance>.env`. Provision once,
  distribute many.
- **Deployment source is GitHub, never Gitea or a local path**
  (`roles/autolab_node/defaults/main.yml` header; memory
  `deploy-from-github-never-gitea`). agecho therefore gets a GitHub remote
  first (Step 1). agping lives inside runsmoke1's Gitea repo and is not a
  candidate this phase.
- **cagent applies, humans/Omni decide.** cagent's authenticated doors
  already have `run` with only a destroy-class denylist; this phase adds no
  tool, only a paragraph in `cagent/agent/AGENTS.md` saying the playbook
  exists and what it is for (Tool Giving). The window door stays read-only.
- No backward compatibility: `autolab_node`'s per-role `*_profile` variables
  and the gateway stay only if the new role can express them; otherwise the
  gateway keeps its own small role and the listener moves.
- Plane per-agent accounts still out of scope; the shared key is copied if
  the placement asks for it.

Constraints: secrets in `.local/` / controller env sources only, never in
inventory or Nautobot; GitHub push before any node pulls; no passwordless
sudo on aghub (`aghub-proxmox-host` memory) — none is needed, everything is
user-space on agautolab1; cost is not a concern.

## Facts checked at planning time

- **Listener entry drift already exists**: `roles/autolab_node/templates/autolab-zulip.service.j2`
  runs `python -m agautolab.zulip_listener`, but since p2 the entry is
  `agautolab.listener` (`zulip_listener.py` has no `main` any more). The
  agautolab1 placement has `zulip_listener: false` so nothing broke, but the
  unit is wrong. Criterion 5 fixes it.
- `roles/autolab_node` already does every file the new role needs except
  `instance.toml`: `agents.local.toml.j2` (harness command paths, provider
  URLs, per-role profile overrides), `plane.env.j2`, `zulip.env` copied
  from `AUTOLAB_NODE_ZULIP_CREDENTIALS_SOURCE`, anthropic key to
  `.local/secrets/`, systemd user units, `claude_code_agent` role for the
  CLI. Read it first; most of the new role is renaming variables.
- Deployment profiles: `vars/deployment_profiles.yml` maps a profile's
  `variables` to Ansible variables with types; `DesiredServicePlacement`
  (`nctl/src/nctl_core/sources/desired.py:278`) carries `deployment_profile`,
  `instance_name`, `config`, `node_id`. `nctl render production` turns
  placements into `inventories/generated/production.yml` host vars and the
  group named by the profile. The `autolab_node` entry at line ~200 is the
  template to copy.
- `DesiredAgent` (`desired.py:323`) has slug, zulip_user_id, plane_user_id,
  channels, workspace, placement — enough to link the agent to its
  placement; it does not need path fields. Whether to create a
  `DesiredAgent` for agecho-agautolab1 or only the placement: do the
  placement first, add the agent record if nodeutils/Nautobot need it to
  show liveness (`nodeutils_collect.py` reads `<workspace>/.local/agag-status.json`,
  so the workspace must be declared somewhere it can find — check what it
  reads today for agautolab1).
- cagent (`pj-clusterintent/cagent`): human door `:8789` bearer token in
  `~/.local/state/cagent/human_token`; `run` tool = shell in the repo root;
  `agent/AGENTS.md` is its guide; denylist in `src/cagent_api/agent_runner.py:52-77`
  (playbooks matching `destroy_*` are blocked, `setup_*` is not). `nctl
  reconcile --yes` is the precedent for cagent driving Ansible. Ansible runs
  from `pj-clusterintent/ansible_agdev` with the generated inventory
  (`devdocs/README_DEV.md` "Ansible Commands").
- agautolab1 = VMID 109 on aghub, Ubuntu, user-space `uv`/`claude`/`node`
  under `~/.local` (`autolab_node_service_path`); systemd user units with
  `WantedBy=default.target` — check `loginctl enable-linger` is on, or the
  listener dies at logout.
- agecho: `/Users/eiji/projects/agecho`, 2 local commits, no remote.
  Instance name is in ignored `.local/instance.toml`; the second instance
  will be `agecho-agautolab1`, a different bot from `agecho-agstudio1`
  (instance name = bot = channel, p1 rule). `agag provision` takes the root
  and reads the instance from `.local/instance.toml` — for provisioning a
  *remote* instance from the controller it needs `--instance <name>` (or a
  scratch root). Small addition, Step 2.
- The generated `listen.sh` is `exec uv run python -m <agent>.listener`;
  the unit can call that or the venv python directly as autolab's does.

## Step 1 — agecho on GitHub

Create `iwaag/agecho` (public, like the others), push. Nothing else changes
in the repo. Hint: `gh repo create iwaag/agecho --public --source
/Users/eiji/projects/agecho --push`.

## Step 2 — provision the second instance from the controller

Add `--instance <name>` to `agag provision` (pyagag) so it can provision
without a local `.local/instance.toml`, writing `zulip.env` to `--out
<path>` (default `<root>/.local/zulip.env`). Run:

```
AGAG_ZULIP_ADMIN_ENV=pj-agdev/.local/zulip/provisioner.env \
  agag provision agecho --instance agecho-agautolab1 \
  --out pj-agdev/.local/zulip/agecho-agautolab1.env
```

That file is the credential source the role copies. Push pyagag; no
consumer lock bump is needed for this (only the controller runs it), but
bump agautolab anyway if you touch anything else.

## Step 3 — `agag_agent` role + playbook + profile

`ansible_agdev/roles/agag_agent/` from `autolab_node`, variables renamed to
`agag_agent_*`: `repo_url`, `repo_version`, `repo_dest`, `agent` (python
package / short name), `instance_name`, `zulip_credentials_source`,
`plane_credentials_source` (optional), `anthropic_api_key_source`,
`claude_code_command`, `uv_command`, `provider_ollama_base_url` (optional),
`role_profiles` (a dict, replacing five `*_profile` scalars).

Tasks, in order: git checkout from GitHub → `uv sync --frozen` →
`.local/` dirs → `instance.toml` (new: `name = "{{ instance_name }}"`) →
`agents.local.toml` from the existing j2 → `zulip.env` copy →
optional plane/anthropic → systemd user unit
`agag-{{ agent }}.service` running `{{ repo_dest }}/.venv/bin/python -m {{ agent }}.listener`
with `WorkingDirectory={{ repo_dest }}` (that is where `agag-status.json`
lands) → enable + start → post the intro once
(`python -m {{ agent }}.intro`) when the checkout revision changed
(`register`/`changed_when` on the git task).

`playbooks/agent/setup_agag_agent.yml`: hosts `agag_agents`, roles
`claude_code_agent`, `agag_agent`. Profile `agag_agent` in
`deployment_profiles.yml` with group `agag_agents` and the variables above
(`role_profiles` as type `object` if the schema allows, else one string
variable `role_profiles_toml`).

Then `autolab_node`: either re-express autolab as an `agag_agent`
placement plus a separate `autolab_gateway` role for the gateway, or keep
`autolab_node` and fix only its unit template. The first is cleaner and is
what criterion 5 wants; do it unless the gateway's PATH/env coupling makes
it a day's work — then fix the unit and note the rest.

Test offline first: `ansible-playbook --check --diff` against agautolab1
with a hand-written inventory entry, before any Nautobot change.

## Step 4 — declare the placement

`nctl desired apply` (or the Nautobot UI) for a placement on agautolab1
with `deployment_profile: agag_agent`, `instance_name: agecho-agautolab1`,
config `{repo_url: https://github.com/iwaag/agecho.git, agent: agecho,
repo_dest: ~/agecho, instance_name: agecho-agautolab1,
zulip_credentials_source: <controller path>}` — check which of these
belong in config vs. role defaults; secrets sources are controller env,
never Nautobot. `nctl render production` → verify the host var block for
agautolab1 in `inventories/generated/production.yml`.

## Step 5 — cagent applies it

One paragraph in `cagent/agent/AGENTS.md`: the playbook exists, what
placement it serves, the exact command
(`ansible-playbook -i inventories/generated/production.yml playbooks/agent/setup_agag_agent.yml --limit <node>`),
and that `nctl render production` comes first. Restart cagent.

Through the human door (web chat or `curl` with the bearer token):
"agecho-agautolab1 is declared on agautolab1; render and apply it." Expect
cagent to render, run the playbook, and report changed tasks. If it asks
for confirmation, that is fine; if it refuses, the AGENTS.md paragraph is
what to fix, not the denylist.

Verify on the node: `systemctl --user status agag-agecho`,
`~/agecho/.local/agag-status.json` fresh, `#agents/intro-agecho-agautolab1`
posted, `#agecho-agautolab1` channel exists.

## Step 6 — Front greets it

"say hello to agecho-agautolab1 and tell me what it replied" → reply at
home. Then check Nautobot/nodeutils shows the instance alive (the
`agag-status.json` path); if it does not, that is the `DesiredAgent`/
workspace gap from the facts — add the record and note what was needed.

## Step 7 — record

`p4/report.md`: the role's variable list, the placement as declared, the
cagent transcript (request → playbook output → report), Front's exchange,
nodeutils/Nautobot evidence, and what `autolab_node` became. Note what a
second placement (say forge on a GPU node) would still need — that is the
next braindump.

## Out of scope

- Running `agag init`/`provision` on the node itself (the controller holds
  the provisioner key; fine).
- macOS/launchd as a target of the role (agstudio stays hand-installed).
- Per-agent Plane accounts; Front's own intro; the supercoder
  "correction outranks task text" guide line (agent_standardize).
