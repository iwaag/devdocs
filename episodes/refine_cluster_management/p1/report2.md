# Step 2 report — Always-on setup

Status: complete.

## Docker services

- Moved the live Nautobot PostgreSQL and Redis out of the legacy
  `service_scripts` Compose into the dedicated tracked
  `devenv/nautobot-dependencies/compose.yaml`.
- Preserved PostgreSQL in place through an ignored
  `NAUTOBOT_POSTGRES_DATA_DIR`; took a 3.2 MiB pre-switch custom-format dump at
  `.local/backups/responsible-cagent-step2.dump`.
- Kept the public host ports 5432/6379, so Nautobot's existing
  `host.docker.internal` settings did not change. Redis now has AOF persistence
  in its own named volume.
- Applied `restart: unless-stopped` to PostgreSQL, Redis, Nautobot web/worker/
  scheduler/MinIO, and agdevworld web/assistant. Gitea already used it. Plane's
  vendor-managed stack retains its equivalent `restart: always` policy.
- Rebuilt and started agdevworld. Web :8090 and assistant :8091 both returned
  HTTP 200.

The desktop container engine is OrbStack on this host rather than Docker
Desktop. The survey occurred immediately after a login/restart and OrbStack
had started automatically and restored eligible containers, which is the
required platform behavior.

## Native services

Installed and bootstrapped four user LaunchAgents with `RunAtLoad`,
`KeepAlive`, and a ten-second throttle:

| Label | Endpoint | Result |
|---|---|---|
| `com.agdev.agforge` | :8092 `/healthz` | HTTP 200 |
| `com.agdev.agautolab-gateway` | :8791 `/healthz` | HTTP 200 |
| `com.clusterintent.cagent-opencode` | :4097 `/global/health` | HTTP 200 |
| `com.clusterintent.cagent-api` | :8789 `/llms.txt` | HTTPS 200 |

Tracked plist templates live in each owning project's `devenv/launchd/`; the
installed plists contain local absolute paths and remain outside Git. Secrets
remain in their previous ignored/default files. The first agforge launch
exposed launchd's minimal `PATH`; adding the explicit Homebrew path fixed it,
and that setting is captured in the template.

## Retired relics

Stopped and removed the following containers without deleting any volume or
bind-mounted data: Keycloak, the old RabbitMQ/Hatchet/dashboard/PostgreSQL
stack, Adminer, RedisInsight, Excalidraw, legacy MinIO, agstudio Portainer, and
the duplicate legacy Nautobot app/worker/scheduler and init containers.

Read-only inventory had resolved two remote relics precisely; the authorized
cleanup then removed agpc's standalone `portainer3` container and stopped
agautolab1's unowned `python3 -m http.server 8080`. Both absences were verified.

Post-switch checks passed for PostgreSQL, Redis, Nautobot web/worker/scheduler,
Nautobot MinIO, Gitea, Plane, agdevworld, and all four native jobs.

Commits:

- pj-clusterintent `16388fd` — dependency Compose, Nautobot restart policy,
  cagent launchd templates.
- agdevworld `d906819` — agdevworld restart policy.
- pj-agdev `475c072` — submodule update and agdev launchd templates.

Removed old runtime scaffolding for cagent and agdev agents — handoff
candidate.
