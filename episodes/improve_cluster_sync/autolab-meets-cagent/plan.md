# autolab-meets-cagent — plan

Source: [braindump.md](braindump.md). Feasibility check: 2026-08-09, against live
`nctl desired export` / `nctl drift --json` and the cagent/nintent/nctl/ansible sources.

## Premises

- Experimental cluster, no production workload. Backward compatibility is NOT
  required anywhere in this episode; break APIs, configs, and frozen contracts
  freely (update the contract docs when you do, don't keep dual paths).
- Prohibitions are deliberately minimal — everything not listed is implementer
  discretion:
  1. No secrets (tokens, keys) in Git-tracked files or in request evidence.
  2. cagent keeps denying irreversible operations (`--allow-destroy`,
     `nctl prune`, braindump purge/review-delete, proxmox destroy playbooks).
     Everything else may be opened up.
  3. Don't claim a step complete without the named evidence (probe the real
     endpoint/process, record where the evidence lives).
- Out of scope (per braindump line 7): desired_workspace registration of
  autolab job workspaces. Only *services* are registered; workspaces stay
  autolab-managed.
- Write a short `report[N].md` per step (repo policy: reports strongly
  recommended).

## Settled decisions (do not re-litigate during implementation)

- **cagent becomes a writer.** The cluster_agent roadmap's prohibition #1
  ("no cluster mutation from node-originated requests") is repealed for this
  cluster: the injection boundary moves from "cannot write" to "cannot
  destroy". Desired state is recoverable (`.local/backups/` Postgres dumps +
  `nctl desired export` snapshots), so this is an accepted trade.
- **Autolab nodes authenticate with the human-entrance bearer token**, not
  mTLS enrollment. Per-node identity in evidence is knowingly given up (all
  requests label as `operator`). If per-node identity is wanted later, the
  right fix is an Ansible playbook automating CSR→sign→ledger→cert — an ENT
  episode, not this one.
- **Service type = deployment profile.** `DesiredService` has no type field
  and needs no schema change; the new "browser game" type is one new profile
  in `ansible_agdev/vars/deployment_profiles.yml`.
- **ON/OFF = a `run_state` config knob** (`started|stopped`) interpreted by
  the serve role's playbook. `desired_state: disabled` means "leave production
  inventory", which is not the same thing; nctl has no stop-a-service action
  kind, and we are not adding one.

---

## Step 1 — agautolab1 into production scope

Goal: `nctl drift` no longer reports `waiting_for_manual_initial_access` for
agautolab1, and production composition includes it.

- That state is **derived, not a flag**
  (`nctl/src/nctl_core/compute/manual_initial_access.py`): it clears
  automatically on the first nodeutils observation of `agautolab1` /
  `agautolab1.local`. No manual state switch exists or is needed.
- SSH already works: `~/.ssh/ansible_key` via the static inventory
  `ansible_agdev/inventories/agautolab.yml` (setup_autolab_node succeeded
  2026-08-08). So the only real work is one observation+ingest cycle, e.g.
  `nctl reconcile agautolab1 --refresh-observation --yes` (dry-run first), or
  the mDNS bootstrap inventory path if reconcile refuses the node before
  observation.
- Known IP situation, don't chase it as a bug: desired endpoint is
  `dhcp_reserved 192.168.0.130`, actual lease is `.220` because the
  reservation is served by agdnsmasq's DHCP and agdnsmasq is currently
  down/stale (`stale_actual_data`, `service_config_mismatch` in drift;
  unresponsive per localenv memo). mDNS (`agautolab1.local`) is guest-side
  avahi and works regardless — use it as the access path.
- Optional sub-step (fine to defer to the end of the episode): revive
  agdnsmasq, `nctl reconcile agdnsmasq --yes`, renew agautolab1's lease,
  confirm `.130`. Expect a temporary endpoint-IP drift after observation
  until this lands; that drift is correct, not a defect.

Evidence: `nctl drift --host agautolab1 --json` showing production
`state: included` (or at least the waiting reason gone), plus the reconcile
operation ID.

## Step 2 — relax cagent to "everything but irreversible"

Goal: cagent can run `nctl desired apply -f - --yes` and
`nctl reconcile ... --yes` end-to-end; destroy-class operations still refused.

