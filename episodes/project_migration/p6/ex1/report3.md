# project_migration p6 ex1 — Step 3 report: cagent installs the share on the selected Mac

Status: complete (2026-10-17). cagent installed and verified the `agshare`
NFS shared area through its `change` workflow, with the Developer's selected
Mac as the server. The run is `change/run-005`, a successor of the paused
`change/run-004`.

Three live defects in the new macOS support were found during execution.
Two were fixed through further cagent `develop` runs; the third was a macOS
behaviour that changed the design. Two follow-ups remain open. Raw evidence
is in the ignored `pj-agdev/.local/project-migration-p6/ex1/step3/`: the
plan, the change record, run histories, the getfsent test and the requests.

## Result

| Fact | State |
| --- | --- |
| Server | The selected Mac exports its local backing directory under `/opt` to the second PC only, mapping every client to the share owner (`-mapall`) |
| Share owner | The Mac's executor identity, so local writes on the Mac and NFS writes from clients have one owner |
| Mac's own access | Local path, no mount (see "macOS serves itself through a symlink" below). The executor's own launchd identity wrote, read, renamed and hashed a probe there without sudo |
| Second PC | Linux NFS client with systemd automount |
| Integrity | A 512 MiB file written over NFS from the second PC hashed identically on the Mac. A file written locally on the Mac hashed identically over NFS on the second PC |
| Automatic remount | Unmount on the second PC; the automount brought the share back |
| Idempotence | A second reconcile of both hosts reported no changes |
| Observation | Fresh observe and drift: both hosts `converged`, with the export and mount checks satisfied |
| Probes | Removed; the share is kept (`kind: maintain`) |
| Deployment sources | The live checkout was moved through the ledger: `898dcf6` → `ff6343c` → `9ac3151` → `e4f856f`, each by an agreed `sources update` |
| Records | `records/changes/change.run-005.1.md`. The run was accepted and integrated into pj-cagent `9d4fdde`. Nautobot holds the desired services `agshare-server` and `agshare-client` with their placements |

I re-read the Mac directly before accepting: the export block, the owner
and the executor's write permission, `nfsd` running, `showmount`, and the
empty share. The large-file check ran in one direction only: the Mac wrote
a small file and the second PC a 512 MiB one.

## What happened, in order

| # | Discovery | Resolution | Who |
| --- | --- | --- | --- |
| 1 | Omni's requester review of the first plan: no cross-host hash, no service-context test, remount on one client only, no rerun check, and a `.local` server name that may resolve to IPv6 | Plan revised (instruction i1); Developer agreed (q2) | Omni review, Developer agreement |
| 2 | nctl refuses a service whose placements mix the `nfs_server` and `nfs_client` profiles (planner `manual_review`) | Batch 2 split it into `agshare-server` and `agshare-client` (q3) | cagent, Developer agreement |
| 3 | The macOS client appended a second `/-` line to `/etc/auto_master`; macOS reads only the first | `develop/run-002`. Its first fix deleted the stock `/- -static` line; Omni sent it back. The final fix declares mounts in `/etc/fstab` for the stock `-static` map and tidies the role's own old lines | cagent; Omni review |
| 4 | macOS `getfsent(3)` silently drops an fstab entry with a trailing `# …` comment | Omni proved it read-only with an interposed `fopen` harness. `develop/run-003` moved the marker onto its own line, and its committed tests use the real macOS parser | Omni diagnosis, cagent fix |
| 5 | On the Mac, the automount of its own export became a root-owned symlink to the local path instead of an NFS mount (automountd's `self_check`). Through it the executor could not write the share owned by 1000:1000 | Plan change (q8–q9): drop the Mac's client placement, use the local path, make the owner the executor identity, and narrow the export to the second PC | Omni diagnosis, Developer decision |
| 6 | nctl compares NFS placements by existence only, so batch 3's owner and client change produced no reconcile action | `setup_nfs_server_macos.yml` added to cagent's playbook allow-list by Omni and run through the ledgered `cagentctl playbook` (q11–q12) | Omni allow-list; cagent execution |

Each discovery stopped the run before an unknown effect. No effect was
repeated to find out what happened, and nothing was destroyed.

## Development runs (backend changes after Step 2)

| Run | Change | Cost |
| --- | --- | --- |
| `develop/run-002` | ansible_agdev `1541eda`: macOS client through fstab and the stock `-static` map; superproject `9ac3151` | $5.16 |
| `develop/run-003` | ansible_agdev `9c54208`: marker on its own line, real-`getfsent` tests; superproject `e4f856f` | $2.60 |
| `change/run-005` | Installation (15 attempts) | $19.11 |

Omni also committed nodeutils `c86a852`, replacing the two real host
addresses in a Step 2 test fixture with documentation addresses
(superproject `21bb2a5`, merged into `9ac3151`). The addresses remain in
Git history; older fixtures already contain private-range addresses.

## Who decided what

- **Plan agreements and design decisions (q2, q3, q5, q7, q9, q10):** the
  Developer.
- **q4:** the Developer paused the run.
- **q6:** answered by the Developer with text Omni prepared.
- **q11–q13:** the Developer delegated the rest of the run to the Omni Agent.
  - The session classifier first refused both the approver change and the
    proxy answer. The Developer then switched to manual approval and
    approved them.
  - The Omni Agent was added to this run copy's `approvers` only, answered
    q11–q13 under its own name citing the delegation, and was removed from
    the list after integration. The workspace configuration never listed it.
- **Wording correction:** q5's text says "the Developer's accepted fix".
  `develop/run-002` was accepted by the Omni Agent as requester, not by the
  Developer.
- **Omni interventions, beyond requester reviews:**
  - the playbook allow-list addition;
  - the getfsent diagnosis;
  - the automountd symlink diagnosis;
  - the fixture fix.

  None of these installed anything on cagent's behalf.

## Open issues

1. **Leftovers on the Mac.** The marked fstab lines and the
   `/Volumes/agshare` symlink from the dropped client placement remain
   (deferred by the Developer, q10). They are harmless: the symlink points
   to the share itself. The role's own cleanup cannot reach the Mac once it
   leaves the `nfs_clients` group, and `cagentctl` has no ledgered command
   for a one-off privileged action.
2. **nctl does not detect configuration changes for NFS placements.** It
   compares them by existence only; only dnsmasq has a content comparator.
   A changed export option, owner or client list is never reconciled
   automatically.
3. **nctl's one-profile-per-service rule (planner)** is an implementation
   limit, not a safety rule. Dependency ordering is per profile and host.
   Planning one action per service and profile would remove it.
4. **`cagentctl` lacks a ledgered ad-hoc effect.** Issues 1 and 6 above both
   needed a person or an allow-list edit because the only direct operations
   are allow-listed playbooks.
5. **A macOS server cannot be its own NFS client.** automountd substitutes a
   symlink. Projects resolve the share per host (p6 Step 4's `shares.json`),
   so the Mac's local path is the expected form. Profiles and documentation
   should say so instead of offering a self-mount.
