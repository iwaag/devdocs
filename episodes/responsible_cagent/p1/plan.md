# Phase 1 Plan — Bring dependency / in-system services under cluster intent

Source: [braindump.md](../braindump.md) phase 1.

Goal: enumerate every dependency service and in-system service of pj-agdev /
pj-clusterintent, give each one a proper always-on setup, and register the keepers
in cluster intent. All work is done directly by the Omni Agent; do not route work
through cagent.

Ground rules (deliberately few):

- This is an experimental, non-production environment. Backward compatibility is
  NOT required; deleting or rebuilding scratch state is fine.
- Do not put secrets/tokens into tracked files. Secret locations:
  `pj-clusterintent/.local/secrets` (Nautobot token),
  `pj-agdev/.local/plane-credentials.env`, `~/.local/state/cagent/human_token`.
- Structured desired-state changes go through `nctl desired apply` (preview,
  then `--yes`), not hand-edits in the Nautobot UI, so they stay reproducible.
- Everything else is implementer's discretion. Prefer acting and reporting over
  asking.

Write `report1.md` … `report5.md` (one per step) in this folder.

## Baseline findings (2026-08-12, read-only survey)

- 12 placements registered in intent (`nctl relations --json`): agautolab×2,
  cagent-snake-e2e, comfyui, dnsmasq, heartbeat-cron, node-agent×3, ollama,
  snake-web, swarmui. **All 12 are drifting with `service_observation_stale`**
  — nodeutils dumps are 32–315h old against a 24h threshold. Registration alone
  is not enough; observation freshness is part of this phase (Step 5).
- Running on agstudio but NOT registered: redis (`service_scripts-redis-1`,
  from the legacy compose at `~/services/service_scripts`), `my_postgres_db`,
  the Nautobot stack itself (`pj-clusterintent/devenv/nautobot/`), gitea
  (`autodev-gitea`, port 3000 — the deploy source for agautolab1), nautobot-minio
  (:9100, signs presigned image URLs), Plane stack (:8290), keycloak, rabbitmq,
  hatchet, adminer, redis-insight, excalidraw, portainer.
- Native hand-started processes with no supervision (they die on reboot):
  agforge request service `agforge/service/serve.sh` (:8092), autolab gateway
  `agautolab/agent/gateway.py` (:8791, nohup), cagent human entrance (:8789).
- `nctl status` is healthy: Nautobot reachable, worker running, submodules clean.

## Step 1 — Inventory and classification

Deliverable: `p1/inventory.md`, one row per service.

Enumerate services on agstudio (docker ps + compose files + native processes +
launchd), and on agautolab1 / agpc / aghub (via nodeutils dumps in
`/var/lib/nodeutils`, `nctl relations`, and ansible ad-hoc if needed; aghub has
no passwordless sudo — read-only checks only there).

For each service record: where it is defined, how it currently starts, who
depends on it, and a decision:

- class: `dependency` / `in-system` / `relic`
- plan: `nctl_managed` / `manual` placement / retire / out-of-scope

Hints:
- The interesting judgment calls are in `~/services/service_scripts` and the
  standalone containers: keycloak, rabbitmq (:5673), hatchet (+postgres-hatchet),
  adminer, redis-insight, excalidraw, my_minio, portainer. Grep pj-agdev and
  pj-clusterintent for their hostnames/ports to prove or disprove dependency.
  Anything with no consumer is a relic — mark it retire, no nostalgia.
- Known consumers to verify, not rediscover: nautobot → my_postgres_db + redis;
  agautolab1 deploys from gitea; agforge presigns against nautobot-minio :9100;
  Plane is used by autolab Plane-reporting.
- Present the finished table to the user once before Step 2 (this is the one
  intentional checkpoint of the phase).

## Step 2 — Always-on setup, one pattern per kind

Apply the Step 1 decisions. No compatibility with the old startup style is owed.

- Docker services worth keeping: move each out of relic composes into its own
  small compose (suggested home: the owning project's `devenv/`, e.g. a
  dedicated redis compose next to `devenv/nautobot/`), set
  `restart: unless-stopped`, confirm Docker Desktop starts at login. Update
  consumers' connection settings (`devenv/.env` `NAUTOBOT_REDIS_HOST` etc.) and
  then stop the legacy copy. Data worth keeping (postgres volumes) can be
  reused by pointing the new compose at the existing volume, or dumped and
  restored — implementer's choice; `.local/backups/` has the dump precedent.
- Native processes worth keeping (agforge :8092, autolab gateway :8791, cagent
  :8789): wrap each in a launchd agent (`~/Library/LaunchAgents/`,
  `RunAtLoad` + `KeepAlive`), logs to the project's `.local/`. Kill the nohup
  habit. Note macOS local-network privacy: a fresh launchd-started process may
  need the same LAN permission the shell-started one had.
- Relics: stop and remove containers; leave volumes unless clearly junk.

## Step 3 — Nautobot self-hosting posture

Nautobot is the intent store, so intent cannot report Nautobot being down
(chicken-and-egg). Decide and implement:

- Auto-recovery: `restart: unless-stopped` across the nautobot compose and its
  new postgres/redis composes; verify start order tolerance (nautobot retries
  until DB/redis are up — confirm rather than assume).
- Register the whole stack as placements on agstudio anyway (management_mode
  `manual`) so it exists in the map cagent explains.
- Write the cold-start runbook (what to start, in what order, how to verify)
  into `report3.md`.

## Step 4 — Register in cluster intent + endpoint/dnsmasq sync

- Encode Step 1's keeper rows into `.local/desired-state.yaml`, then
  `uv run --project nctl nctl desired apply -f .local/desired-state.yaml`
  (preview → `--yes`). Verify with `nctl drift` and `nctl relations`.
- agautolab1 address cleanup, folded in here: desired says 192.168.0.130
  (inside the desired ip-range DHCP reservation area) while mDNS/actual is
  192.168.0.220, and nctl currently derives connection_address=.220 and calls
  the node converged. The dnsmasq placement carries `service_config_mismatch`,
  which is likely the unapplied reservation. Confirm the desired endpoint entry,
  then `nctl render dnsmasq` to inspect, and prefer `nctl reconcile --yes` (or
  `nctl apply dnsmasq`) to push the config; a DHCP lease renewal / reboot of
  agautolab1 may be needed before it actually moves to .130. Afterwards verify
  resolution and that the gitea→ansible deploy path still works
  (`ansible-playbook … setup_autolab_node.yml --limit agautolab1`).

## Step 5 — Observation freshness + final verification

- Fix the universal `service_observation_stale`: give nodeutils observation a
  schedule (extend the existing `heartbeat-cron` placement or add a cron/launchd
  job; `nctl` side is `nctl … --refresh-observation` / `observation.py`). Target:
  all dumps younger than the 24h threshold without manual action.
- Full restart test on agstudio (reboot, or at minimum Docker restart + logout
  of launchd agents): everything decided as keep comes back by itself.
- `nctl drift` shows no gap other than genuinely-out-of-scope items; screenshot
  or paste the summary into `report5.md`.
- Close with `p1/report.md`: what is now managed, what was retired, leftovers
  handed to phase 2.
