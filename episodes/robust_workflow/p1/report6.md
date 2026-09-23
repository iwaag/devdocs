# Step 6 report — rollout verified, documentation aligned

Plan: [plan.md](plan.md) step 6. The phase report is [report.md](report.md); this records the
rollout checks behind it.

## Versions actually running (2026-09-23 UTC)

| Consumer | pyagag in its venv | Listener restarted after the install |
|---|---|---|
| agfront | `87ac87e` | 18:48:54Z |
| agautolab | `87ac87e` | 18:40:35Z |
| agobserver | `87ac87e` (+ agobserver `879fb49` in pj-agdev) | 18:49:43Z |
| agforge | `87ac87e` | 18:40:35Z |
| archsage | `87ac87e` | 18:40:35Z |
| cagent | `87ac87e` | 18:40:35Z |
| agentroom relay | `7beeec5` (unchanged: not a listener, uses none of the changed code) | — |
| agautolab1 VM | not redeployed (gateway only, no listener) | — |

Read from each venv's installed `direct_url.json` and the process start time, not from the lock
files. Startup recovery after the last restarts queued 0 conversations in agfront, agautolab,
archsage and cagent; agobserver (2) and agforge (1) re-read mentions they deliberately ignore
(no run, no post). `nctl drift`: 46 converged, unchanged; liveness info as before
(`agobserver-agstudio1: unobserved` — see the phase report's remaining limits).

## Commits

| Repository | Final |
|---|---|
| pyagag | `87ac87e` |
| agautolab | `cab3353` |
| agfront | `dce4199` |
| agforge | `b1e5ba4` |
| archsage | `8ffd04f` |
| pj-agdev (agobserver, submodule pointers) | `879fb49` |
| pj-clusterintent (cagent lock) | `1c0050d` |
| devdocs | this commit |

## Introductions and documentation

- Re-posted: autolab (step 3, the progression contract), Observer (step 4, the monitor). forge,
  archsage, cagent and Front publish nothing that changed.
- `devdocs/README_DEV.md`: the autolab section's "a post is what starts a task" replaced by the
  new ownership; Observer's monitor; a section on `agentchat trace`, opfail notes, request-aware
  resolve and `unresolve`, and callbacks in ✔'d conversations.
- Guides: Front's per-task start paragraph and the retired `agentchat wait` removed (step 3);
  autolab's supercoder guide carries `hold.flag`, the already-started rule and the push-state
  rule (steps 3, 4); no other guide asked for per-task starts.
- The ignored local environment memo records the fault hook, the incident store, the monitor's
  switches and the pins.
