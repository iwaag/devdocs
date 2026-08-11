# Phase 1 service inventory

Surveyed 2026-08-12 from live `docker ps`, Compose labels/files, listening
processes, launchd/crontab, `nctl status`, `nctl relations`, nodeutils dumps, and
read-only Ansible checks. A *service* below is an independently operated logical
service; tightly coupled implementation containers are grouped as one stack.

| Node | Service | Definition / current start | Known consumer | Class | Plan |
|---|---|---|---|---|---|
| agstudio | PostgreSQL (`my_postgres_db`, :5432) | legacy `~/services/service_scripts/docker-compose.yml`; container restart `always` | Nautobot; local runtime test gate | dependency | move to `pj-clusterintent/devenv/nautobot-dependencies`, keep data, `manual` placement |
| agstudio | Redis (`service_scripts-redis-1`, :6379) | same legacy Compose; restart `no` | Nautobot cache/Celery | dependency | move beside Nautobot, `restart: unless-stopped`, `manual` placement |
| agstudio | Nautobot web (:8000) | `pj-clusterintent/devenv/nautobot/docker-compose.yml`; restart `always` | nintent, nctl, cagent and cluster workflows | in-system | keep, change to `unless-stopped`, `manual` placement |
| agstudio | Nautobot worker | same Nautobot Compose; restart `always` | Nautobot Jobs / nctl mutations | in-system | keep, change to `unless-stopped`, `manual` placement |
| agstudio | Nautobot scheduler | same Nautobot Compose; restart `always` | scheduled Nautobot work | in-system | keep, change to `unless-stopped`, `manual` placement |
| agstudio | Nautobot MinIO (:9100/:9101) | same Nautobot Compose; restart `always` | `nctl upload`; agforge S3 endpoint is `agstudio.local:9100` | dependency | keep, change to `unless-stopped`, `manual` placement |
| agstudio | Gitea (`autodev-gitea`, :3000/:2222) | `pj-agdev/agautolab/devenv/gitea/compose.yaml`; `unless-stopped` | source of agautolab1 and generated static apps | dependency | keep, `manual` placement |
| agstudio | Plane stack (:8290) | self-contained Plane CE Compose under ignored `.local/plane-selfhost`; every long-running component restart `always` | agautolab Plane reporting and agdevworld task UI | dependency | keep as one logical `plane` service, `manual` placement |
| agstudio | agdevworld web (:8090) | `pj-agdev/agdevworld/compose.yaml`; restart `no`; down after host restart | human world UI | in-system | add `unless-stopped`, start, `manual` placement |
| agstudio | agdevworld assistant (:8091) | same Compose; restart `no`; down after host restart | agdevworld browser, cagent/agforge/autolab/Plane passthroughs | in-system | add `unless-stopped`, start, `manual` placement |
| agstudio | agforge request service (:8092) | `pj-agdev/agforge/service/serve.sh`; formerly hand/nohup; down after restart | agdevworld and autolab asset workflows | in-system | launchd `RunAtLoad` + `KeepAlive`, `manual` placement |
| agstudio | agautolab gateway (:8791) | `pj-agdev/agautolab/agent/gateway.py`; formerly hand/nohup; down after restart | agdevworld monitor / local autolab requests | in-system | launchd `RunAtLoad` + `KeepAlive`; existing manual placement |
| agstudio | cagent OpenCode (:4097) | `pj-clusterintent/cagent/opencode/start.sh`; hand-started; down after restart | cagent API backend | in-system | launchd `RunAtLoad` + `KeepAlive`, `manual` placement |
| agstudio | cagent API (:8788/:8789) | `uv run --project cagent cagent-api`; hand-started; down after restart | nodes and human cluster entrance | in-system | launchd with dependency retry semantics, `manual` placement |
| agstudio | node-agent (:4096) | `~/Library/LaunchAgents/com.clusterintent.opencode.agent.plist`; `RunAtLoad` + `KeepAlive` | cluster workflows | in-system | keep existing `nctl_managed` placement |
| agstudio | Ollama (:11434) | Homebrew launch agent | node agents, agautolab | dependency | keep existing `nctl_managed` placement |
| agstudio | heartbeat cron | user crontab every 10 minutes | existing cluster heartbeat | in-system | keep existing placement; extend observation schedule in Step 5 |
| agstudio | snake-web (:8123) | intent-managed static web app; not listening after restart survey | experiment workload | in-system | keep existing `nctl_managed` placement; reconcile later |
| agstudio | Keycloak (:8180) | legacy service_scripts Compose; restart `always` | no source/config consumer found | relic | stop/remove container; retain volume/data |
| agstudio | RabbitMQ (:5673/:15673) | legacy service_scripts Compose; restart `no` | only the unused Hatchet stack | relic | stop/remove container; retain volume |
| agstudio | Hatchet engine/dashboard (:7077/:8080) + PostgreSQL (:5435) | legacy service_scripts Compose | no pj-agdev/pj-clusterintent consumer found | relic | stop/remove containers; retain data |
| agstudio | Adminer (:8181) | legacy service_scripts Compose | no consumer; operator convenience only | relic | stop/remove container |
| agstudio | RedisInsight (:5540) | legacy service_scripts Compose | no consumer | relic | stop/remove container |
| agstudio | Excalidraw (:8282) | legacy service_scripts Compose | no consumer | relic | stop/remove container |
| agstudio | legacy MinIO (`my_minio`, :9001/:9090) | legacy service_scripts Compose | no consumer; agforge and nctl use Nautobot MinIO :9100 | relic | stop/remove container; retain data |
| agstudio | Portainer (:9000/:9443) | legacy service_scripts Compose | no project consumer | relic | stop/remove container; retain volume |
| agstudio | duplicate legacy Nautobot app/worker/scheduler | legacy service_scripts Compose; currently only created/exited | superseded by project Nautobot Compose | relic | remove containers; retain volumes |
| agstudio | Docker/OrbStack engine | OrbStack GUI/login process; containers recovered after the 2026-08-12 login | all local Compose stacks | dependency | out-of-scope platform; verify automatic engine recovery |
| agautolab1 | agautolab gateway (:8791) | enabled `autolab-gateway.service` user unit | remote autolab and agdevworld | in-system | keep existing `nctl_managed` placement |
| agautolab1 | cagent-snake-e2e (:8124) | nctl-deployed Python static server | e2e experiment | in-system | keep existing `nctl_managed` placement |
| agautolab1 | stray Python HTTP server (:8080) | long-running `python3 -m http.server 8080`; no unit/consumer | none found | relic | stop process |
| agpc | node-agent | user systemd unit, stale observation | cluster workflows | in-system | keep existing `nctl_managed` placement |
| agpc | SwarmUI (:7801) + managed ComfyUI child (:7821) | StabilityMatrix/native process | agforge image path | dependency | keep existing manual placements |
| agpc | ACE-Step API (:8001) | native process | music-gen | dependency | keep, manual placement |
| agpc | music-gen (:8093) | native process | agforge music requests | dependency | keep, manual placement |
| agpc | Nomad / node exporter | enabled system services | no current pj-agdev/pj-clusterintent consumer proven | dependency | out-of-scope host infrastructure; existing catalog entries have no placements |
| agpc | Portainer (`portainer3`, :8000/:9000) | standalone Docker container | no project consumer | relic | stop/remove container; retain volume |
| aghub | node-agent | active user systemd unit | cluster workflows | in-system | keep existing `nctl_managed` placement |

`agdnsmasq` is represented by its existing desired placement and stale nodeutils
dump; it was not reachable during this survey, so Step 4 uses nctl's supported
reconcile/apply path rather than ad-hoc access. No new service on `aghub` was
found beyond the node-agent. The read-only `aghub` checks did not use sudo.

No source/config references were found for Keycloak, Hatchet, Adminer,
RedisInsight, Excalidraw, the legacy MinIO, or either Portainer. The only
Hatchet mention outside history is nodeutils' generic detector, which is not a
consumer.
