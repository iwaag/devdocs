# Rename agops to warden: final report

Date: 2026-11-02. Plan: [plan.md](plan.md). Step reports: [1](report1.md), [2](report2.md),
[3](report3.md), [4](report4.md), [5](report5.md).

## Outcome

The project is now `warden` in its identity, display, repository, workspace, documentation and recovery
routing. Recovery was exercised on the deployed system and ended `restored`. Existing runs, their
history and their artifacts remain readable. No new operations features were added.

## What changed

| Surface | Now |
| --- | --- |
| Project and workspace | `warden` |
| Repository | `developer/pj-warden`, same id 138, branches and history |
| Workspace folder | `<area>/pj-warden` |
| Recovery | `warden/recover`, channel `wf-warden.recover` |
| Zulip | `wf-warden` folder and channels, same numeric ids and history |
| New runs | `wfe/warden/...`, `wf-warden.<workflow>.<run>.io` |
| Unchanged by design | `recovery-agent`, workflow ids |

Older runs keep their UID and branch (`agops/...`, `wfe/agops/...`). Their current project moved through
a new recorded operation, `run.rename-project`. The registry's `formerly` resolves what existing records
stored, and nothing new is routed through it.

## Revisions

| What | Revision |
| --- | --- |
| WFE, deployed on the coordinator and pinned on `gpupc` | agdevworld `d7a3bd1` (previous `2951d02`) |
| WFE test-only follow-up, not a release | agdevworld `9975c43` |
| Observer, submodule pins | pj-agdev `71d525b`, then the commit recording this report's pin |
| warden | `51bfb98` (definition, README), `ab3a3a1` (tree-only record) |
| curator | `f475161` (catalogue entry) |
| Executor | unchanged |

## Migration counts

106 runs renamed (105 with copies, 1 tree-only; 1 retired). 105 manifests, 104 tree remotes, 114 Zulip
names, 67 session folders, 6 presentation configs, 7 incidents. Notification rows moved with their ids:
0 queued again, 0 old targets. Service interruption: about 2 minutes (17:22:15 to 17:24:28 JST).

## Checks that ran

- WFE release check: type check, 717/717 tests, builds. Observer `pytest`: 122/122.
- Rehearsal on isolated copies, including two `kill -9` interruptions that converged.
- Live: new run, full recovery path, new verified backup, retired-run restore rehearsed on two hosts.

Details are in reports 3 and 5. The rehearsal found five defects before the live change, including a 106th
run the plan's count of 105 had missed, and old channels being provisioned again.

## Limits and open items

- **Not verified:** a pre-rename held run continuing (none exists), a recovery run's own failure live, a
  new delegation relay or review page on an old run, `freework`, a real live reinstate. See report 5.
- **Old links do not resolve:** `#/ws/agops/...` browser links in earlier messages and notes, as the plan
  allowed. Numeric Zulip links keep working.
- **A live reinstate from an older generation** can bring back a pre-rename record if that record is
  missing; rerunning the migration converges it. Older generations age out at 7 kept.
- **Watchdog `degraded`:** 5 old recovery requests never taken up. It predates this work and is unchanged.
- **Owners received a false alarm:** my test load delayed the executor's heartbeat, and an `executor-down`
  DM went out at 16:33 (report 4).
- **GitHub:** agdevworld was pushed, because the worker pin fetches it. pj-agdev and devdocs are committed
  locally and not pushed.
- **The old Gitea name redirects** to the new one by default. Nothing uses it.
