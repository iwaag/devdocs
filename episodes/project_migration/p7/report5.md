# p7 Step 5 — deploy and register the PC worker

Physical facts, raw logs and the rollback commands are in pj-agdev's ignored
`.local/project-migration-p7/notes.md`.

## Deployed revisions

| Component | Revision |
| --- | --- |
| WFE (coordinator release, and the worker's checkout) | agdevworld `14e2b42` |
| Executor, existing worker | agautolab working tree at `0a813b0` (relaunched on `84ae825` first) |
| Executor, PC worker | `0a813b0` (installed at `84ae825`, then moved to `0a813b0`) |
| Observer | pj-agdev `c15bb7b` (source; the Observer process was not restarted in this step) |
| Deployment support | ansible_agdev `2ae175d`, nctl gate, pj-clusterintent `0995e58` |
| Pins | pj-agdev `f7d46a0` |

## Migration

1. Took a backup of the exec, notify, intake and present stores and the area settings before
   deploying.
2. Wrote `workers.json` (default and local `studio`, remote `gpupc`). Then activated the release
   (release check 403/403) and restarted the existing executor as `worker = "studio"`.
3. The exec store migrated to schema v3 on first open. 301 historical attempts are now `studio`'s,
   and the old heartbeat row became `studio`'s.
4. `wfe runs assign-nodes` recorded `run.assign` for the one unfinished run (jevlab
   `experiment/run-001`, held stopped). It was not resumed.
   - Correction to report2: playprey `onboard/run-002` was already cancelled and needed nothing.
5. Rollback is coherent but not one-step. Activate the previous release *and* restore the exec
   store from the backup, because the old code does not know schema v3. Then drop `worker` from
   the executor settings and disable the PC unit.

## Provisioning through cagent

The PC was provisioned by cagent `change/run-008` (luna). It used desired state and nctl, with no
hand edits on the hosts:
- `agshare-server-agstudio` gained the PC as a client;
- new placement `agshare-client-agpc` (`nfs_client`);
- new placement `agautolab-agpc` of service `agautolab` with the new deployment profile
  `wfe_worker`.

Its first reconcile found a defect in the new role: the executor checkout's `.local/` was not
created before its overlay was written. The install stopped half-way, yet nctl reported
converged, because the profile's existence check was `worker.json`, an early artifact.

- Fixed the role. The check is now the systemd unit, the role's last artifact.
- Resumed the run with the facts. It reconciled again, proved the share, verified, presented, and
  Omni accepted its result. Its integration is queued.
- One gap surfaced and was worked around:
  - The run's first attempt ended on an *external* wait (an nctl operation).
  - WFE has nothing that resumes an external wait when its result arrives; a person's
    `wfe run resume` was needed.
  - This is pre-existing and recorded as a limitation (report.md).

The deployment support is reusable. The profile `wfe_worker` takes the worker id, coordinator,
pinned WFE and executor commits, controller-local token files, shares, display and Godot version.
Nothing project- or machine-specific is in the code.

The role installs:
- pinned checkouts and their locked dependencies;
- Claude Code (native, pinned) and Godot (pinned);
- the worker area: `worker.json`, tokens, Gitea setting, `shares.json`, launcher, and executor
  settings with intake/notify off;
- a lingering systemd user unit with `KillMode=process`, so an executor restart leaves agents
  running for adoption.

The PC has its own Gitea token (`write:repository`), not a copy of the coordinator's.

Updating a pin is also proven. A config change of the placement (executor `84ae825` → `0a813b0`)
is not detected by nctl, which compares this profile by existence like NFS. It was applied with
the setup playbook, the same documented route as NFS config changes.

## Placement and observation

- `nctl drift --host agpc` is converged with `agautolab-agpc` and `agshare-client-agpc` applied
  (plus the existing ComfyUI and SwarmUI).
- nctl proves installation (the unit file). Liveness is the worker's own heartbeat at the
  coordinator: `GET /api/workers` reports `gpupc` `available`, with its offer and host facts
  (OS, Godot path, display, shares).
- The legacy gateway and deployment role were not revived.

## Readiness checked on the PC

| Check | Result |
| --- | --- |
| `wfe-worker.service` | active |
| Godot | 4.7.2 |
| Claude Code | 2.1.284, logged in (a non-interactive prompt answered) |
| `/mnt/agshare` | lists the project data; the worker's own `wfe share list` says `agshare ok` |
| cagent's share probe | list/write/rename/hash passed, then removed |
| Coordinator from the PC | WFE on `:8093`/`:8098` and Gitea answer |

**Offer.** The worker offers every profile in `agents.toml`, but only `sonnet` (claude_code) is
set up on the PC. Codex there is the old 0.114 snap, so `luna` would fail at launch. Narrowing the
offer to what the host can run is a follow-up.

**Resources.** At deployment the GPU was idle (ComfyUI holding about 260 MiB) and the load average
was 0.3. No resource restriction was added; competition is measured during the trial (Step 6).

Fetching every repository, staging shared inputs, invoking the harness, running Godot and
publishing a result are exercised by the real trial in Step 6.
