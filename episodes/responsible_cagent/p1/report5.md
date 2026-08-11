# Step 5 report — Observation freshness and restart verification

Status: complete.

## Automatic observation

- Added `com.clusterintent.observation-refresh`, a launchd job with `RunAtLoad`
  and a six-hour interval. It refreshes the five nodes carrying active service
  placements: agautolab1, agdnsmasq, aghub, agpc and agstudio.
- Each host gets one bounded, host-scoped `nctl reconcile
  --refresh-observation --max-rounds 1 --yes` run. Destruction is not enabled;
  a lock prevents overlap; JSON evidence is retained per host; every action and
  observation result is validated. The expected `max_rounds_reached` result is
  accepted only when all executed actions succeeded.
- launchd's minimal environment initially lacked Ansible. The installed and
  tracked plist now supplies the Homebrew path. A live RunAtLoad pass completed
  all five targets without manual collection.
- Fixed the Linux collection preflight so an already-installed Git does not
  require a successful APT refresh. Also made observation fail closed when the
  collection or report-retrieval command fails: a still-young old report can no
  longer be mistaken for the result of the requested collection.
- Added explicit process observation for manual native services and verified
  agforge, cagent API/OpenCode, ACE-Step and music-gen. Renamed the dependency
  Compose project/Redis container so the existing generic Docker observer sees
  `redis`, not an ambiguous `nautobot-*` substring.

All keeper-node dumps are now below the 24-hour threshold. The universal
`service_observation_stale` gap is gone.

## DNS reachability found by fresh observation

Fresh agpc evidence exposed one real problem: IPv6 router RDNSS took precedence
over `192.168.0.2`, so `agstudio.home.arpa` was sent to public DNS. The active
NetworkManager profile now ignores automatic IPv6 DNS and routes `~home.arpa`
to its DHCP-provided IPv4 DNS. `resolvectl query` returns `192.168.0.100` and
the Ollama binding probe returns HTTP 200. A following observation showed the
agpc node-agent active and its binding reachable.

## Restart test

- Stopped and started OrbStack. Its start command reported a startup wait
  timeout, but the Docker API became ready during the bounded retry.
- Running containers before/after: 21/21, with no missing or extra container.
  PostgreSQL, Redis, Nautobot web/worker/scheduler/MinIO, Gitea, Plane and
  agdevworld all returned automatically. External checks returned HTTP 200 for
  Nautobot, Gitea, Plane, agdevworld web/assistant, and agforge.
- Confirmed `unless-stopped` on the owned dependency/Nautobot containers;
  Plane's vendor stack retains `always` and also recovered.
- Booted out and bootstrapped agforge, agautolab gateway, cagent API/OpenCode,
  the node-agent and observation-refresh LaunchAgents. Two immediate first
  bootstrap attempts returned launchd's transient error 5; retry succeeded,
  matching their throttle/recovery posture. Final health: agforge `/healthz`
  200, gateway `/healthz` 200, cagent human HTTPS root 200, both OpenCode
  `/global/health` endpoints 200, and the scheduler's full pass exited 0.

## Final drift

Final summary:

```text
converged=39  unknown=1  drifting=0  converging=0
severity: error=1 warning=5 info=9
```

There are no keeper service gaps and no stale service observations. The one
unknown/error is `agbach: stale_actual_data`; it has no active placement and is
outside this phase. Five warnings are catalog-only services with no active
placement (`haos`, `grafana`, `nomad`, `prometheus`, and
`prometheus-node-exporter`), also outside this phase.
