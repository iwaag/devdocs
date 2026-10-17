# Workflow editor p11 — remove development-process restrictions

## Objective

Realize [braindump.md](braindump.md): remove rules and deterministic guards that
limit agents' freedom in WFE, nctl, cagentctl and the development processes that
use them. Include shared guides, project-specific rules, generated instructions,
workflow definitions, local configuration and deployment tooling. Retain rules
that guarantee freedom; describe tools and observed behavior without turning
that information into mandatory procedures.

This is a removal project, not a broader approver list, an additional bypass
flag, a permissive mode beside the old restricted mode, or a new approval
process. The default supported path must give agents the freedom requested.
Earlier episodes' decisions to retain a guard do not exempt it from this work.

This document plans the work. Its creation does not execute the implementation,
resume stopped runs, change infrastructure or remove existing data.

## Starting evidence and ownership

Paths below are relative to the projects directory unless a managed project is
named. Resolve managed project locations from the current registry; do not put
machine-specific workspace paths or configuration values in tracked reports.

| Surface | Inspected starting points | Work to cover |
| --- | --- | --- |
| WFE engine | `pj-agdev/agdevworld/workflow_editor/shared/{approval,compile,run,validate}.ts`; `server/{runs,exec,integrate,boundary,access,git,publish,shares}.ts` | Definition approvals, generated plan/report gates, confirmation and acceptance requirements, revision/graph restrictions, repository boundaries, publication restrictions and integration refusals |
| WFE interfaces and installation | `workflow_editor/cli/`, UI, `templates/AGENTS.md`, `docs/`; `pj-agdev/agdevworld/scripts/wfe-release.sh` | Restrictions in CLI/API/UI/help, generated authoring guidance, installation and release checks |
| Executor and shared instructions | `pj-agdev/agautolab/agent/guides/wfrun/guide.md`, `src/agautolab/wfexec.py`; shared guide and harness code in `pyagag` | Required human questions, prescribed sequencing, commit/push/post and workspace prohibitions, launcher permissions and automatic stopping |
| Managed cagent | `tools/cagentctl/src/cagentctl/{context,effects,sources,nctl,ledger,cli}.py`, `agent/`, workflow definitions and `.local/cagent.toml` | Approver identity, workflow allow-lists, exact-plan agreement, batch/source approval binding, host/playbook allow-lists, compulsory person handoffs |
| Cluster backend | `pj-clusterintent/nctl/src/nctl_core/cli/main.py`, `reconcile/executor.py`, action handlers and related backend packages | Confirmation switches, destroy-class gates, imposed host scopes, policy refusals and downstream enforcement |
| Managed playprey | `agent/guide.md`, `README_DEV.md`, `knowledge/`, workflows; `godot_lab/mouth_start/tools/delivery.py` | Local prohibitions, mandatory human tuning boundaries, dirty-tree delivery refusal and workflow-specific review gates |
| Other development entry points | Registered projects, `devpolicy`, `devdocs/README_DEV.md`, `localrule.md`, settings repositories, active agent guides, project instruction files and hooks | Restrictions inherited outside the three named tools, including instructions regenerated at project creation or run launch |

The managed cagent project is the active cagentctl owner. The retired
`pj-clusterintent/cagent` implementation and `experiments/workflow_editor` are
historical sources; determine whether anything still consumes their templates
before editing them. A change only to an archived copy does not implement p11.

Planning-time `nctl status --json` succeeded: Nautobot was reachable and
authenticated, the intent API was present, and the worker had no pending jobs.
Observation ages varied; this is connectivity evidence, not a claim that every
service or host was freshly verified. Re-read Nautobot/nctl when implementing
or deploying. Keep raw cluster details in ignored evidence.

### p6 failures to reproduce

Use `devdocs/episodes/project_migration/p6/report7.md`, `mid-report7.md`,
`report3.md`, and `ex1/quick-fixes.md` in that episode as evidence, not as the
new policy.

