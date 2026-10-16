# project_migration p5 — implement and prove the cagent migration

## Basis and intended result

Implement the managed cagent project approved in [pre1/design.md](pre1/design.md), following decisions D1–D14 in [pre1/decisions.md](pre1/decisions.md). [pre1/report.md](pre1/report.md) closes the investigation and design phase; it does not claim an implemented system. This plan covers the implementation phase following pre1.

The result is `pj-cagent`, with `consult`, `report`, `scan`, `change` and readonly `freework`; a project tool `cagentctl`; an observation-only nctl command; a working cluster snapshot consumer; and retirement of the old cagent entrances after their replacements are demonstrated.

Keep Nautobot/nintent, nctl, nodeutils/nauto and Ansible. Preserve legacy records as history. State-platform replacement, continuous convergence, per-target lock enforcement, a WFE scheduler, a second execution host and Nautobot-side report storage remain deferred. Front routing and the argue host's replacement invocation are named follow-ups, not additional implementations in this phase.

Follow the workspace development policies and `localrule.md`: English artifacts, steps starting at 1, Tool Giving, evidence-backed guidance, no backward compatibility by default, deployed and prepared human verification, LAN accessibility for services, and commit/push in each changed repository. Keep machine names, addresses, inventory, credential locations and raw operational evidence in ignored local notes. Project-owned cluster records also need an explicit publication boundary; inspect repository visibility and content placement before publishing them.

## Implementation boundaries

- The Developer agrees each change plan before cluster effects. A request, workflow-definition approval and result acceptance are distinct facts. Delegation does not give a parent agent the Developer's plan authority. Proxy authority is used only when explicitly delegated for this episode.
- Destroy-class changes and changes to cagent's own dependencies remain person-operated, with commands and verification prepared by the agent. This is the agreed D4/D9 boundary, not an additional approval policy.
- `cagentctl` enforces its own contract. Repository bindings, guides and tool wrappers do not establish OS-level isolation or prevent every direct shell route; describe that limit accurately.
- Observation collects data on hosts and ingests it into Nautobot. It is not a database-only read, and must not run repair or provisioning actions. A scheduled observation job remains independent of WFE.
- Unknown effects are investigated, not re-issued. Git integration publishes records; it neither authorizes earlier effects nor reverses them.

## Step 1 — verify contracts and prepare the implementation workspace

Repositories: `pj-agdev/agdevworld`, `pj-agdev/agautolab`, `pj-clusterintent` and its backend submodules; new `pj-cagent`.

1. Record clean/dirty state, revisions, deployed releases and applicable repository instructions. Use Nautobot/nctl to check current registered service state; treat pre1's inventory as historical. Keep raw evidence in `pj-agdev/.local/project-migration-p5/`.
2. Verify WFE contracts for project registration, isolated run copies, custom documentation, generated `_plan`, final result confirmation, delegation, requesters, execution profiles, external waits, Observer recovery, snapshots/artifacts and integration. Identify supported CLI/API operations before writing adapters.
3. Resolve the exact tool-to-run contract: locating the current attempt, reading its plan agreement and approver, finding the agreed plan content and batch hashes, storing durable effects, and making those records available to a later recovery attempt. `WFE_ATTEMPT` plus an arbitrary supplied hash is not proof of an agreed plan.
4. Close implementation ambiguities in a short contract note: observation authority for `scan` versus the `change` plan gate; whether an observation has an effects entry or an observation operation record; the snapshot consumer's schema and selection rule; and how a long nctl operation survives or reports interruption when an attempt ends. Keep the approved behavior; return only substantive design changes to the Developer.
5. Establish how ignored configuration reaches run copies, and how redaction handles actual secret values. Listing an env file containing only credential paths does not demonstrate redaction of the referenced contents. Verify with synthetic secrets, not production tokens.
6. Create the project skeleton in an isolated checkout: `project.yaml`, declared areas, `agent/`, `tools/cagentctl/`, `records/`, `desired/batches/`, workflow storage and ignored configuration. Confirm the destination repository's publication scope. Do not copy raw cluster inventory into the shared episode repository.

Deliverable: `report1.md` with revision anchors, a capability/gap matrix, the concrete run/tool contract and repository ownership. Acceptance: implementation seams are identified; any required common-system change is bounded and justified; no operational migration has occurred implicitly.

