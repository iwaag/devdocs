# Phase 6 plan — deployment and local environment reconciliation

Make the Phase 2–5 result reproducible outside the developer checkout: the
agdevworld assistant image carries both pinned harnesses and generates its own
local configuration; the agautolab Ansible path installs OpenCode and Claude
Code without forcing a long-running OpenCode server on every node; agautolab
nodes receive profile/provider settings through intent, not hand-written
files; machine facts stay deployment-supplied. Destructive phase — the legacy
`AUTOLAB_WINDOW_BACKEND`/`AUTOLAB_OLLAMA_URL`/`claude_bin` deployment path is
deleted, not preserved.

Read first: `../p5/report.md` and `../p4/report.md` (what the image and
overlay do today), `../p3/report.md` §"Contract findings" (the `/v1` endpoint
fact), `devpolicy/contracts/agent/spec.md` §1 and §6 (exactly what an overlay
may contain — the deployment templates must produce nothing outside that
scope), `pj-clusterintent/ansible_agdev/vars/deployment_profiles.yml`,
`roles/autolab_node/` and `roles/opencode_agent/`, and
`pj-clusterintent/nctl/docs/add-a-basic-service.md` (how placements carry
config into the generated inventory).

Hard constraints (the roadmap's minimum set — everything else is your call):

- no credentials or generated private payloads committed;
- no silent harness/model fallback;
- keep enough raw output to diagnose a failed agent run;
- no `--dangerously-skip-permissions`, `opencode run --auto`, or equivalent.

The scratch cluster permits rebuilds, restarts, and failed experiments.
Ansible against `agautolab1` and Nautobot desired-state changes are normal
Phase 6 work, not stop-and-ask events; Proxmox operations and anything
outside the named targets still are.

Environment facts you will need:

- **agdevworld image today**: `assistant/Dockerfile` installs pinned
  `opencode-ai@1.18.10` and copies `agents.toml`/`opencode.json`; no Claude
  Code, no auth. The container overlay is the *hand-written, git-ignored*
  `.local/agents.compose.toml`, bind-mounted by `compose.yaml` over
  `/app/.local/agents.local.toml` — a fresh clone cannot start the assistant
  at all. It carries only the opencode command and
  `base_url = "http://host.docker.internal:11434/v1"`. `README_DEV.md`'s
  harness matrix already promises Phase 6 will fix the `claude_code` row.
- **Overlay mechanics**: `assistant/server.mjs:37-38` reads
  `<root>/agents.toml` + `<root>/.local/agents.local.toml`; the provider URL
  flows overlay → `providerBaseUrl` → child env
  `AGENT_PROVIDER_OLLAMA_BASE_URL` → `opencode.json`'s `{env:…}`. Claude runs
  already use a per-run scratch cwd and a generated strict MCP config, so the
  container needs only binary + auth + an overlay entry.
- **Claude auth**: host uses the subscription login under `~/.claude`; the
  container must use `ANTHROPIC_API_KEY` or a mounted credential via compose
  env/secret plus a spec §6 `[local.secrets]` reference. Never bake
  `~/.claude/.credentials.json` into an image.
- **Ansible today**: `roles/autolab_node` clones agautolab from the agstudio
  gitea (`http://agstudio.local:3000/autodev/agautolab.git`), installs the
  `autolab-gateway.service` user unit, and still writes three env vars and a
  pointer file that *nothing consumes anymore* —
  `AUTOLAB_WINDOW_BACKEND=ollama`, `AUTOLAB_OLLAMA_URL` (no `/v1`),
  `AUTOLAB_WINDOW_MODEL` in `templates/autolab-gateway.service.j2`, and
  `.local/agent/claude_bin` in `tasks/main.yml:22-26`. `gateway.py` reads only
  `AUTOLAB_GATEWAY_HOST`/`PORT`; agautolab's `tests/test_legacy_removed.py`
  already bans these names project-side. All four are pure deletion targets.
- **The OpenCode coupling the roadmap names**: `roles/opencode_agent`
  downloads pinned OpenCode 1.18.10 (SHA-256 checked), writes a *global*
  `~/.config/opencode/opencode.json`, then unconditionally installs and
  health-probes an `opencode serve` daemon on `:4096`. Install/config and
  serve are not separable today. Nothing installs Claude Code anywhere.
- **Who gets what**: in the rendered inventory (generation `4e678c5c…`),
  `agautolab1` is in `autolab_nodes` but *not* `node_agent_hosts`, so it is
  the one agent-running node with no `nintent_opencode_ollama_url`. That
  variable reaches `aghub`/`agstudio` through the `node_agent → llm_provider`
  binding (`nctl/src/nctl_core/production/service_dependencies.py:30`, value
  `http://agstudio.home.arpa:11434/v1` — note the `/v1`, which P3 proved
  mandatory). Group membership and host vars come only from Nautobot
  placements + `vars/deployment_profiles.yml`; changing that file changes the
  profile digest and requires `nctl render production` (write path:
  `uv run --project nctl nctl …` from pj-clusterintent, token auto-read from
  `.local/secrets`).
- **nintent duplication trap**: `nintent/nautobot_intent_catalog/models.py:648-660`
  duplicates `PROFILE_BINDING_NAMES` because Nautobot can't read
  `deployment_profiles.yml`. If you add a binding to a profile, both places
  change — and nintent deploys via GitHub (`commit → user pushes →
  docker compose build`), not local mount. Adding plain profile *variables*
  (placement `config` keys mapped in `production/composer.py::map_placement_config`)
  avoids the nintent rebuild; a binding is only needed if the endpoint should
  be derived from a service relation like `node_agent`'s. Choose accordingly.
- **No overlay template exists**: nothing in Ansible generates
  `agautolab/.local/agents.local.toml`, which the loader requires for
  opencode/claude commands, the ollama `/v1` base URL, and per-node
  `[roles.X] profile` overrides. This template is the heart of goal 3.
- **Node state**: `agautolab1.local` currently resolves to 192.168.0.220
  (Nautobot desires .130) — known drift, call it out, don't fix it silently.
  Its gateway runs a stale checkout (a `/jobs` bearer-token era); deploying
  means push current agautolab `main` to gitea first (token at
  `agautolab/.local/gitea/autolab-agent.token`), then run the playbook.
  Ansible via `~/.ssh/ansible_key` is the only controller channel; plain ssh
  is refused. No passwordless sudo exists on cluster hosts — keep everything
  in the user scope (`$HOME`, systemd user units), as the current roles do.
- **agstudio is the developer Mac**: its services (nginx `:8090`, assistant
  `:8091`, gateway `:8791`, agforge `:8092`) start by hand from documented
  workflows in `pj-agdev/.local/devenv.md`. "agstudio services start from
  their documented workflows" is a *verification* item, not an automation
  mandate — do not aim Ansible at this Mac.

## Step 1 — agdevworld image self-sufficiency

Make `docker compose up --build web assistant` work from a clean checkout
with no hand-written `.local` file:

- add pinned `@anthropic-ai/claude-code` to `assistant/Dockerfile` beside
  OpenCode (pin the version you verify against; record it);
- replace the `.local/agents.compose.toml` bind-mount with a *generated*
  overlay — an entrypoint that renders spec-§6 TOML from env
  (`AGENT_PROVIDER_OLLAMA_BASE_URL`, optional claude command, optional
  `[local.secrets] anthropic_api_key_env = "ANTHROPIC_API_KEY"`) is the
  obvious shape; compose supplies the values with sane defaults
  (`host.docker.internal:11434/v1`) and an optional env/secret for the key;
- keep failure honest: `sonnet-front` selected with no key present must
  still die with a clear auth error in the record, not fall back.

Verify: compose build from clean state; `/healthz`; one `local-front` and one
`sonnet-front` request through the container, records + raw transcripts in
`assistant_records`. Update `README_DEV.md`'s matrix row and auth section.
Extend the assistant tests only where the overlay-generation logic is real
code (a unit test on the generator beats a container test).

## Step 2 — Ansible harness roles: decouple, install, delete

In `pj-clusterintent/ansible_agdev`:

- **Decouple**: give `opencode_agent` an install/config-only mode (e.g.
  `opencode_agent_serve: false` gating the service include, handlers, and
  `:4096/doc` probe — or split the role; your call). Existing `node_agent`
  placements on aghub/agstudio keep the server; autolab nodes get CLI only.
- **Claude Code install**: new tasks (in `opencode_agent`'s neighbor or a new
  `claude_code_agent` role) installing a pinned Claude Code CLI into
  `$HOME/.local/bin` (npm global into `$HOME/.local/node` exists on nodes;
  the native installer is also fine — pick what you can checksum or pin).
  Auth on nodes is deployment-supplied and optional this phase: support a
  vaulted/locally-provisioned key file referenced via `[local.secrets]`, and
  let a node without a key simply report `claude_code` unavailable — loudly,
  at selection time.
- **Delete the legacy path**: `autolab_node_window_backend`,
  `autolab_node_window_model`, `autolab_node_ollama_url`,
  `autolab_node_claude_bin`, the `claude_bin` file task, and the three
  `AUTOLAB_*` lines in `autolab-gateway.service.j2`. Also reconsider the
  global `~/.config/opencode/opencode.json` written by `opencode_agent`:
  agautolab passes per-role `OPENCODE_CONFIG`, so a hardcoded global `model`
  is at best redundant on autolab nodes — don't let two configs fight.

Check syntax with `ansible-playbook --syntax-check` and, where useful,
`--check` against `inventories/agautolab.yml` before touching the node.

## Step 3 — intent carries profiles and endpoints to autolab nodes

Give `agautolab1` the facts its loader needs, through the intent path rather
than hand-edited files:

- extend the `autolab_node` entry in `vars/deployment_profiles.yml` with the
  provider endpoint and (optionally) per-role profile overrides — plain
  placement-config variables avoid the nintent rebuild; use the
  `node_agent`-style `llm_provider` binding instead only if you want the
  endpoint derived from the service graph (then update
  `PROFILE_BINDING_NAMES`, `service_dependencies.py`, `contract.py`, and go
  through the nintent GitHub deploy);
- template `~/agautolab/.local/agents.local.toml` from `roles/autolab_node`
  using those host vars: opencode command, claude `command_glob`/command,
  `[local.provider.ollama] base_url` (with `/v1`), optional
  `[roles.X] profile` overrides, optional `[local.secrets]` reference.
  Stay strictly inside spec §6 — the contract's first deployment-generated
  overlay is itself evidence, so note anything §6 made awkward;
- apply the desired-state change with `nctl desired apply -f` (preview, then
  `--yes`), re-render with `nctl render production`, and confirm the new host
  vars appear for `agautolab1` in `inventories/generated/production.yml`.

## Step 4 — deploy agautolab1 and prove a profile-selected run

- Snapshot `nctl status --json` and `nctl drift --host agautolab1 --json`
  before touching anything; keep operation IDs.
- Push current agautolab `main` to the agstudio gitea, then run
  `ansible-playbook -i inventories/agautolab.yml playbooks/agent/setup_autolab_node.yml`
  (plus your opencode/claude install playbook) — or run against the rendered
  production inventory with `--limit agautolab1` if Step 3 landed there;
  either is acceptable, say which you used.
- Evidence to collect on the node (via Ansible ad-hoc or the gateway API):
  gateway `/healthz` healthy under the cleaned service unit; the deployed
  `.local/agents.local.toml` resolving; **one completed agent run selected by
  profile using only deployed configuration** — a small gateway window
  request or a one-iteration coding job via `local-coder` is enough; a
  `sonnet-coder` run too if you provisioned a key, otherwise record the loud
  unavailability instead. Keep the normalized record and raw transcript.
- Re-run `nctl status`/`drift` after; record IDs and call out unrelated drift
  explicitly (the .220/.130 address mismatch is the known candidate; Nautobot
  itself being down was P4's precedent — restart the documented compose stack
  if needed, as P5 did).
- Confirm the agstudio side still starts from its documented workflows
  (compose web/assistant from Step 1; native gateway per `devenv.md`) — this
  is the remaining "agstudio services" evidence item.

## Step 5 — documentation, cleanup gate, and report

- Write the missing Ansible-side agent deployment doc (the `ansible_agdev`
  README has no agent section at all): which playbook/role does what, where
  profile/endpoint intent lives, how a key is provisioned. Update
  `agautolab/agent/README.md`'s deployment pointer and
  `devdocs/README_DEV.md`'s Ansible section if commands changed.
- Grep gate over `ansible_agdev`: no `AUTOLAB_WINDOW_BACKEND`,
  `AUTOLAB_OLLAMA_URL`, `AUTOLAB_WINDOW_MODEL`, `claude_bin`, or
  backend-meaning `ollama` naming remains (the `ollama` *server placement*
  profile and `provider.ollama` are canonical and stay). agdevworld docs no
  longer describe the compose overlay as hand-written.
- Project test suites still pass (`npm test` in agdevworld; agautolab's suite
  including `test_legacy_removed.py` if its docs changed);
  `git diff --check` in every touched repo.
- Write `report.md` in this directory in the P3–P5 shape: outcome,
  implementation, verification table with record/operation IDs, contract
  findings (did spec §6 survive its first generated overlays — this phase's
  core question), deployment findings, and the constraint check.