| Evidence | Required p11 outcome |
| --- | --- |
| S1: shared guide prohibited committing; delivery required a commit | Remove remaining Git-action prohibitions and their generating sources. The earlier commit fix is already present; verify it rather than claiming it as new work. |
| S2: blanket dot-entry filtering omitted a file named by the delivery manifest | Permit intended dotfiles in delivery; explicitly identify transport metadata rather than excluding a class of user files. Remove the project workaround that silently drops desired content. |
| S3: a generated final report after confirmation invalidated integration | Reporting and further edits do not require another acceptance. Remove the acceptance prerequisite rather than adding a validator that forbids the workflow. |
| S4: external auto-mode classifier refused authorized operations | Identify the actual harness owner and supported configuration separately. Repository edits cannot promise removal of a provider-side restriction. |
| S5: WFE's 50 MiB rule refused publication/checkpointing | Remove the size-based development veto, including its guide/test equivalents. Shared storage remains an available tool, not a mandatory route. |
| S6: cagent plan edits repeatedly required agreement | Initial and subsequent operations do not depend on plan approval, plan hashes, a configured approver or answer take-up. |
| cagent repair required playbook registration and person steps | Agents can select tools and perform supported operations directly; no human-only role or per-command allow-list remains in the project workflow. |
| playprey delivery required a clean tree | Permit delivery from a working tree and record the actual input snapshot; do not disguise working bytes as a clean commit. |

## Step 1 — inventory restrictions and their active consumers

1. Record repository revisions and existing changes. Map independent repositories,
   submodules, managed workspaces, run copies, installed releases and generated
   guides. Inspect current runtime facts through Nautobot/nctl.
2. Enumerate every registered project and its effective instructions: `AGENTS.md`,
   `CLAUDE.md`, `README_DEV.md`, `localrule.md`, tool-specific instruction files,
   local settings, hooks, workflow text and imported guides. Include parent
   instructions and ignored configuration; a tracked-files-only search is
   insufficient. Record names and effects, not credentials or local addresses.
3. Trace restrictions from text and UI through CLI/API to the final implementation.
   Search refusal paths, allow-lists, approval/confirmation checks, role checks,
   hard-coded limits and tests, then read callers to establish their actual effect.
   Include backend nintent/Ansible checks when nctl merely forwards a restriction.
4. Produce an inventory with owner, source, active consumers, trigger, observed
   effect, proposed deletion and proving scenario. Distinguish policy vetoes from
   missing capabilities and real execution errors. A rule does not survive merely
   because it was called safety, validation, integrity or a historical contract.
5. Audit project-specific local rules explicitly, including the root `localrule.md`
   and rules transitively loaded by cagent/playprey and other managed projects.
   Identify which statements guarantee autonomy and which prescribe or prohibit
   actions. Record the disposition of each active restriction.

Deliverable: `report1.md` and a restriction inventory used through subsequent
steps. This inventory is implementation coverage, not a permission gate.

## Step 2 — make WFE a tool for agent-directed work

1. Remove mandatory definition approval, generated plan agreement, fixed report
   completion, human confirmation and result acceptance as execution or integration
   prerequisites. Keep plans, reports, questions, reviews and decisions available
   as records that an agent or person can choose to use. Do not fabricate an
   approval or acceptance when none happened.
2. Remove policy-based node ordering, adjustment, reopening, pending-instruction
   acknowledgement and delegation restrictions. Let agents revise the work graph,
   bindings, execution choices and completed work when needed, recording what
   changed. Dependencies remain useful planning information; they must not become
   another permission mechanism. Preserve explicit user stop/cancel instructions
   as user intent rather than treating them as automatic policy holds.
3. Remove readonly/protected-project and fixed-path policy vetoes across authoring,
   run execution, cross-project access, CLI and API. Give operations an explicit
   target and accurate help. Remove size limits that prohibit storing, committing,
   pushing or publishing intended work. Streaming/chunking can address resource
   limits without a blanket refusal policy.
4. Decouple integration from accepted-revision hashes and human decisions. Record
   which commits and working changes were actually integrated; later reporting
   must not trigger re-agreement. Provide agent-usable merge/conflict resolution
   instead of a mandatory human handoff.