## Step 2 — add and deploy observation-only collection

Repositories: `pj-clusterintent/nctl`; `pj-clusterintent` for the submodule pin and refresh script.

1. Add `nctl observe [HOST]…` using the existing observation/ingest implementation and operation evidence. Define explicit host resolution and the no-host default; an invalid or unresolved target must not widen the target set.
2. Execute only scoped `observe_node` collection/retrieval and ingest. Preserve current SSH trust behavior, operation IDs, artifacts and per-host results. Do not invoke unrelated reconcilers when the snapshot also contains repairable drift.
3. Specify the JSON envelope and exit semantics for complete success, partial collection, ingest failure and unavailable hosts. Separate successful observation from convergence: existing drift is not a reason to repair.
4. Test the command, action boundary and error paths with fake adapters. Include an input with unrelated actionable drift and assert that no repair is dispatched. Run the backend's required checks, including the compute-conformance freshness gate named in its development instructions.
5. Update `devenv/observation/refresh-node-observations.sh` to consume the new command and envelope, retain failed results, and timestamp outcomes. Preserve its independent schedule and bounded host scope.
6. Commit/push the submodule and parent pin. Deploy the script/runtime and observe one bounded refresh cycle. Compare its actual action records with the intended host set and inspect ingested observation times. Do not describe an unreachable host as successfully refreshed.

Deliverable: `report2.md`, command documentation, test results and redacted references to the live operation evidence. Acceptance: the scheduled path no longer invokes general `reconcile --yes`; all recorded actions are observation-only. A deployment recovery procedure must not silently restore unattended repair.

## Step 3 — implement `cagentctl` and its durable records

Repository: `pj-cagent`; narrowly scoped common-system changes only if Step 1 establishes a missing interface.

1. Implement `read`, `observe`, `apply`, `reconcile`, `playbook`, `ledger` and `snapshot` with JSON output and discoverable `--help`. Reuse the old read-only adapter with source/commit provenance; do not recreate nctl's state model or executor.
2. For changes, bind the active attempt to the agreed plan and exact batch content, command scope and targets. Refuse absent/mismatched agreement before effects. Re-read relevant prerequisites before execution; changed assumptions return to a question instead of extending the agreed scope.
3. Resolve destroy-class and self-dependency effects before dispatch, including actions inferred by reconciliation rather than only explicit CLI flags. Refuse them with a concrete person-operated command and verification. If nctl cannot expose enough information to establish the action set, record the gap and resolve it before allowing that execution path.
4. Persist an entry before each change effect, then append/update its outcome with operation references and evidence. Use durable writes and identities that survive attempt restart. Preserve an incomplete entry as unknown. Distinguish a prepared person handoff from a confirmed person action; preparing a command is not proof it ran.
5. Make operation correlation usable after a lost response: retain the backend operation identifier as early as the backend permits and record the available lookup evidence. Recovery reads the ledger, `nctl ops show`, then fresh observation/drift. If correlation or outcome remains ambiguous, keep it unknown.
6. Preserve the original agreed batch bytes and their digest, report provenance, duplicate/supersedes references and a discoverable records index. Ensure distinct runs cannot accidentally allocate the same record identifier. The target index remains advisory; do not introduce the deferred target lock.
7. Build `snapshot` against the actual cluster view's drift/actual/workspace contract. Record source times, failures and artifact identity. A newer failed scan must not silently masquerade as a fresh complete snapshot.
8. Test refusal-before-dispatch, altered batches, generated destroy actions, self-dependency handoff, partial success, interrupted ledger writes, lost responses, repeat recovery, scope selection, duplicate records and synthetic-secret redaction. Use fake effects for failure injection until the live trial is explicitly prepared.

Deliverable: `report3.md`, the tool and its tests. Acceptance: each allowed operation has traceable evidence; refused commands dispatch nothing; an unknown effect cannot be repeated as a recovery shortcut. Long-operation ownership and external-wait behavior are demonstrated with a fake operation before live use.

## Step 4 — complete workflow definitions, guides and registration

Repository: `pj-cagent`; ignored WFE/intake configuration in `pj-agdev`.

