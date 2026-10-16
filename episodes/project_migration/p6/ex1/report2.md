# project_migration p6 ex1 — Step 2 report: cagent develops macOS NFS server support

Status: complete (2026-10-16). cagent implemented, tested and published
macOS NFS server support through its new `develop` workflow (`develop/run-001`).
The result was reviewed and accepted by the Omni Agent as requester after
one round of requested fixes, and integrated by WFE into every backend
repository with the superproject pins. No host was changed. Raw evidence
(request, cagent's design and report, test logs, run history, integration
journal) is in the ignored `pj-agdev/.local/project-migration-p6/ex1/step2/`.

## The run

| Fact | Value |
| --- | --- |
| Run | `cagent/develop/run-001`, requested by the Omni Agent with the reviewed run-004 conversation as context |
| Nodes | inspect/design → implement → test → confirm (q1 not accepted, q2 accepted) → final report |
| Attempts | 3 (initial; resume after q1; resume after q2) |
| Elapsed | 40 min (13:17–13:57 UTC) |
| Cost | $8.46 (5.72 + 2.43 + 0.30), backend `claude_code` / sonnet profile |
| Copy | All seven repositories on the run branch: the root, `backend/pj-clusterintent` and its five submodules |

The run was created with the first `develop` definition, which put the
final report after the confirmation. An instruction (i1) told cagent to
write `report.md` before the confirm question and present it there. cagent
acknowledged it and did so; the definition is fixed for later runs (Step 1
amendment).

## What cagent developed

| Repository | Commit | Change |
| --- | --- | --- |
| ansible_agdev | `1f8ee9c`, `8502948` | `nfs_server` dispatches by OS: `linux.yml` (the former body) and a new `macos.yml`. On macOS: root check before any change; the backing directory; rejection of client specs `exports(5)` cannot express; one marked block per placement in `/etc/exports` with `-mapall=<uid>:<gid>` (the analogue of `all_squash`); pruning only of its own blocks of inactive placements, with every candidate validated by `nfsd -F <file> checkexports` before it is written; `nfsd enable`/`start`/`update`; verification by `pgrep -x nfsd` and `showmount -e localhost`. A `sandbox_check failed` validation error becomes an explicit Full Disk Access message. New `setup_nfs_server_macos.yml`; `nfs_server` gains a `macos` entry in `playbook_by_os`; README updated; new `tests/test_nfs_server_guard_order.py` |
| nodeutils | `5c8ed83` | `nfs_export_status` dispatches on the platform. macOS reports configured (exports file), running (`nfsd` process) and exported (`showmount`) separately, with bounded subprocesses. The Linux path is unchanged; `nfs_mount_status` was already OS-neutral |
| nctl | `ad03af9` | Profile test expects the new `playbook_by_os` entry |
| pj-clusterintent | `ff6343c` | Pins the three commits above |

Read-only facts cagent gathered on the hosts:
- become reaches root on the selected Mac and on the second PC
  (`ansible <host> -b -m command -a id`). This is the check my own session
  could not run (report1).
- The target runs macOS 26.2.
- `nfsd` is enabled but not running, and there is no `/etc/exports`.
- `man exports`/`man nfsd` confirmed the vocabulary the role uses.

## Review (Omni as requester)

The first presentation (q1) was not accepted, for three reasons:

1. **Linux regression.** Creating the backing directory had moved into the
   shared `main.yml`. On Linux it therefore ran before the
   unprivileged-container refusal, although the report called the Linux path
   unchanged.
2. **Validation order.** `/etc/exports` was rewritten before
   `nfsd checkexports`, although `nfsd -F <file> checkexports` can validate a
   candidate file first.
3. **A factual error.** The report said the man pages name no privacy
   permission. `man nfsd` documents `sandbox_check failed` and the Full Disk
   Access remedy.

cagent fixed all three in `8502948`:
- Each OS path keeps its guard before its first change, with a real
  `ansible-playbook` regression test that fails on the earlier commit.
- Both `blockinfile` tasks validate their candidate file with
  `nfsd -F %s checkexports`.
- The role explains a Full Disk Access failure, and the report is corrected.

I re-ran the tests in the run's copy myself before accepting:
`ansible_agdev/tests` 7 passed and nodeutils 104 passed.

**One correction was recorded in the acceptance (q2) rather than as another
round.** The design suggested a `/srv/...`-style backing directory. On macOS
the root volume is sealed and read-only (`touch /srv-test` → "Read-only file
system"), and `/srv` does not exist. The backing directory must be under a
firmlinked data-volume location (`/opt`, `/Users/Shared`, `/Library`, or a
disk under `/Volumes`), and that choice also decides the Full Disk Access
question. The role takes the path from the placement and would fail visibly
at the directory task, so the code is correct. The choice and its
verification belong to the installing change run (Step 3).

## Tests (as recorded by cagent, final revision)

| Check | Result |
| --- | --- |
| nodeutils ordinary | 104 passed (99 before + 5 macOS) |
| nctl ordinary | 1357 passed |
| Ansible conformance | 4 passed |
| Compute conformance | 1 passed |
| ansible_agdev/tests | 7 passed (2 new: guard order) |
| `ansible-playbook --syntax-check` | Clean on all four NFS playbooks |
| Local `--check` dry run on macOS | Dispatch selects the macOS path, and the privilege guard fails closed without become |

**Not exercised:**
- No mutating macOS step has run on a real host: writing `/etc/exports`,
  activating `nfsd`, a live `showmount` and a client mount.
- Whether `nfsd` needs Full Disk Access for the chosen path is unknown.

Both belong to the Step 3 change run. Unit tests and syntax checks are not
evidence that the macOS server works.

## Integration and publication

WFE integration 37 (decision 0) completed in one pass:

| Repository | Method | Result |
| --- | --- | --- |
| ansible_agdev | fast-forward | `8502948` |
| nctl | fast-forward | `ad03af9` |
| nodeutils | fast-forward | `5c8ed83` |
| nintent, nauto | already (unchanged) | — |
| pj-clusterintent | fast-forward | `ff6343c`, pinning the integrated commits |
| pj-cagent root | merge (main had moved) | `ae2d518` |

- **Remote check:** GitHub `pj-clusterintent` `main` is `ff6343c` and pins
  `ansible_agdev 8502948`, `nctl ad03af9` and `nodeutils 5c8ed83`.
- **Public-repository check:** the added lines contain no cluster host
  names, local paths or real addresses. A nodeutils test fixture uses two
  private addresses that are in neither the inventory nor the host notes.
- **Workspace:** the cagent workspace was fast-forwarded to `ae2d518` with
  its submodules at the new pins, so the next run copy starts from them.
- **Live deployment checkout:** still `898dcf6`. Moving it is the first
  agreed effect of the Step 3 change (`sources_update: true`).

This was the first real use of WFE `b40378f`'s nested ownership: nested
repositories were integrated first, and the superproject recorded their
integrated commits.

## Interventions and notes

- **Omni intervention, recorded:** I gave the i1 workflow correction. I
  did not implement anything for cagent.
- **cagent's question rounds were reviews, not approvals.** Its own
  acceptance decision cites my q2 answer.
- **Report1 guidance correction:** cagent ran `ansible … 2>&1 | tail`
  without the "non-blocking IO" failure my own tool shell had. The
  `develop.md` note on redirecting Ansible output to a file therefore
  describes my session's environment more than the agent's. It is harmless
  and stays until evidence says otherwise.
- **Earlier status update:** while waiting for the integration I told the
  user that a "30-minute stall" had happened. It had not: my wait loop
  exited early on a loose match, and the run was progressing normally.
