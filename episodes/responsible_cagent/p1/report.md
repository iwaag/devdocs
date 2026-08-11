# Phase 1 report — Responsible cagent service foundation

Status: complete.

Phase 1 now has a reproducible service map, automatic local recovery, intent
placements for every keeper, authoritative DHCP/DNS for agautolab1, and fresh
observation evidence without operator-driven collection.

## What is now managed or supervised

- Dedicated always-on PostgreSQL and Redis for Nautobot, plus Nautobot web,
  worker, scheduler and MinIO under Compose restart policies and a tested cold
  start runbook.
- Gitea, Plane and agdevworld as persistent Docker services.
- agforge, agautolab gateway and cagent human/OpenCode entrances as launchd
  agents with tracked templates, health checks and project-local logs.
- Existing keeper workloads represented by intent: agautolab, node agents,
  Ollama, ComfyUI/SwarmUI, snake services, ACE-Step and music-gen.
- Twenty-seven service placements in the intent graph, including manual
  placements for the self-hosting boundary and externally supervised services.
- Six-hour launchd-driven nodeutils observation across every node with an
  active keeper placement. Collection failures are fail-closed and recorded.

agautolab1 now consistently leases `192.168.0.130` from agdnsmasq at
`192.168.0.2`; both `home.arpa` and mDNS resolve to it. The generated production
inventory and the Gitea-backed Ansible deploy path work at the new address.
The router at `192.168.0.1` remains the gateway and no longer serves DHCP.

## What was retired

Stopped and removed the unconsumed Keycloak, RabbitMQ/Hatchet and old database
stack, Adminer, RedisInsight, Excalidraw, legacy MinIO, both Portainer copies,
duplicate old Nautobot containers, and agautolab1's stray port-8080 HTTP server.
Volumes were deliberately retained. A pre-migration PostgreSQL dump is retained
under the ignored local backup directory.

## Final evidence

- Docker restart: all 21 running containers returned; keeper HTTP boundaries
  were healthy.
- LaunchAgent cycle: every owned native keeper and the observation scheduler
  returned and passed its endpoint/operation check.
- Tests: nodeutils 92 passed; nctl 1333 passed.
- Final intent state: 39 converged, zero drifting, one genuinely out-of-scope
  unknown (`agbach`); no `service_observation_stale` remains.

## Left for phase 2

- Decide whether to retire or place the catalog-only HAOS/Grafana/Nomad/
  Prometheus/node-exporter rows and whether agbach should rejoin observation.
- Generalize the agpc IPv6-RDNSS override if more dual-stack clients need the
  `home.arpa` routing policy.
- Remove preserved relic volumes only when their recovery value is no longer
  wanted.
- Use this now-stable intent and evidence layer for cagent explanation and
  responsibility work; cagent was intentionally not used to execute this phase.