1. Implement the approved workflow shapes and bindings from design §4. Use `custom` documentation and put result confirmation last. `consult`/`freework` perform no host mutation; `report` verifies through reads; `scan` collects and records without repairing; `change` registers, obtains plan agreement, executes, verifies and confirms.
2. Write guides as role facts and output contracts, with command usage in `cagentctl --help`. State source age/unknowns, reported versus observed versus desired information, prerequisite checks, person handoffs, recovery ownership and the meaning of `one-time` versus `maintain`. Neither kind enables autonomous convergence.
3. Make report integration and subsequent scans preserve provenance and contradictory evidence. An unreachable host is not evidence that a reported installation is absent. Copy useful purpose information into desired placement configuration only through an agreed change.
4. Validate definitions in a scratch registry, then publish/register/obtain the project. Configure the approved execution profile, Developer requester, named agent requesters and delegated report/change entry. A delegated request still addresses plan agreement to the Developer.
5. Prepare ignored local configuration and redaction on the deployed execution path. Inspect run access and artifact paths from a disposable run copy rather than assuming the working checkout's files are available.
6. Present valid, inspectable definitions for the Developer's definition approval through the existing editor. Apply any resulting corrections before trial intake starts.

Deliverable: `report4.md`. Acceptance: all five definitions validate; the registered project and its request channels resolve; run copies have the intended records/tool/configuration access; definition approval is recorded as itself, not as permission for a live change.

## Step 5 — deploy trial prerequisites and prepare the trial sheet

1. Commit/push and deploy any common-system changes required by Steps 1–4. Record source/release correspondence and back up affected operational stores before a migration. Use the real executor and LAN-accessible run views for the trials.
2. Use fresh Nautobot/nctl reads to select the disposable guest target, its resource scope, expected creation result and person-operated retirement command. Do not create a desired-state record as an unrecorded preparation step; include that write in the agreed change.
3. Prepare one managed project's delegation into `report`, plus a delegated `change` case to verify that child plan agreement still belongs to the Developer. The cases may share the disposable guest scenario.
4. Prepare a stale-data consultation and an unavailable-host observation case. Prefer an already unavailable or isolated trial target; do not disrupt an unrelated service to manufacture failure.
5. Specify interruption timing, process scope, operation lookup, cleanup and expected recovery. Distinguish interrupting an attempt from terminating its backend operation. Prepare all commands and evidence collectors before asking for trial agreement.
6. Present the trial sheet and target to the Developer with ready request/run links. Record the agreement required by the approved design; do not infer it from the request to write this plan.

Deliverable: `report5.md` with expected results, ownership and cleanup for each case. Acceptance: trial prerequisites are deployed and the agreed scope is recorded; the Developer is left only the human decisions and person-only operations required by D4/D9.

## Step 6 — run staged trials and repair demonstrated defects

| Trial | Behavior | Required evidence |
| --- | --- | --- |
| T1 | Two consultations, including stale or unknown data | Answers cite sources and ages; no collection or mutation is disguised as a read; result decisions recorded. |
| T2 | Direct installation report, delegated report and duplicate | Provenance and verification records, duplicate relation, accepted integration, and parent's result disposition. |
| T3 | Scoped scan including an unavailable host | Collection/ingest operation IDs, successful and failed hosts distinguished, updated records and usable snapshot; no repair action. |
| T4 | Disposable guest creation through `change`, with delegation coverage | Developer agreement on batch/scope, closed effects entries, fresh observation and drift, service probes where relevant, result acceptance and integration. |
| T5 | Interrupt T4 or a bounded follow-up at the prepared point | Durable unknown entry, held attempt, recovery reading ledger → operation evidence → fresh state, no repeated unknown effect; unresolved ambiguity remains explicit. |
| T6 | Retire the disposable guest | Tool refusal before destruction, prepared command, person confirmation, evidence attributed to the person and fresh post-action verification. |
| T7 | Harmless self-dependency handoff | Refusal before dispatch, prepared command/check, person execution if agreed, recorded performer and independent verification. |

Use fake tests for altered prerequisites, duplicate delivery and unknown recovery where a live experiment would add no evidence. Exercise a changed prerequisite through a bounded live case only when needed to resolve a remaining implementation concern.

