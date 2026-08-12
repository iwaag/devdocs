# Phase 1 Extra 1 report — Phase 1 leftovers closed

Status: complete (2026-08-12). Single consolidated report per user request.

Final drift: `summary: converged=35` — zero warnings, zero errors, zero
unknown. Nautobot, Gitea, agdevworld assistant and agforge all answered their
health checks after the cleanup.

## Step 1 — Catalog noise

- Deleted the five catalog-only desired_service rows (`haos`, `grafana`,
  `nomad`, `prometheus`, `prometheus-node-exporter`) via
  `nctl desired apply` (`op: delete`; dry-run showed 5 clean deletes, 0
  conflicts, then committed). User had confirmed them as old-experiment
  leftovers.
- agbach is up again. One host-scoped
  `nctl reconcile agbach --refresh-observation --max-rounds 1 --yes`
  collected fresh evidence; `stale_actual_data` cleared and agbach is now
  converged. Added `agbach` to the default host list of
  `devenv/observation/refresh-node-observations.sh`, so the six-hour
  launchd observation job keeps it fresh from now on.

## Step 2 — IPv6-RDNSS posture

- agpc: the user disabled IPv6. Verified persistent — the NetworkManager LAN
  profile has `ipv6.method: disabled`, no global IPv6 address, DNS server
  list is exactly `192.168.0.2`, `agstudio.home.arpa` resolves in ~1 ms, and
  the Ollama binding probe returns HTTP 200.
- Exposure survey of the other nodes found one more exposed host:
  **agautolab1** still listed the router's RDNSS-advertised public IPv6 DNS
  as a fallback resolver. agdnsmasq (no global IPv6) and aghub (static
  `/etc/resolv.conf`) are not exposed. agbach could not be checked (SSH to
  192.168.0.120 times out; nodeutils observation works through nctl's own
  channel, so this does not affect operations).
- Fix generalized into ansible: `common_linux_baseline` now writes
  `/etc/netplan/99-disable-ipv6-ra.yaml` (`accept-ra: false` on the primary
  interface) on any netplan host, with a `netplan apply` handler. Applied the
  same drop-in to agautolab1 now: its resolver list is down to `192.168.0.2`
  only, no global IPv6 address, `home.arpa` resolution correct, autolab
  gateway `/healthz` 200.
- Ansible changes are uncommitted in pj-clusterintent
  (`roles/common_linux_baseline/*`, `devenv/observation/*`); commit when
  convenient.

## Step 3 — Relic data deletion (pre-approved)

Verified before deleting: the live PostgreSQL uses the bind mount
`~/services/postgres_data` (untouched); the live Redis is `nintent-redis` on
`nintent-dependencies_redis_data` (untouched); no running container
referenced anything deleted below.

Deleted:

- 16 Docker volumes: 12 dangling anonymous volumes (one held a 91 MB
  orphaned PostgreSQL data dir, the rest ~0 B),
  `service_scripts_nautobot_static` (156 MB), `service_scripts_nautobot_media`,
  `service_scripts_portainer_data`, and `nautobot-dependencies_redis_data`
  (leftover of the p1 Compose-project rename to nintent-dependencies).
- The legacy `~/services/service_scripts/` compose directory (nothing live
  anchored to it; only a stale comment in `devenv/nautobot/docker-compose.yml`
  still mentions the name).
- Legacy bind data under `~/services/`: `hatchet_postgres_data` (607 MB),
  `hatchet_rabbitmq_data`, `hatchet_config`, `hatchet_certs`, and the legacy
  MinIO store `minio_data` (88 MB — old-experiment buckets `agdev`/`agvideo`/
  `sahred`/`stage`/`staged`, last touched 2026-01-22; the live :9100 store
  holds only `agforge` and `nctl-outbox`, and nothing references the legacy
  store).

Roughly 1 GB reclaimed. Kept: `~/services/postgres_data` (live),
`~/services/nodeutils` (live), and the pre-migration PostgreSQL dump under
`.local/backups/` — the designated recovery artifact.

## Left over

Nothing. The intent/evidence layer is all-green and phase 2 can proceed.