5. Review execution ownership, locking, retries and state validation individually.
   Keep mechanics necessary to make an operation well-defined and report real
   failures, but remove behavioral restrictions layered on them. Where a running
   attempt or conflicting write prevents an immediate operation, expose the
   concrete state and an agent-operated transition rather than an approver gate.
6. Update model/types, storage readers, API, CLI, browser controls, executor queue,
   help and tests together. Eliminate dormant restrictive branches and flags;
   do not leave the UI permissive while the executor refuses the same action.

Deliverable: `report2.md`, updated WFE behavior and focused regression evidence
for previously refused operations, including p6 S2/S3/S5.

## Step 3 — remove cagentctl and nctl process gates end to end

1. Separate cagentctl's useful run/context discovery and effect recording from
   `agreed_plan`, approver membership, workflow type and answer-state enforcement.
   Operations can use an ordinary run context or a directly supplied context;
   neither requires a special change-workflow approval ritual.
2. Remove agreed batch hashes, expected preview totals, approved source revisions,
   `sources_update` permission, playbook allow-lists and compulsory host allow-lists
   as authority checks. Keep previews, hashes, revisions and target selection as
   useful information and operation arguments. An explicit target selector still
   identifies what to operate on; dropping an approval scope must not silently
   reinterpret that selector as the entire cluster.
3. Remove destroy-class and privileged-operation person handoffs. Expose supported
   operations to agents with usage information. Where a missing command forced a
   handoff, provide the operation or a direct-tool path instead of an exception
   list. Remove the read-command allow-list wrapper as a restriction on tool choice.
4. Follow these operations into nctl and downstream handlers. Remove extra
   confirmation/authorization switches and policy refusals, including
   `allow_destroy` and imposed `allowed_hosts` enforcement. Keep an explicitly
   selected preview operation separate from execution; do not unexpectedly turn
   existing read-only planning into mutation by deleting a flag check.
5. Keep records of attempted effects, actual commands, source revisions, outcomes
   and failures. Unknown outcomes remain visible and investigable; an agent can
   choose recovery or retry without a person-only release step. Do not equate
   freedom to execute with pretending that a failed or unobserved effect succeeded.
6. Update cagent workflows, guides, examples, backend documentation and tests with
   the new semantics. Check both the managed project's backend pins and the live
   backend checkout so removed guards do not remain in the actual dispatch path.

Deliverable: `report3.md` with demonstrated cagentctl-to-nctl operations without
approver configuration, plan agreement, repeated agreement or playbook registration.

## Step 4 — remove restrictions from shared and project-local guidance

1. Rewrite shared executor guidance as identity, available context, tools and
   observed facts. Remove mandatory human questions, commit/push/post prohibitions,
   required document order and workspace/scope restrictions found in Step 1.
   Put tool syntax in help and explain capabilities without prescribing one route.
2. Apply the same audit to every active project's guidance and workflow definitions,
   including cagent, playprey, forge and archsage if currently registered. Remove
   local prohibitions about storage choices, tuning, optimization, platform choice,
   delegation, completion and method selection. Keep factual capability limits
   visible without turning today's implementation into a permanent prohibition.
3. Update playprey's delivery tool to capture working-tree content, including intended
   dotfiles, with truthful provenance. Remove dirty-tree refusal and the mandatory
   workaround ordering. Keep shared-storage and committed-build operations usable.
4. Update root/project local rules, templates, initializers, settings repositories
   and help that would otherwise regenerate deleted restrictions. Preserve
   historical reports as evidence; mark superseded operational instructions where
   they are still used as present guidance rather than rewriting historical events.
5. Inspect harness settings controlled by these projects and document supported
   autonomy settings. Distinguish editable project settings from restrictions of
   the external execution service/model. Do not claim an external restriction is
   removed without evidence; name its owner and unresolved effect explicitly.

Deliverable: `report4.md`, project-by-project rule dispositions and proof that a
new project and a new attempt receive the revised guidance.

## Step 5 — migrate existing state and deploy the complete change