Read results from run records, integration status, ledger entries and backend operation evidence, not agent statements alone. Record costs, manual interventions and defects. Fix defects and repeat the affected cases; a defect undermining the agreed effect boundary, recovery or replacement path blocks retirement. Complete guest cleanup, or explicitly retain responsibility for an outstanding person-operated cleanup.

Deliverable: `report6.md`, trial evidence and defect dispositions. Acceptance: T1–T7 are demonstrated, accepted where required, and linked to actual records; no outcome is promoted from unknown to success without evidence.

## Step 7 — switch the cluster snapshot consumer and prove deployment

Repositories: `pj-agdev/agdevworld`; `pj-cagent` if artifact publication needs adjustment.

1. Replace `cluster:fetch`'s old HTTP-agent request with the scan artifact source agreed in D12. Define which completed scan is selected, how to locate/download its artifact, and how missing, partial or stale snapshots appear. Reuse existing artifact access rather than building another state service.
2. Test parsing and selection with representative artifacts, then deploy the consumer. Verify it with a fresh scan produced through the real workflow and inspect the actual cluster view.
3. Prepare a request link and run/result view for each workflow. Have the Developer make the minimal deployed consultation check required by the overall plan. Keep the legacy API until the new snapshot consumer is proven.

Deliverable: `report7.md`. Acceptance: the deployed cluster view consumes the scan artifact, shows its freshness honestly and no longer relies on cagent-api; the Developer's consultation reaches the managed project.

## Step 8 — retire legacy paths and reconcile registered state

Repositories: `pj-clusterintent`, `pj-clusterintent/ansible_agdev`; ignored host configuration and the registered desired state.

1. Recheck active callers, open requests and scheduled dependencies immediately before cutover. If work appeared since pre1, give it an explicit disposition before stopping its entrance; preserve historical conversations and evidence.
2. Stop the old Zulip listener, publish the agreed retirement introduction, then stop the API after Step 7. Archive local service definitions and remove obsolete deployment templates. Keep the old cagent source for the agreed history interval with a retirement notice.
3. Remove the obsolete `executor/`, `autolab_gateway`, `cagent_client`, their unused playbooks and the power webhook after checking references. Preserve shared Ansible roles still needed by nctl observation/reconciliation. Run relevant repository checks.
4. Prepare and preview the Nautobot retirement batch for the Developer to apply as agreed in D5/D10. Verify its actual application and resulting drift instead of equating a stopped process with a retired placement. If an automatic approval review blocks an action, report the specific rejection; do not generalize historical refusals to unrelated actions.
5. Confirm the observation-only job still works independently, old entrances stay stopped, the snapshot view works and new requests reach the managed project. Document operational recovery using the retained artifacts without silently re-enabling unattended repair.
6. Record Front's routing follow-up (`wf-cagent.consult`, `wf-cagent.change` and requester configuration) and the argue host's consult handoff, with owners and completion criteria. Do not recreate compatibility shims by default.

Deliverable: `report8.md`. Acceptance: replacement paths are proven, obsolete services and registered placements agree on retirement, useful history remains accessible and outstanding caller changes have explicit owners.

## Step 9 — close the phase and publish evidence

Write `report.md` covering the delivered workflows/tool/backend command, deployed revisions, trial evidence, costs, defects, person-operated actions, cleanup, retirement, follow-ups and deferred decisions. Update operating documentation and local-only environment notes in their appropriate repositories.

Review diffs for local environment details, secrets, unsupported claims and unrelated changes. Commit/push each affected repository and then its parent gitlink where applicable. Keep pre1's approved design and decision history intact, recording any later amendments explicitly in the implementation report.

## Completion criteria

- Five managed workflows operate through the deployed path with the agreed authority and documentation behavior.
- Observation-only collection is tested and used by the independent refresh job; scans do not repair drift.
- Changes are linked to the Developer's agreed plan, exact inputs, durable effects evidence and fresh verification; ambiguous outcomes remain unknown and are not blindly retried.
- Destroy-class and self-dependency operations use the agreed person-operated handoff, with no false claim that the agent performed them.
- T1–T7, delegated plan authority, snapshot consumption and cleanup have evidence; relevant defects are resolved before retirement.
- Legacy paths are retired in both process and registered state, history is retained, follow-ups have owners, and all intended changes are committed and pushed.
