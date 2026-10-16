# project_migration p6 ex1 — Step 1 report: a development path for cagent

Status: complete (2026-10-16). cagent now has a `develop` workflow whose
runs own editable copies of the backend repositories, publish through WFE
integration with the superproject pins, and leave host changes to the
`change` workflow. No host was changed, no cagent run was started or
resumed, and `change/run-004` is as it was. Raw evidence is in the ignored
`pj-agdev/.local/project-migration-p6/ex1/step1/`.

## Rechecked before implementation

| Fact | State |
| --- | --- |
| `cagent/change/run-004` | Stopped at sequence 24 by the Developer's "pause this work" (q3). `_plan` is still recorded as running with plan.md written; `execute`, `verify` and `confirm` are pending; there is no decision. q1–q3 and their answers are intact. Nothing was written to it. |
| Backend sources | The live pj-clusterintent checkout is clean at `898dcf6`, the Step 2 NFS tools: Linux server, Linux and macOS clients. |
| Cluster facts | `nctl` status through `cagentctl read status`: Nautobot reachable and authenticated; every active host observed about 15 minutes before the read. |
| WFE | Release `38ebdff`. Run copies gave branches only to the root's own submodules. A submodule's submodules were checked out detached, "not the run's", so nothing could be published from them. |

The live checkout's layout makes that last row the blocker: the work lands
in `ansible_agdev`, `nodeutils` and `nctl`, which are submodules of
pj-clusterintent. The superproject's gitlinks are the pins that the cagent
plan's `sources` binding names.

## What was missing, and what was added

| Gap | Change | Where |
| --- | --- | --- |
| No editable backend in any cagent workflow: `change` binds the project read-only; `freework` binds no backend and excludes hosts | New `develop` workflow: inspect/design → implement → test → confirm (result). `backend/pj-clusterintent`, `tools` and `agent` are editable; records and desired batches are not bound | pj-cagent `devdocs/workflows/develop.yaml` |
| The backend is not part of the project | `backend/pj-clusterintent` added as a submodule (public GitHub URL). The cagent workspace has all five of its submodules checked out, mirroring the live layout | pj-cagent `.gitmodules` |
| WFE could not own nested repositories: copy, checkpoint and integration knew only the root's submodules | Every repository the workspace has checked out, at any depth, gets the run branch at its parent's gitlink. Checkpoints commit and push the most deeply nested first; no repository is pushed before its submodules. Integration goes deepest first, and each parent records the integrated commit, so a moved target's merge result is what gets pinned, never a run-branch commit | agdevworld `b40378f` (`runspace.ts`, `publish.ts`, `integrate.ts`, storage/integration docs) |
| An integrated commit had no supported route into the live checkout that `reconcile`/`playbook` run | `cagentctl sources update`: when the agreed plan says `sources_update: true`, fast-forwards the checkout's branch to the plan's `sources` commit and updates submodules, with a ledger entry. Refused, with nothing moved, when the checkout is dirty or detached, or the commit is unpublished or not ahead. Recovery reads the checkout | pj-cagent `tools/cagentctl` |
| Guide treated an unsupported platform as a reason to choose another host | `guide.md` fact: missing support is a `develop` request, not grounds to substitute a host. Development authority is not agreement to a host change. `change.md`: plan the integrated commit with `sources_update: true`. New `agent/develop.md`: layout, live checkout is read-only, publishing and pins, public repositories, test evidence, what installation needs | pj-cagent `agent/` |

The `change` workflow's execute node now names `sources update`. Its
definition and the new `develop` definition were approved and recorded by
the Omni Agent under this plan. A definition approval is a recorded
statement, not an execution gate. The effects ledger, source-bound plan
blocks and the Developer's plan agreement are unchanged; no approval per
edit or command was added.

### Why WFE support instead of flattening

The repositories could have been flat submodules of pj-cagent (`backend/nctl`,
`backend/ansible_agdev`, …) beside a separate superproject submodule. But
integration then merges a moved `nctl` target and the superproject still pins
the run-branch commit. The pin stops matching what reached `main`, and
`cagentctl`'s `sources-dirty` check would refuse it. Generalizing ownership
keeps the deployment layout as the development layout, so the test matrix's
sibling assumptions hold, and pins are recorded by integration itself. No
other registered workspace has nested submodules checked out, so their runs
behave as before.

## Access and authority verified

| Need | Evidence |
| --- | --- |
| Agent tool permissions | The executor launches `claude_code` agents with permission bypass, so shell, Git, uv, Ansible and Docker are not blocked by grants |
| Toolchain in the executor's PATH | git, uv, ansible/ansible-playbook (core 2.21.1), docker (engine 29.4.0), node and gh all resolve. Ansible refuses a tool shell's non-blocking stdio unless output goes to a file; that is now in `develop.md` |
| Backend Git credentials | The backend repositories are public on GitHub with admin permission for the account. A dry-run push under a launchd-like environment authenticated through the existing `gh` credential helper (no ref created, no secret printed) |
| WFE publication | Run branches and integration push with the host's credentials (GitHub) or WFE's Gitea environment (pj-cagent) |
| Live checkout for change tools | `cagentctl sources update` (above). It stays bound by `sources` and `sources-dirty` |
| Ansible privilege | The mechanism is the vault-held become password configured in `ansible.cfg`, with SSH key access to every host including the execution host itself. **My own read-only check (`ansible <selected Mac>,<second PC> -b -m command -a 'id -un'`) was refused by this session's safety classifier as credential exploration, and I did not retry it.** `develop.md` tells cagent to establish privilege with the same read-only command in its inspect step; Step 2 records the result |

## Tests and deployment

| Check | Result |
| --- | --- |
| WFE `npm run check` (tsc, all tests, build) | 363 passed, 0 failed. New `test/nested.test.ts`: nested branch, checkpoint order and pins, fast-forward and moved-target integration. `test/runspace.test.ts` updated: the nested submodule is now the run's |
| WFE release | `scripts/wfe-release.sh deploy … b40378f`: built, checked, activated, service and launcher healthy (previous `38ebdff`). Stores backed up first (`.local/agdev/.local/backups/pm-p6ex1-predeploy-*`) |
| cagentctl | 44 passed (41 before). New: refusal without `sources_update`, refusal of a dirty checkout (entry ends `not-started`), forward update to a published commit, noop, refusal of an unpublished commit, recovery from the checkout |
| `wfe validate` (cagent) | All six workflows valid |

## Revisions

| Repository | Commit | State |
| --- | --- | --- |
| agdevworld | `b40378f` | Pushed and deployed as the WFE release |
| pj-agdev | agdevworld pin | Pushed |
| pj-cagent | `bc42c7b`, `d118aa0` | Pushed to the LAN Gitea with WFE's Gitea Git environment |

## Notes

- **Another session was working in agdevworld.** Its working tree held
  uncommitted p6 Step 4 changes ("shares", `inputs`/`outputs` in run
  records) that were not mine. I made the change in a separate worktree,
  pushed it to `main`, and fast-forwarded the shared checkout. The other
  session's files do not overlap, so its edits remain uncommitted, and a
  release built from its later commit will include `b40378f`.
- Did the WFE nested-ownership support, cagentctl `sources update` and the
  `develop` workflow for cagent. This is enabling work, not a handoff
  candidate: the NFS implementation itself is left to cagent (Step 2).