1. Specify handling of old approval records, frozen definitions, pending questions,
   accepted revisions, queued integrations and saved restrictive configuration.
   Preserve historical actors and decisions as facts. Obsolete policy holds should
   no longer require new human answers; deliberate user holds remain distinguishable.
2. Update registered workflow definitions and existing run copies that still load
   old guidance/configuration. Record an explicit migration of frozen definitions
   where needed. Old runtime records must not restore removed enforcement after
   restart. Backward compatibility is not a requirement, but existing work needs a
   defined migration rather than accidental reinterpretation.
3. Commit/push source changes in dependency order, update parent gitlinks and deploy
   the corresponding WFE/executor/backend/project sources. Include release-script
   policy guards found in Step 1; installation must not itself retain an approver
   bottleneck. Record actual deployed versions and command resolution.
4. Verify service state through Nautobot/nctl, then the relevant version and health
   endpoints. Check the service process, installed CLI and a fresh agent attempt;
   source-tree tests alone cannot prove deployment.

Deliverable: `report5.md`, migration results and deployed revision evidence.
Do not restart the p6 runs stopped by the Developer merely to migrate them.

## Step 6 — prove the new behavior and close the episode

Use temporary projects, local fixtures and disposable infrastructure resources
where a test needs effects. The planning task itself runs none of these effects.

| Scenario | Evidence required |
| --- | --- |
| Ordinary work with no approvals | Create, execute, revise, report, publish and integrate a run without definition approval, plan agreement or result acceptance. |
| Changed plan and sources | Multiple plan edits, a changed batch and an updated backend revision execute without re-agreement; actual inputs are recorded. |
| Agent changes its approach | Adjust/reopen work, change bindings and delegate without a person-only permission step; CLI/API/UI agree. |
| p6 delivery and integration | Deliver intended dotfiles and working-tree edits, publish a file above the former size threshold, add a final report and integrate without another acceptance. |
| Infrastructure operations | A command outside the former playbook list and a disposable guest's lifecycle run through the agent-operated path with no human handoff. |
| Existing runs and restart | Migrated policy waits cease blocking; explicit user stops remain stopped; restart does not resurrect old enforcement. |
| Project-local instructions | Fresh cagent/playprey attempts and a newly generated project contain no deleted restrictions; other registered projects have recorded audit results. |
| Real errors remain truthful | Invalid operation arguments, unavailable tools, a transport failure and a genuine Git conflict produce accurate diagnostics and agent-usable recovery, not false success. |

1. Replace tests that assert removed policy refusals with successful-operation
   cases. Add coverage at the real boundaries affected; do not just test that a
   configuration boolean is enabled. Audit test helpers for old mandatory approvals.
2. Run WFE `npm run check`, the cagentctl and nctl pytest suites, relevant executor
   tests and the applicable cross-package checks documented by backend owners.
   Use browser/CLI integration checks for changed interactions. Any old check whose
   sole purpose was enforcing a deleted rule is changed or retired, not reinstated
   to make the suite green.
3. Revisit every restriction inventory entry. Report removed enforcement, retained
   autonomy guarantees, factual implementation necessities and external limitations
   separately. Account for indirect callers and generated copies; a text search
   with zero matches alone is not proof.
4. Write `report.md` with before/after behavior, p6 reproductions, remaining external
   limits, deployment/migration evidence and actual test results. Update maintained
   development docs and ignored environment notes. Commit/push intended changes and
   verify remote correspondence without including unrelated working-tree changes.

## Completion criteria

- WFE, nctl and cagentctl no longer impose development-process approval, role,
  allow-list, workflow-order or publication-policy vetoes covered by the braindump.
- Shared and project-local instructions no longer recreate those restrictions;
  retained rules affirm freedom, and useful tool descriptions remain discoverable.
- The p6 stalls have direct regression evidence, including the restrictions that
  earlier reports called meaningful rather than recommending removal.
- Existing work has a tested migration, and installed tools and fresh attempts
  use the changed behavior.
- Decisions and results remain truthful records without being permission gates.
- External restrictions that cannot be changed in these repositories are reported
  explicitly. They are not silently counted as removed or used to narrow the audit.
