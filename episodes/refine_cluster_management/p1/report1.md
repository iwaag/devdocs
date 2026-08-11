# Step 1 report — Inventory and classification

Status: complete.

The live inventory and decisions are in [inventory.md](inventory.md). The
survey covered:

- all 32 local containers and their Compose ownership/restart policy;
- local listeners, native processes, crontab and user LaunchAgents;
- current Nautobot status/relations and the seven nodeutils dumps;
- read-only Ansible inspection of agautolab1, agpc and aghub;
- project-wide consumer searches excluding history, tests and lockfiles.

Important findings beyond the plan baseline:

- The machine had restarted shortly before the survey. OrbStack and eligible
  containers recovered, while all three hand-started service groups
  (agforge, local agautolab, cagent) and agdevworld remained down. This is
  direct evidence for Step 2 rather than a hypothetical failure mode.
- agpc also hosts live ACE-Step (:8001) and music-gen (:8093), used by the
  agforge music path; they need manual intent placements.
- agautolab1 has an unowned Python HTTP server on :8080 with no discovered
  consumer. It is classified as a relic.
- The Plane stack is already self-contained and restart-enabled; it is treated
  as one externally consumed logical service rather than exposing every Plane
  implementation container in cluster intent.
- The legacy service_scripts stack contains two keepers (Nautobot PostgreSQL
  and Redis) and otherwise only superseded or unconsumed services.

Baseline remained healthy: `nctl status --json` reported Nautobot reachable,
one worker, zero old pending jobs, and clean submodules. All 12 existing
placements remained stale against the 24-hour observation threshold.

Did the cross-node service inventory for cagent — handoff candidate.
