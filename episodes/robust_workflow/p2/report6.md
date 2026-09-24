# Step 6 report — rollout completed

Plan: [plan.md](plan.md) step 6. 2026-09-24 (JST), by the Omni Agent.

| Item | State |
|---|---|
| pyagag | `deec513` pushed; locked and synced in agfront, agautolab, agforge, agobserver, archsage, cagent and the agentroom relay; every venv's installed commit checked from `direct_url.json` |
| Running processes | all six listeners and the relay started on that revision (2026-09-23 23:01:52Z; Observer, Front and autolab restarted later in trials on the same revision); `/healthz` → `observer_monitor: ok` |
| agobserver | `0bf46af` (code `dc21380`; the introduction since); introduction re-posted in `#agents › intro-agobserver-agstudio1` (#9888) — the monitor section now describes requests followed by id until finished, rescue only on evidence, the owed note, and the relay watching the monitor |
| agdevworld | relay `e6643b4` (watchdog, `/ops` and `/healthz`); frontend rebuilt and the `web` container restarted — the ops board's header names the monitor's state ahead of the rows |
| launchd | Observer's plist sets `AGAG_STATUS_PATH` to the workspace's status file; the relay's carries `AGENTROOM_MONITOR_HEALTH`, `…_STATUS`, `…_ALERT=dm`; templates in `pj-agdev/devenv/launchd/` |
| nctl | after one observation refresh of agstudio (converged scope; only `observe_node` and the inventory render ran), Observer reads **`polling`** — the p1 `unobserved` is gone |
| Docs | `devdocs/README_DEV.md`: Observer's monitor (identity, retention, evidence, receipts, health and watchdog, fault hooks, withdraw), the trace's receipt rule and parent hop, the mention route by trigger id, id-based served marks and home anchors, the relay's one DM exception. The local environment memo has the host-side notes |
| Obsolete instructions | none found: the agents' guides only say how to read a `✔` topic, which still holds; no guide or help text described the p1 rules that changed (speech as a receipt, name-keyed marks) |
| Submodules | `pj-agdev` records agfront `243c9dd`, agautolab `a507617`, agforge `62d7409`, agdevworld `e6643b4`; every repository clean and pushed |

p1's name-keyed incident store is kept aside on the host (`incidents-p1/`); its incident topics in
Observer's channel remain the record.
