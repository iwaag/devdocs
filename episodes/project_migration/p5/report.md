# project_migration p5 — final report: cagent migration

Status: implementation, deployment and operator verification complete
(2026-10-16). The work was taken over after Steps 1–5 and the initial trials
had run. [The plan](plan.md), [approved design](pre1/design.md) and
[Developer amendment](self-dependency-removal.md) define the scope.

## Delivered

The private managed project `pj-cagent` now provides five deployed
workflows: `consult`, `report`, `scan`, `change` and `freework`. Installation
reports keep purpose, ownership and provenance; scans observe without
repair; changes use agreed batch/scope, durable effects and fresh evidence.
Nautobot/nintent, nctl, nodeutils/nauto and Ansible remain the backend.

`cagentctl` no longer refuses dependency hosts. Its `apply`, `reconcile`
and `playbook` paths use ordinary plan/scope checks there. The independently
agreed destroy-class handoff remains. A scope bug found during takeover is
also fixed: explicit `expect: []` means no new action is allowed, rather
than skipping the check. Missing command coverage is not another
prohibition; supported direct nctl work retains an effects record.

The independent refresh job uses observation-only `nctl observe`.
`cluster:fetch` now reads the managed scan artifact through WFE, validates
all source envelopes and retains provenance/partial state. The deployed
nodes/workspaces views display snapshot time, age and coverage. They do not
need the retired HTTP agent or its human token.

Legacy HTTP/listener services are stopped and their plists archived. The
old service/agent are retired and placement disabled in Nautobot. Obsolete
executor, webhook and Ansible client/gateway deployment paths are removed.
Source and historical evidence remain available without a live listener.

## Evidence and validation

- [report6.md](report6.md): T1–T7, interruption/recovery, delegation,
  cleanup, corrections and operator interventions.
- [report7.md](report7.md): actual scan artifact import, frontend deployment,
  browser screenshot/read verification, timestamp/coverage behavior and
  confirmation that import still works after API shutdown.
- [report8.md](report8.md): source retirement, registered/actual state,
  process/port checks and follow-up owners.
- All **11 cagent runs** completed, have accepted decisions and integration
  records, and pass `wfe run check` without warnings. The two delegated
  parent trials are also completed, accepted and integrated. The freework
  smoke check changed only its run documents and dispatched no effect.
- Checks: **31 cagentctl tests**, **2 importer tests**, frontend build and
  Docker deployment, **4 Ansible conformance tests**, **23 retained legacy
  TLS/client tests**, and the required clean Nautobot runtime gate
  (**263 cases**) passed.

The Developer's actual plan agreements and guest-destruction confirmation
remain in the records. Takeover trial decisions and manual lifecycle
close-out are attributed to Omni Agent; they are not presented as new
Developer-performed operations or a human browser trial. Results were
checked against operations, files, fresh state and Git integration rather
than agent claims.

## Trial resource and final operating state

The disposable guest was created once, survived the prepared attempt
interruption through the backend operation, and was later destroyed with
the Developer's confirmation. Omni Agent completed the already-authorized
prune, recorded its actual performer/operation, and verified the guest no
longer appears in drift/actual. No trial guest cleanup remains.

Both legacy listener labels are absent and all three old ports are closed.
Fresh post-retirement observation succeeded. Final backend drift is
**45 converged / 1 drifting**. The one Observer `service_missing` condition
was present before migration and remains a proposed change, not an
incidental repair.

The browser consumes the published scan generated at 03:39:41 UTC. A later
backend observation verifies retirement; the static view remains an
explicitly timestamped snapshot of its scan, not continuous live state.
Future freshness uses a new scan followed by `cluster:fetch` and static web
publication. A WFE scheduler and continuous convergence remain deferred.

## Revisions and operation

| Repository/component | Result |
| --- | --- |
| `pj-cagent` (private) | Source fixes include `9cc0601` (remove host refusal), `f7d7697` (tool coverage/evidence), `cecbbc7` (empty scope); final trial integrations through `fe86534`. |
| `nctl` | `8bb4d3b`: observation-only command, delivered before takeover. |
| `ansible_agdev` | `e74195d`: obsolete roles/playbook/webhook removed. |
| `pj-clusterintent` | `685d0cc` removal/pin; final retirement intro `bb64e6a`. |
| `agdevworld` | `e5488f6`: importer, view metadata and operating guidance, deployed. |
| `pj-agdev` | `2d74765`: published frontend pointer. |
| WFE installed package | `38ebdff`; no WFE package changes were needed during takeover. |

Use the managed workflow's request topic, the editor's request action or a
delegation. `consult` reads existing state; `scan` collects and publishes;
`change` obtains the Developer's agreement before effects. Credential
locations, actual host addresses, prepared rollback batch, archived plists,
operation IDs, screenshots and raw evidence are in ignored
`pj-agdev/.local/project-migration-p5/` notes and the private project.

## Costs, limits and follow-ups

The known reported subtotal is **USD 22.42361** across 50 measured final
results from 51 attempts. One interrupted attempt has no final cost record,
so this is a subtotal rather than a complete billed total. Each attempt's
last final result is used; repeated intermediate results are not summed.
The figures cover the collected cagent and delegated/recovery trial runs,
not these outer development sessions or all infrastructure work. Final
trial queues have no active attempt or queued job.

Front routing and the argue host's managed consultation invocation remain
follow-ups with owners and completion criteria in report8. Deferred:
state-platform replacement, nctl's long-term fate, continuous convergence,
per-target locks, a WFE scheduler, a second execution host and Nautobot-side
report records. Collector evidence is bounded by what it observes; a
`not-found` report verification is not proof that an arbitrary service
cannot exist outside that evidence.

Omni notes: completed authorized prune, repaired guidance/scope handling,
reviewed the takeover trials, recorded supported confirmation/cleanup and
integration operations for cagent — handoff candidates. Earlier record-index
conflicts also required manual integration; no fully autonomous migration
success is claimed.
