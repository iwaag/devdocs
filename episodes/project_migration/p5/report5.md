# project_migration p5 — Step 5: prerequisites deployed, trial sheet

Written 2026-10-16. The trial prerequisites are deployed, and this sheet is presented to the Developer. **Historical preparation record:** trials subsequently ran; see [report6.md](report6.md). At preparation, no trial had started. The design requires the Developer's agreement to this sheet and target, and approval of the definitions, before trial intake. Neither is recorded yet, and neither is inferred from the request to carry out the plan. Request texts, the interruption and evidence scripts, and host facts are in the ignored `pj-agdev/.local/project-migration-p5/step5/`.

## Deployed

| Item | State |
| --- | --- |
| WFE release | agdevworld `38ebdff` (previous `47a365d`), `wfe-release.sh deploy`. The release's own check passed 360/360, and the service and `bin/wfe` report the new commit. No store migration. |
| Backups before | exec, notify, intake, present and delegate stores (`.backup`), and the registry and `zulip`, `system`, `present` and `node-catalog` JSON, in the area's ignored `backups/pm-p5-predeploy-*` |
| Executor | Unchanged. It calls `bin/wfe`, so the redactor change reached its notify pass with the release. |
| nctl and observation | Step 2: `nctl` `8bb4d3b`, observation-only refresh job |
| pj-cagent | `c033533`, registered; channels `wf-cagent.*` |
| agops (delegating project) | `bc34931`: `cluster-report-check` (delegate to `cagent/report`, `accepts: parent`) and `cluster-change-check` (delegate to `cagent/change`, `accepts: person`), both valid |
| pj-agdev | pointer to agdevworld `38ebdff` |
| LAN | Run views and the editor are served through agdevworld's nginx, as in p4 |

## Trial target

A disposable, resource-bounded LXC guest on the existing Proxmox platform,
with an unused identifier/address checked before the creation run. Exact
host names, addresses, VMID, template, resources, network settings and raw
reads are retained in ignored `pj-agdev/.local/project-migration-p5/step5/`
notes, not in this portable report.

The expected creation result is a realized guest awaiting manual initial
access, with no guest SSH enrollment and no DNS-service modification. The
desired-state batch is part of the agreed change, not an unrecorded setup
step. Retirement first declares absence, then a person performs the
prepared destroy command. Once destruction is independently verified, the
documented prune route removes its linked actual roots and desired
records. The Developer subsequently explicitly instructed the agent to
execute prune; the takeover records its actual Omni Agent execution.

## The cases