- `cagent/opencode/config.json.template` (and the live config it was copied
  to): delete the `*nctl*reconcile*--yes*` and `*nctl*desired*apply*--yes*`
  deny rules; add denies for `*--allow-destroy*`, `*nctl*prune*`,
  `*braindump*purge*`, `*review-delete*`, `*playbooks/proxmox/destroy_*`.
  Exact glob syntax is OpenCode's — verify a deny actually fires by asking
  cagent to run a denied command before trusting it.
- Rewrite the "What you must never do" section of `cagent/opencode/AGENTS.md`
  to the new boundary (irreversible ops → human; everything else allowed;
  still never expose the OpenCode port, still never put secrets in
  responses). Keep the style rules.
- **Restart `./opencode/start.sh`** — AGENTS.md changes only affect sessions
  created after restart. `cagent-api` itself needs no restart.
- Record the repeal: one short note in
  `pj-clusterintent/devdocs/vision/cluster_agent/roadmap.md` (prohibition #1
  relaxed, date, pointer to this episode). Breaking-change policy applies; no
  compatibility shims.
- Safety net before first live write: take one `nctl desired export`
  snapshot and confirm a recent Postgres dump exists in `.local/backups/`.

Evidence: transcript of cagent executing a harmless `desired apply --yes`
(e.g. a no-op re-apply of the current export) and being denied on a
`--allow-destroy` attempt.

## Step 3 — give autolab nodes a cagent client path

Goal: the autolab agent on agstudio and agautolab1 can submit a cagent
request and poll the result, with zero enrollment ritual.

- Auth: reuse `~/.local/state/cagent/human_token` against the human entrance
  `https://agstudio.local:8789` (server-only TLS, self-signed — clients must
  accept the CA or skip verification; agdevworld's `cluster:fetch` script is
  a working reference caller).
- Distribution: one task in the `autolab_node` role (or a tiny dedicated
  role) copying the token file to the node, mode 0600, never tracked.
  agstudio already holds it natively.
- Client shape is implementer's choice: a curl snippet documented in
  agautolab's CHARTER.md resources section is enough; a small wrapper script
  is fine too. Remember the contract: requests are async — submit, poll
  `GET /requests/{id}`, allow several minutes.
- Update `agautolab/agent/CHARTER.md` "Resources on this machine" so the
  mediator agent knows cagent exists, what to ask it, and when (mission
  completion of a resident-service project → report/register).

Evidence: from agautolab1, one full request→completed cycle against cagent.

## Step 4 — `static_web_app` profile + serve role, proven on agstudio

Goal: a gitea-hosted static browser game runs as an nctl-managed service on
agstudio via `nctl reconcile`.

- Reality check of the payloads: autolab's shipped projects (snake-web,
  snake-web-b, othello-web, gallery-web at
  `http://agstudio.local:3000/autodev/*.git`) are dependency-free static
  sites (`node --test` gates, no build step). Serving = clone + static file
  server. Keep the role that thin.
- `ansible_agdev/vars/deployment_profiles.yml`: add profile
  `static_web_app` — `group: static_web_app_hosts`, `config_schema_version:
  "1"`, variables approximately `repo_url` (string, required), `repo_version`
  (string), `port` (integer, required), `run_state` (string). Add the
  matching `deployment_profile_reconciliation` entry with `kind: playbook`.
  Targets span macOS (agstudio) and Linux (agautolab1) — use
  `playbook_by_os` (precedent: `nomad_client`).
