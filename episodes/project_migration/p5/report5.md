# project_migration p5 — Step 5: prerequisites deployed, trial sheet

Written 2026-10-16. The trial prerequisites are deployed, and this sheet is presented to the Developer. **No trial has started.** The design requires the Developer's agreement to this sheet and target, and approval of the definitions, before trial intake. Neither is recorded yet, and neither is inferred from the request to carry out the plan. Request texts, the interruption and evidence scripts, and host facts are in the ignored `pj-agdev/.local/project-migration-p5/step5/`.

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

## Trial target (fresh reads, 2026-10-16)

- **The Proxmox host's platform** has two guests (VMIDs 108 and 109) and no desired row named `agtrial1`.
- **Address.** `192.168.0.199` does not answer and is not in desired state. It is the slot the 2026-08 scratch guest used and that was pruned.
- **The guest.** `agtrial1` is an LXC container: 1 vCPU, 512 MB, 8 GB, the ubuntu-22.04 standard template the DNS guest also uses, VMID 199, static `192.168.0.199/24` via `.1`, MAC `02:00:00:00:00:c7`, mDNS `agtrial1.local`.
- **No dnsmasq record.** With `generate_dnsmasq: false` the change stays off the LAN's DNS guest.
- **Expected creation result.**
  - The plan's reconcile scope is `agtrial1` plus the Proxmox host as control node.
  - `create_compute_instance` runs, then the realization link and the control node's observation.
  - The guest ends `waiting_for_manual_initial_access`: it has no guest OS access, which the nctl recipe documents as the expected end. A repeated dry plan creates nothing.
- **The desired-state write is part of the agreed change.** It was not made as preparation. The batch is written and previewed in T4's own plan step.
- **Retirement (person-operated).** The agent applies the retire batch: `lifecycle: retired`, `desired_presence: absent`. Then:
  1. cagentctl refuses the destroy and prepares `nctl reconcile agtrial1 --allow-destroy --yes --json`, with the check `nctl reconcile agtrial1 --allow-destroy --json`. The check must show exactly one destroy for VMID 199 on the Proxmox host.
  2. After the destroy, `nctl prune agtrial1 --yes --json`, also the person's.
  3. Verification: `nctl reconcile agtrial1 --json` shows no actions, `nctl prune agtrial1 --json` reports `pruned`, and drift shows nothing for `agtrial1`.

## The cases

| Trial | Request (by) | Expected | Who decides / acts | Cleanup |
| --- | --- | --- | --- | --- |
| T1 | Two `consult`s (Developer): which hosts run Ollama, with sources and ages; and the often-asleep laptop's current state | Answers cite `cagentctl read` views with times; the laptop's facts carry their collection age, or are said to be stale or unknown; no `observe` or effect entry exists in either run | Developer accepts or corrects | none |
| T2 | `report` (Developer): Open WebUI on the GPU workstation, port 3000, purpose stated. Then agops `cluster-report-check` delegating a report (the relay's health page) with `accepts: parent`. Then the Developer's first report again. | Records with provenance (the post or the delegation id), verification from reads (`reachable`, `not-found` or `not-checked` with evidence); the third record `duplicate_of` the first; integration into pj-cagent `main`; the parent run's disposition and `received.md` | Developer accepts T2a and T2c; agops's agent accepts the child and the Developer accepts the parent run | none (records stay as trial history) |
| T3 | `scan` (Developer) of the GPU workstation, the laptop and `agtrial1`, after T4 | One `observation` entry with its operation id; per host `observed` or `unavailable` (`agtrial1` has no trusted SSH key until console setup; the laptop if asleep); `snapshot.json` `complete: false` saying why; drift before and after; no repair; report verifications updated | Developer accepts | none |
| T4 | agops `cluster-change-check` delegating the guest creation to `cagent/change`, `accepts: person` | The child's plan question goes to the Developer. If the parent's agent answers it, `cagentctl` refuses `not-an-approver`, which is the delegated-authority check. After the Developer's agreement: `apply` entry (batch hash = the agreed one), `reconcile` entry with operation id and `--allowed-hosts`, fresh observation and drift, change record, the Developer's acceptance and integration. The parent records its disposition. | Developer agrees the plan and accepts the child; agops's agent judges and the Developer accepts the parent | T6 |
| T5 | During T4's `reconcile`: `interrupt.sh` SIGKILLs the **attempt's** process group once the entry is open and its operation running. nctl's own session is untouched. | The attempt ends `interrupted` and the run is held. Observer opens `agops/recover`, and the recovery run continues it (or leaves it held, saying why). The next attempt runs `ledger recover`, which reads `nctl ops show` and `result.json`, then fresh drift. No second create (one `--yes` call in nctl's operation list for `agtrial1`). If the operation's outcome cannot be read, the entry stays unknown and says so. | Observer and recovery as designed; the Developer only if recovery leaves it held | none beyond T6 |
| T6 | `change` (Developer): retire `agtrial1` | Retire batch applied by the agent. The destroy is refused before dispatch (`destroy-class`) and a `person` entry with the prepared command and check is created. The Developer runs the destroy and the prune; `person confirm` records their answer; fresh reads verify. | **Developer runs two commands** (D4) | the guest is gone; Nautobot rows pruned |
| T7 | `change` (Developer): reconcile the command node, expected to have nothing to do | Refused before dispatch (`self-dependency`), with the prepared `nctl reconcile agstudio --yes --allowed-hosts agstudio --json` and a drift check. Run by the Developer if they agree; confirmed from their answer; verified by drift. | **Developer runs one command** (D9), or declines and the refusal alone is the evidence | none |

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
2. **Agreement to this trial sheet and target**: `agtrial1` as specified, cases T1–T7, the T5 interruption, and the person-operated commands in T6 and T7.
3. **During the trials**:
   - the plan agreements of T4, T6 and T7, and acceptances;
   - the commands in T6 (destroy, prune) and T7, or a decision to decline T7's.

   Whether the Omni Agent may answer the *other* questions for the Developer in this episode (consult and report acceptances, parent dispositions the person owns) is the Developer's choice. Plan agreements for changes are asked of the Developer either way.

## Acceptance

| Criterion | State |
| --- | --- |
| Trial prerequisites deployed | met |
| Ownership and cleanup stated per case | met |
| Scripts and request texts prepared | met |
| The agreed scope recorded | **outstanding**: the Developer's agreement |
