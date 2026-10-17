# project_migration p6 — Step 3 report: the shared area, installed and repaired by cagent

Status: complete (2026-10-17). The `agshare` NFS shared area is installed,
is usable from two hosts, and is recorded in cagent and Nautobot. The
installation was performed by managed cagent through its `change` workflow.
A defect found after installation was repaired through a second cagent
change run. One part of the planned proof was skipped on the Developer's
instruction; see "Limits". Raw evidence is in the ignored
`pj-agdev/.local/project-migration-p6/step3/` and `.../ex1/step3/`.

## How Step 3 was carried out

| Part | Where | Report |
| --- | --- | --- |
| First plan (Linux hypervisor as server), declined by the Developer, who chose the always-on Mac; run paused | cagent `change/run-004` q1–q4 | [ex1](ex1/report.md) |
| Missing macOS support developed by cagent (`develop/run-001..003`) | pj-clusterintent `e4f856f` | [ex1 report2](ex1/report2.md) |
| Installation and two-host verification | cagent `change/run-005` | [ex1 report3](ex1/report3.md) |
| Cleanup of the dropped Mac client placement (run-004 continued on another model) | cagent `change/run-004` | [ex2](ex2/report.md) |
| Hang found and repaired | cagent `change/run-006` | this report |

The Developer chose the server, so the Step 2 premise "Debian-family Linux
server only" was overtaken. cagent developed macOS support instead of
refusing the platform.

## Installed state

| Fact | State |
| --- | --- |
| Server | The always-on Mac exports a local backing directory to the Linux client only, with every client mapped to the share owner (the Mac's executor identity) |
| Mac's own access | Local backing path (a macOS server cannot be its own NFS client: automountd substitutes a symlink). Projects resolve the logical share per host (Step 4) |
| Client | The always-on Linux VM, through a systemd automount with `noresvport` (since run-006) |
| Server setting | `nfs.server.mount.require_resv_port = 0` in the Mac's `/etc/nfs.conf` (since run-006) |
| Records | cagent `records/changes/change.run-005.1.md` and `change.run-006.1.md`. Nautobot desired services `agshare-server` and `agshare-client` with their placements (`kind: maintain`) |
| Observation | Fresh drift (2026-10-17 ~03:20 UTC): `agshare-server` and `agshare-client` converged. The only drifting target is the unrelated pre-existing Observer `service_missing` |

## Verification

| Check | Run | Result |
| --- | --- | --- |
| Executor identity in its service context on the Mac | run-005 | Wrote, read, renamed and hashed a probe in the local backing path without sudo |
| Large file across hosts | run-005 | 512 MiB written over NFS from the client hashed identically on the Mac. A file written on the Mac hashed identically over NFS on the client |
| Automatic remount | run-005 | Unmounted on the client; the automount brought the share back |
| Idempotence | run-005 | A second reconcile of both hosts made no changes |
| After the repair | run-006 | Baseline list/write/rename/read-back, hash and cleanup on the client succeeded. The settings were checked independently by Omni (see below) |
| Omni's independent read after run-006 | — | The client lists the share. The live mount has `noresvport` and its TCP connection uses an unprivileged source port. Drift converged. The share is empty (probes removed) |

## The hang and its repair (change/run-006)

After run-005 was accepted, the client's mount stopped answering.
- **Symptom:** the 01:02 UTC automatic observation reported "mounted, listing
  timed out"; `ls` on the client hung.
- **Diagnosis:** the Mac's nfsd drops connections idle for 3600 s
  (`nfsd_sock_idle_timeout`) but left them in `CLOSE_WAIT`. The Linux client
  sat in `FIN-WAIT-2` on reserved source port 850 and could not reconnect, so
  every access blocked on the hard mount.

| # | Event | Resolution | Who |
| --- | --- | --- | --- |
| 1 | Diagnosis above | Repair request filed as cagent `change/run-006` (predecessor run-005) | Omni (read-only diagnosis, request) |
| 2 | Plan: declare `noresvport` on the client placement, reconcile the client only, test after a 3,660 s idle interval | Agreed (q1) | cagent; Omni under the Developer's delegation |
| 3 | The Developer found the idle test too long | Skipped (instruction i1; plan revised, q2) | Developer |
| 4 | The client setup playbook blocked in uninterruptible sleep at "ensure mount point exists" on the hung mount, before changing the declaration | Person step `umount -f -l` on the client (q3–q4); the blocked task ended | Omni as the delegated person |
| 5 | The new declaration was deployed, but every mount was "access denied by server": with `noresvport` the MOUNT request also uses an unprivileged port, and macOS mountd requires a reserved port for MOUNT by default (`nfs.server.mount.require_resv_port`, default 1; separate from `nfs.server.require_resv_port`, default 0) | Person step on the Mac: append `nfs.server.mount.require_resv_port = 0` to `/etc/nfs.conf`, restart nfsd (q5–q7) | Plan by cagent; executed by Omni as the delegated person |
| 6 | Mount, baseline I/O, cleanup, fresh drift | Accepted (q8) and integrated | cagent; acceptance by Omni under delegation |

Every effect has a ledger entry (`e0001`–`e0009`). The interrupted reconcile
(e0003) was settled from its operation record (`partial`/`non_converged`),
never repeated blindly.

## Who decided what

- The Developer chose the Mac as server (run-004) and skipped the idle test.
- On 2026-10-17 the Developer delegated the run's answers to the Omni Agent
  ("私の代わりに進めて") and approved by hand the steps the session
  classifier refused: adding the Omni Agent to run-006's run-copy approvers
  (removed again after integration; the workspace configuration never listed
  it) and relaxing the MOUNT reserved-port requirement.
- The two person steps were run by the Omni Agent through Ansible with
  privilege escalation, recorded in cagent's ledger as person effects, and
  confirmed in the Omni Agent's own name.
- Deus ex machina notes:
  - Omni diagnosed both the hang and the access denial for cagent (handoff
    candidate: cagent's own diagnosis from the client's journal and socket
    state).
  - Omni executed both privileged person steps (handoff candidate: a ledgered
    privileged effect in `cagentctl`).

## Limits

1. **Recovery after an idle drop was not re-tested.** The 3,660 s test was
   skipped by the Developer. `noresvport` is deployed and the connection now
   uses an unprivileged port, but no run has shown the mount surviving the
   Mac's idle drop. If it hangs again, the Step 1 observation will report the
   mount `unusable`.
2. **The Mac's `nfs.conf` setting is not declared.** It was a person step.
   The macOS `nfs_server` role should declare it whenever clients use
   `noresvport` (develop follow-up).
3. **The `nfs_client` role cannot repair a hung mount.** Its mount-point task
   stats the path first and blocks (my Step 2 defect). It should skip or
   bound that check when the path is an NFS mount (develop follow-up).
4. **run-006's ledger kept no hash value** for its baseline probe; run-005's
   large-file hashes remain the integrity evidence.
5. From ex1, still open: nctl compares NFS placements by existence only, and
   `cagentctl` has no ledgered ad-hoc privileged effect.

## Acceptance check

- **Usable from two hosts:** yes. The client is over NFS. On the Mac, the
  executor identity uses the local backing path, which is the only form a
  macOS server can use for itself. Both were verified, with content hashes in
  run-005 and live use after run-006's repair.
- **Installed through cagent:** yes. Server and client were reconciled under
  agreed plans with an effects ledger. The two privileged person steps were
  ledgered handoffs.
- **Operational records:** cagent's change records name what was configured
  and tested, Nautobot holds the desired services and placements, and fresh
  drift is converged. The idle-drop gap is stated above.
