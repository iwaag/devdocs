# Step 1 — identify and stop the old entrances

## Found at execution time

- On agstudio, launchd still ran the old Zulip listener (`com.agdev.agautolab-zulip`, `python -m agautolab.listener`, up since 2026-09-29) and the gateway (`com.agdev.agautolab-gateway`, `agent/gateway.py`, :8791). They ran alongside the retained `com.agdev.agautolab-wfexec`.
- On agautolab1 (the VM placement), a hand-installed `autolab-gateway.service` was running `agent/gateway.py` from a `b38a6af` checkout. No Ansible role installs it any more. Its old `autolab-zulip.service` and `autolab@.service` units were present but disabled.
- Nautobot (`nctl relations`, `nctl desired export`) had service `agautolab` with two placements:
  - `agautolab-agautolab1` (nctl_managed, profile `autolab_node`, endpoint :8791, listener off);
  - `agautolab-agstudio` (manual, profile `autolab_node`, endpoint :8791).
  Each placement had a workspace and a DesiredAgent (`autolab-agautolab1` = bot 12, `autolab-agstudio` = bot 11).
- The `autolab_node` profile's health check is HTTP `/healthz` on the gateway endpoint. That check is the gateway's only remaining deployment dependency. No component calls `POST /window` (subagent surveys of agfront, agobserver, comfynotify, agdevworld, archsage and pyagag).

## Unfinished work

- No old worker process was running. The listener had no children, and its last execution record is from 2026-09-28.
- Every `.local/missions/<slug>/` directory is empty: no mission worktrees remain.
- Every `autolab/m*` branch, plus `autolab/set-aside-20260924`, in the project clone sets is contained in another branch or remote, with 0 unique commits. Nothing was replayed as a workflow run and nothing was discarded.
- Locations are recorded in the ignored local notes, under agautolab's `.local/`.

## Done

- The listener and the gateway were booted out of launchd on agstudio and their installed plists removed. Copies of the plists are kept in ignored local notes. `com.agdev.agautolab-wfexec` is untouched (the executor was idle and available).
- On agautolab1, the gateway unit was stopped and disabled, and all three legacy unit files were moved out of the systemd user directory. `:8791/healthz` no longer answers. The checkout, which holds the bot-12 credential and old records, was kept.
- On the board, `#agents › intro-autolab-agstudio1` was resolved (✔), which is the board's retirement mechanism. No introduction is needed for the executor: it posts only `run-…` topics in `wf-…` channels.
- A desired-state batch was prepared in two passes and previewed clean (0 conflicts):
  - delete DesiredAgent `autolab-agautolab1`, placement `agautolab-agautolab1`, workspace `agautolab-agautolab1` and endpoint `agautolab1/agautolab`;
  - convert `agautolab-agstudio` to `manual_toolchain` with `process_pattern: agautolab\.wfexec` and no endpoint, so the retained placement is observed by the executor process and not by the gateway;
  - pass 2 deletes the `agstudio/agautolab` endpoint.
  
  Service `agautolab`, placement `agautolab-agstudio` and DesiredAgent `autolab-agstudio` (bot 11, used by wfnotify/wfintake) stay active.

## Not done

- **The desired-state commit is not applied.** The auto-mode classifier refused `nctl desired apply --yes`, so it waits for the user's approval. The batch files are in pj-clusterintent's ignored `.local/`.
- The `autolab-agstudio1` instance channel and the old `pj-<slug>` / `work-` channels are left as history. Nothing listens there now.
