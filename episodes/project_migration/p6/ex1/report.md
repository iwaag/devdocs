# project_migration p6 ex1 — final report

Status: complete (2026-10-17). cagent can now develop missing backend
support instead of rejecting the Developer's chosen platform. It developed
macOS NFS server support, installed the `agshare` share on the selected Mac
through its agreed change workflow, and proved use from two hosts. The
paused `change/run-004` is unchanged.

Step reports: [report1](report1.md) (development path and authority),
[report2](report2.md) (macOS server support), [report3](report3.md)
(installation).

## Reviewed conversation

`cagent/change/run-004` (q1–q3, sequence 24) remains as recorded:
- The Developer proposed the always-on Mac with ample storage.
- cagent declined it as "Linux-only".
- The Developer asked why a macOS role could not be added, then paused.

The successor `change/run-005` names run-004 as its predecessor and did not
revive its Linux placement proposal.

## Authority and workflow changes

| Change | Where |
| --- | --- |
| WFE run copies own nested submodules (branch, checkpoint deepest first, integration records pins) | agdevworld `b40378f`, deployed |
| `develop` workflow; `backend/pj-clusterintent` submodule; `agent/develop.md` | pj-cagent |
| `cagentctl sources update` (plan `sources_update: true`), ledgered | pj-cagent |
| Guide: missing support is development work; development authority is not agreement to a host change | pj-cagent `agent/guide.md`, `agent/change.md` |
| Playbook allow-list: `setup_nfs_server_macos.yml` added | pj-cagent workspace `.local/cagent.toml` (ignored) |
| Operating notes | `devdocs/README_DEV.md` (cagent section); later-note on p6 report2 |

The effects ledger, source-bound plan blocks and the Developer's plan
agreement are unchanged. No approval per edit or command was added.

## Runs

| Run | Purpose | Outcome | Cost |
| --- | --- | --- | --- |
| `develop/run-001` | macOS server role, observation, tests | Accepted after one review round; integrated (pj-clusterintent `ff6343c`) | $8.46 |
| `develop/run-002` | macOS client automount | Accepted after one review round; integrated (`9ac3151`) | $5.16 |
| `develop/run-003` | fstab marker form for macOS `getfsent` | Accepted; integrated (`e4f856f`) | $2.60 |
| `change/run-005` | Installation and verification | Accepted; integrated (pj-cagent `9d4fdde`) | $19.11 |

The four runs cost $35.33 in total. Omni's own session is not counted.

## Revisions

| Repository | State |
| --- | --- |
| agdevworld | `b40378f`, deployed WFE release |
| pj-clusterintent | `e4f856f`. ansible_agdev `9c54208`, nodeutils `c86a852`, nctl `ad03af9`. The live deployment checkout is clean at this commit |
| pj-cagent | `9d4fdde` |
| pj-agdev | agdevworld pin |
| devdocs | This episode's reports and operating notes |

All of these were pushed, and each repository matches its remote.

## Evidence

The installed state, the cross-host integrity check (512 MiB over NFS, and
a file written on the Mac read over NFS), the remount, the local probe by
the executor identity, the idempotent second reconcile and the converged
drift are in report3. They were checked by Omni directly before
acceptance. Tests:
- nodeutils 104 passed;
- nctl 1357 passed;
- Ansible conformance 4 passed;
- ansible_agdev tests 7 passed;
- `nfs_client` macOS fstab tests 7 passed, against the real `getfsent`;
- cagentctl 44 passed;
- WFE 363 passed.

## Interventions

Omni enabled the work and reviewed it; it did not install anything on
cagent's behalf:
- WFE nested ownership and `cagentctl sources update`;
- the `develop` workflow;
- three requester reviews that sent results back, each with a concrete
  defect;
- the getfsent and automountd diagnoses;
- a test-fixture address fix;
- a playbook allow-list addition;
- answers to q11–q13 under the Developer's delegation for run-005, after the
  Developer approved the classifier-refused steps by hand.

## Remaining limitations

Follow-up (2026-10-17): [quick-fixes.md](quick-fixes.md) resolves item 3
and documents the correct operating path for items 2 and 5. The other
limitations below remain; no deferred host cleanup was performed.

1. The Mac still has the dropped client placement's fstab lines and its
   `/Volumes/agshare` symlink (deferred by the Developer).
2. nctl compares NFS placements by existence only. Configuration changes
   need the setup playbook.
3. **Resolved in the follow-up:** nctl now plans one action per service and
   deployment profile, preserving host scope and same-host dependency order.
4. cagentctl has no ledgered one-off privileged effect, so 1 and 2 needed a
   person or an allow-list edit.
5. A macOS server cannot be its own NFS client. Local processes use the
   backing path.
6. The large-file check ran from the Linux client to the Mac only.

## Return to p6

The working share is the prerequisite for p6 Step 4 (portable shared-input
references) and the playprey migration. Per-host resolution must map the
logical share to the Mac's local backing path there and to the Linux
client's mount path elsewhere. Another session's uncommitted Step 4 work
("shares") is in the agdevworld working tree; it was left untouched.