- The role: clone/update from gitea, then run a static server pinned to
  `port`, honoring `run_state`. Process supervision is discretionary
  (launchd/systemd-user templates exist as precedents —
  `agautolab/devenv/systemd/autolab@.service`; a nohup'd `python3 -m
  http.server`-class server is acceptable for this episode). Multiple
  placements of different games on one node must coexist — key paths and
  service names by instance/config, not by profile.
- Ports: `DesiredEndpoint` already has `protocol`/`port` fields — declare one
  `endpoint_type: service` endpoint per placement so ports are visible in
  desired state (this is the braindump's port-collision pain). Note honestly:
  there is **no automated port-collision detection** in nctl (no unique
  constraint, no drift comparator); declared visibility is the deliverable,
  a collision comparator is optional stretch work, not required.
- First placement by hand (batch document, `nctl desired apply` preview →
  `--yes`), e.g. snake-web on agstudio on a port that avoids the known
  occupied ones (3000, 5173, 8090–8092, 8788–8791, 9100). Then
  `nctl reconcile agstudio --yes` and prove the game is served (curl the
  port). Flip `run_state` to `stopped`, reconcile, prove it stopped; flip
  back.
- Observation: decide whether nodeutils/drift can see the service
  (`observe_only`-style `checks: [kind: http]` on the reconciliation entry is
  the cheap option — precedents: ollama, swarmui). At minimum the HTTP check;
  deeper actual-state modeling is discretionary.

Evidence: drift showing the placement `applied` and converged; curl output
before/after the run_state flip.

## Step 5 — register agautolab itself as a desired service

Goal: braindump line 9 — the autolab gateway on agstudio and agautolab1 stops
being invisible infrastructure.

- Add profile `autolab_node` reusing the existing role
  `ansible_agdev/roles/autolab_node` and playbook
  `playbooks/agent/setup_autolab_node.yml` (`kind: playbook`). Two known gaps
  between the role and reality, both fine to resolve either way as long as
  the resolution is recorded:
  - the role clones from GitHub; the actual agautolab1 deploy path is the
    agstudio gitea (`http://agstudio.local:3000/autodev/agautolab.git`) —
    parameterize `autolab_node_repo_url` via placement config or change the
    default;
  - on agstudio the gateway is started by hand (no launchd) — a `manual`
    `management_mode` placement (converged on presence alone) is the honest
    low-effort declaration for agstudio if you don't want to automate macOS
    supervision now; use `nctl_managed` for agautolab1 where the systemd
    template applies.
- Declare `desired_service: agautolab` (lifecycle `active` — it is never
  promoted for you) + two placements + service endpoints for port 8791.
- Reconcile and confirm no spurious drift on repeat runs.

Evidence: `nctl relations --json` showing the agautolab service with two
placements and their state.

## Step 6 — the end-to-end scenario

Goal: the braindump's storyline, unassisted: autolab completes a
resident-service mission → reports to cagent → cagent registers the
desired_service + placement → the game is deployed and toggled on another
node via cagent.

- Give the autolab mediator (CHARTER.md addition from Step 3) the
  instruction: on completing a mission that produced a resident/serve-able
  project, ask cagent to register it — service slug, gitea repo URL, target
  node, port. The mediator supplies facts; **cagent owns composing the batch
  document** (it has the repo checkout and `nctl/docs/desired-partial-batch.md`
  as its manual — point its AGENTS.md at that recipe and
  `add-a-basic-service.md`).
- Run one real (or replayed) mission on agautolab1, e.g. a clone of an
  existing game repo as a new project, and let the chain fire: cagent
  `desired apply --yes` → `reconcile agautolab1 --yes` → game served on
  agautolab1.
- Then through cagent conversation only: `run_state: stopped` →
  reconcile → confirmed off; back on. That closes braindump line 5.
- Injection posture (accepted, but verify once): ask cagent, inside a
  mission-shaped request, to run an `--allow-destroy` command; confirm the
  deny fires.

Evidence: the cagent session transcript (request IDs), the operation IDs of
the applies/reconciles, and curl proof of the game on agautolab1 before/after
the toggle.

---

## Useful facts collected during planning

- Live desired state today: 6 nodes (agautolab1/agdnsmasq `service_host`,
  rest devices), 11 services, no `desired_workspace` for autolab jobs.
  Drift totals: 14 converged / 8 unknown (mostly `*_observation_stale`).
- `DesiredServicePlacement` uniqueness is `(desired_service,
  instance_name)`; `config` must be a JSON object; `management_mode:
  manual` = converged on presence alone, no run-state gaps planned.
- The batch endpoint (`nctl desired apply`) is the **only** desired-state
  writer; nintent's UI is read-only. GraphQL is read-only.
- cagent request timing: multi-command turns have taken ~4 minutes;
  `CAGENT_TURN_TIMEOUT_SECONDS` (default 300) may need raising once cagent
  runs reconciles inside a turn.
- cagent's working directory is the pj-clusterintent superproject root, so
  it can read every recipe and run `uv run --project nctl nctl ...` as-is.
- Keep `.local/desired-state.yaml` workflows intact: after cagent starts
  writing, `nctl desired export -o .local/desired-state.yaml` re-syncs the
  operator file (it will drift from the DB by design).
- agstudio rule (memory): never run skip-permissions jobs on this Mac;
  autolab's charter already bans `--dangerously-skip-permissions` — keep it
  banned in any new job configs this episode creates.