| Trial | Request (by) | Expected | Who decides / acts | Cleanup |
| --- | --- | --- | --- | --- |
| T1 | Two `consult`s (Developer): which hosts run Ollama, with sources and ages; and the often-asleep laptop's current state | Answers cite `cagentctl read` views with times; the laptop's facts carry their collection age, or are said to be stale or unknown; no `observe` or effect entry exists in either run | Developer accepts or corrects | none |
| T2 | `report` (Developer): Open WebUI on the GPU workstation, its documented endpoint, purpose stated. Then agops `cluster-report-check` delegating a report (the relay's health page) with `accepts: parent`. Then the Developer's first report again. | Records with provenance (the post or the delegation id), verification from reads (`reachable`, `not-found` or `not-checked` with evidence); the third record `duplicate_of` the first; integration into pj-cagent `main`; the parent run's disposition and `received.md` | Developer accepts T2a and T2c; agops's agent accepts the child and the Developer accepts the parent run | none (records stay as trial history) |
| T3 | `scan` (Developer) of the GPU workstation, the laptop and `the disposable trial guest`, after T4 | One `observation` entry with its operation id; per host `observed` or `unavailable` (`the disposable trial guest` has no trusted SSH key until console setup; the laptop if asleep); `snapshot.json` `complete: false` saying why; drift before and after; no repair; report verifications updated | Developer accepts | none |
| T4 | agops `cluster-change-check` delegating the guest creation to `cagent/change`, `accepts: person` | The child's plan question goes to the Developer. If the parent's agent answers it, `cagentctl` refuses `not-an-approver`, which is the delegated-authority check. After the Developer's agreement: `apply` entry (batch hash = the agreed one), `reconcile` entry with operation id and `--allowed-hosts`, fresh observation and drift, change record, the Developer's acceptance and integration. The parent records its disposition. | Developer agrees the plan and accepts the child; agops's agent judges and the Developer accepts the parent | T6 |
| T5 | During T4's `reconcile`: `interrupt.sh` SIGKILLs the **attempt's** process group once the entry is open and its operation running. nctl's own session is untouched. | The attempt ends `interrupted` and the run is held. Observer opens `agops/recover`, and the recovery run continues it (or leaves it held, saying why). The next attempt runs `ledger recover`, which reads `nctl ops show` and `result.json`, then fresh drift. No second create (one `--yes` call in nctl's operation list for `the disposable trial guest`). If the operation's outcome cannot be read, the entry stays unknown and says so. | Observer and recovery as designed; the Developer only if recovery leaves it held | none beyond T6 |
| T6 | `change` (Developer): retire `the disposable trial guest` | Retire batch applied by the agent. The destroy is refused before dispatch (`destroy-class`) and a `person` entry with the prepared command and check is created. The Developer runs the destroy and the prune; `person confirm` records their answer; fresh reads verify. | **Developer runs two commands** (D4) | the guest is gone; Nautobot rows pruned |
| T7 | `change` (Developer): reconcile the command node, expected to have nothing to do | Refused before dispatch (`self-dependency`), with the prepared the prepared host-scoped reconcile command and a drift check. Run by the Developer if they agree; confirmed from their answer; verified by drift. | Original D9 handoff; superseded by the [Developer amendment](self-dependency-removal.md) | none |

Fake tests already cover changed prerequisites, duplicate delivery and unknown recovery (report3), so no live case manufactures those. Nothing unrelated is disrupted to make a failure: the unavailable hosts in T3 are a guest without guest access and a laptop that may be asleep.

**Interruption details (T5).**

| | |
| --- | --- |
| Timing | after `reconcile`'s ledger entry is `spawned` and before its operation ends (creation takes minutes) |
| Process scope | the attempt's process group only (the executor's attempt pid = its group). The script refuses if that group is the backend's. |
| Operation lookup | `nctl ops show <id>` from the entry |
| Expected recovery | Observer → `agops/recover` → `wfe run recover` → the run's next attempt reconciles its ledger |
| Cleanup | none. T6 retires the guest. If T5 left the operation's outcome unknown, T6 starts from fresh drift rather than assuming the guest exists. |

**Evidence.** `collect.sh <workspace> <wf>/<run> [<host>]` copies:

- the run record;
- the attempts;
- the run folder with its ledger;
- `nctl ops show` for every entry's operation;
- host drift.

Results are read from these, not from agent statements.

## What the Developer is asked for

1. **Definition approval** of `cagent`'s five workflows and agops's two new ones, in the editor. This is a statement about the definitions only.
2. **Agreement to this trial sheet and target**: `the disposable trial guest` as specified, cases T1–T7, the T5 interruption, and the person-operated commands in T6 and T7.
3. **During the trials**:
   - the plan agreements of T4, T6 and T7, and acceptances;
   - the commands in T6 (destroy, prune) and T7 (the T7 host restriction was subsequently removed), or a decision to decline T7's.

   Whether the Omni Agent may answer the *other* questions for the Developer in this episode (consult and report acceptances, parent dispositions the person owns) is the Developer's choice. Plan agreements for changes are asked of the Developer either way.

## Acceptance

| Criterion | State |
| --- | --- |
| Trial prerequisites deployed | met |
| Ownership and cleanup stated per case | met |
| Scripts and request texts prepared | met |
| The agreed scope recorded | **outstanding**: the Developer's agreement |
