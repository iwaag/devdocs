# observer p3 step 7 — deploy, verify and report (2026-10-14)

## Deployed

| Component | Version | How |
| --- | --- | --- |
| WFE (agdevworld) | `314c98b` (was `c100f56`) | `scripts/wfe-release.sh deploy`: built, 301 tests and build passed in the release, service and `bin/wfe` report the commit |
| executor (agautolab) | `1d2bb78` (guide `63e8fa4`) | no restart needed: wfexec calls the released `bin/wfe` and reads the `wfrun` guide at each claim |
| Observer | pj-agdev `c74d6d1`+ | `com.agdev.agobserver` (new), `com.agdev.agobserver-watchdog` (new); `com.agdev.agobserver-zulip` booted out and its plist archived |
| agentroom relay | agdevworld `314c98b` (its watchdog removed) | installed plist without `AGENTROOM_MONITOR_*`, bootout/bootstrap |
| recovery recipient | Gitea `developer/pj-agops` `e685223` | `wfe create` (Gitea), workflows `recover`, `check`, `freework`; Zulip `#wf-agops.recover` provisioned by the notifier |
| intake | area `zulip.json` | `intake.agents: Observer (23) → agops/recover` |

Before the release: nctl status ok; executor idle; one non-terminal run (jevlab `experiment/run-001`, the Developer's hold, untouched throughout). Stores and settings were backed up (`.backup`) first. No store migration was needed. The old Observer was stopped before its local state was archived (`retire_legacy`: 7 incidents and 16 watches, none open, nothing replayed), so no old monitor could emit a request after the cutover; there was no dual-monitor period.

## Verified after the release

- New Observer: first look in 1 s, source live on release `314c98b`, 1 run observed (`respected-hold`), `agag.status.v1` written for nodeutils (nctl status ok afterwards).
- Watchdog: `observer ok`, `executor available`; no alert during the trial.
- Relay `/healthz` ok, without the Observer watcher.
- `GET http://agstudio.local:8093/wfe/api/observe` answers from the LAN (200).

## Production recovery check

A disposable run of `agops/check` (a foreground `sleep 120`, then a note), requested by the Omni Agent for the Developer and interrupted on purpose:

| Time (JST) | Event |
| --- | --- |
| 13:12:08 | requested; attempt `amv7lfpwa-eb9265` began 13:12:11 |
| 13:12:37 | the deployed observation showed the open Bash call with its declared 150 s timeout → `working` (no false restart) |
| 13:12:39 | the attempt's process group killed (SIGKILL); held `interrupted` at 13:12:46 |
| 13:13:36 | Observer: incident `idb1de6a89b`, assignment 1; request posted in `#wf-agops.recover` |
| 13:13:43 | the executor's intake read it: `recover/run-001` |
| 13:14:25 | `run.recover` by `recovery-agent` (run `agops/recover/run-001`, attempt `amv7lhq1w-2365a1`) |
| 13:14:51–13:15:18 | continuation `amv7lj5k4-4e5d24` completed both nodes; `check.txt`: "continued by recovery after attempt amv7lfpwa-eb9265 was interrupted (SIGKILL)" |
| 13:15:37 | Observer: `restored` |

The recovery agent's reason: it checked the record, exec status, the copy (clean, checkpoint published), the definition (the sleep is resumable), the request (a disposable drill) and the session log (a bounded foreground sleep killed by SIGKILL, no other tool calls, no external effect). Cost $0.32 + $0.17. Detection 57 s after the hold (60 s looks), verified progress 3 min after the interruption, no human action after the request. Run view: `http://agstudio.local:8093/wfe/#/ws/agops/run/check/run-001` (the execution section lists the recovery); incident notes: `#agobserver-agstudio1 › incident-idb1de6a89b`.

## Documentation

- `devdocs/README_DEV.md`: the observer section rewritten for the workflow monitor.
- `agobserver/docs/workflow.md` (contract, running it, sessions, retirement), `agobserver/docs/agops/`, WFE `docs/observation.md`, `docs/runs.md`, `docs/intake.md`.
- Ignored notes: `pj-agdev/.local/devenv.md` (*observer p3 on this host*): versions, backups, paths, the retired job, the archived Zulip objects, the push path for agops.

## Clean-up

The step 6 real-path trial's Zulip folders (#33, #34) and channels (#287–#290) are archived. Private trial areas lived in the session scratchpad; their evidence is in `pj-agdev/.local/observer-p3/`.
