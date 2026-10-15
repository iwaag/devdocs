# project_migration p5/pre1 — investigate and design the cagent migration

## Purpose and scope

Establish an evidence-backed design for migrating cagent into the managed-project/workflow system. This phase investigates the existing implementation, prepares concrete alternatives, discusses them with the Developer, and produces the agreed design and implementation plan. It does not implement or deploy the migration.

The source of intent is [the p5 braindump](../braindump.md). Its responsibilities are:

- receive installation/service reports and reflect them in cluster records;
- discover actual cluster state, including unreported and external changes;
- receive and record change requests, then carry out the requested changes;
- answer questions from managed state information.

Reporting is useful but not assumed complete. Providing the regular change route does not make cagent responsible for preventing every alternative route. Nautobot, Ansible and nctl are candidates for reuse, not mandatory parts of the destination design. Replacing the state platform may warrant a separate phase.

The preceding conversation's suggestions about provenance, desired state, execution recovery and explanation of unknowns are **design questions**, not accepted requirements. Workflow names, topology, approval rules, storage and tool choices remain open until the investigation and discussion below.

Follow `devpolicy/styles.md`, `devpolicy/terms.md`, `devdocs/README_DEV.md`, `pj-agdev/.local/devenv.md` and `localrule.md`. Favor Tool Giving and Evidence-Driven Guidance: distinguish requirements supported by actual failures from inherited restrictions and hypothetical concerns. Backward compatibility is not required by default; preserving useful records and unfinished work is a separate decision.

## Starting evidence and limits

This plan was prepared with a preliminary source read, not a completed implementation audit:

| Source (workspace-relative) | Starting observation to verify in detail |
| --- | --- |
| `pj-clusterintent/cagent/README.md`, `src/cagent_api/agent_runner.py` | HTTP entrances have different identities and tool sets; the authenticated runner and unauthenticated window are distinct paths. |
| `pj-clusterintent/cagent/src/cagent_api/topics_serve.py`, `role_run.py`, `change_record.py` | Topic handling uses front/operator handoffs. Recording a requested change is separate from executing it. Tool grants depend on the runner/harness; documentation alone is insufficient to establish effective authority. |
| `pj-clusterintent/nctl/README.md` | nctl owns more than a CLI facade: desired/actual reads, observation orchestration, drift, planning, execution and evidence all require a disposition. |
| `pj-agdev/agdevworld/workflow_editor/docs/{contract,runs,storage,integration,delegation,access}.md` | The destination already has managed projects, run copies, result decisions, Git integration, cross-project access and delegation. External machine changes must be mapped explicitly to this lifecycle. |
| `devdocs/episodes/project_migration/p3/`, `p4/` | Earlier migrations provide examples and acceptance evidence, not proof that cagent can use the same execution boundaries. |

Source anchors at preparation: `pj-clusterintent` at `ff2910da4cb44969bb17e08e8b27bfcbf2b7da1c`; `pj-agdev/agdevworld` at `47a365d0b2c2613325abc84f220849db113eb82b`. Step 1 records fresh revisions and distinguishes source checkout from deployed release.

Read-only `nctl status --json` and `nctl relations --json` queries were made while preparing this plan. They are preliminary inventory evidence, not a fresh end-to-end service verification. Raw relation results are retained in the ignored `pj-agdev/.local/project-migration-p5-pre1/relations.json`; host facts must not enter portable episode documents.

## Step 1 — trace the current implementation and operating responsibilities

Investigate the source and tests before choosing the new workflow shape.

1. Record applicable repository instructions, revisions, dirty work, deployed versions and relevant earlier contracts/trial reports. Identify which documentation describes retired behavior.
2. Trace each actual entrance separately: HTTP node/human/window, Zulip DM/topic, and direct operator tooling. Follow identity, request intake, role selection, tool grants, handoffs, persistence, response and recovery. Inspect `auth.py`, `server.py`, `worker.py`, `store.py`, `agent_runner.py`, `role_run.py`, `zulip_window.py`, `topics_serve.py`, `change_record.py`, `incident.py` and their tests as relevant.
3. Trace reporting and observation through nctl, nodeutils, nintent/Nautobot and Ansible. Determine what is detected today, where observations are ingested, how entities are matched, and how purpose/ownership information is stored. Distinguish installation reports from defect reports; an existing incident recorder does not establish an installation-report ingestion capability.
4. Trace change execution end to end: request record, desired-state edits, plans, SSH/Ansible actions, operation records, post-action observation and failure handling. Identify manual steps, missing connections and the actual actor that executes each operation. Do not infer an automatic executor from the existence of a request topic.
5. Build a responsibility map for cagent, nctl, Nautobot/nintent, nodeutils and Ansible. Include state models, jobs, inventory generation, SSH trust, credentials, operation history and deployment ownership. Record duplicated responsibilities and dependencies that would survive removal of a component.
6. Inventory active callers, schedules, service placements, open change requests and retained evidence. Use Nautobot or read-only nctl queries for registered local services, then targeted process/configuration reads where necessary. Keep inventory timestamps and freshness limitations; do not call a stale observation live confirmation.

Use existing evidence and bounded read-only probes. A scan that triggers collection/ingestion is not merely a database read. Do not execute a reconciliation, refresh job, deployment, credential change or service retirement as an incidental investigation step. If a specific uncertainty needs an operational trial, describe its purpose and effects in the discussion material first.

Deliverable: `report1.md`, containing a source-backed path/responsibility matrix, caller and data inventory, evidence of existing failures, and explicit unknowns. Each capability should distinguish documented, implemented, tested and currently observed behavior. Put machine names, addresses, credentials references and raw runtime evidence in ignored local notes; publish only portable conclusions.

## Step 2 — examine destination fit and prepare design alternatives

Inspect the installed WFE/executor contracts and relevant implementation in `pj-agdev/agdevworld` and `pj-agdev/agautolab`. Compare existing capabilities with Step 1 rather than recreating common infrastructure in cagent.

Cover these decisions in `options.md`:

| Decision | Questions and evidence needed |
| --- | --- |
| Responsibility and workflow boundaries | How do reporting, scanning, explanation, request registration and execution interact? Which need separate runs, shared context or an ongoing schedule? What work belongs to the requesting project? |
| State meaning and ownership | What is reported, observed or desired? Where do purpose, owner, provenance and observation time live? How are disagreement, duplicate reports, absence and failed observation represented? |
| Desired state versus one-time work | Which requests create an enduring maintenance responsibility? What should happen when later scans find drift: record, notify, propose or repair? Is continuous convergence actually wanted? |
| Identity and authority | Who can request which work, and where is that authority exercised? Inspect actual harness permissions, execution users, inherited environment and credential access. Do not equate repository bindings or guide text with host/process isolation. Separate user-facing secret management from credentials required to operate tools. |
| Run and operation lifecycle | How are a request, WFE run and external operation linked? Distinguish plan agreement, permission to perform effects, execution success, verification, result acceptance and Git integration. Result acceptance after an operation cannot authorize it retroactively, and a Git rollback cannot undo a machine change. |
| Failure and concurrency | What happens after partial success, connection loss, an executor restart, duplicate delivery, concurrent work on one target, cancellation or changed prerequisites? When can evidence support retry, and when must the result remain unknown? |
| Bootstrap and self-management | What can cagent do when WFE, its execution host, Nautobot or transport is unavailable? Who can finish or recover an operation that disrupts cagent's own dependencies? |
| Stack disposition | What responsibilities justify keeping each of Nautobot/nintent, nctl, nodeutils and Ansible? What would replacement cost and where would those responsibilities move? Which decisions can be deferred without blocking migration? |
| Migration and retirement | Which callers, open requests, historical records and schedules must move? How can old and new paths coexist briefly without duplicate execution? What evidence permits retirement and what is the recovery route? |

For each consequential choice, provide the smallest viable proposal, a credible alternative, evidence, tradeoffs and a recommendation. Mark assumptions and their consequences. Compare at least reuse of the current backend with replacement of selected responsibilities; do not expand this into an unrelated product-selection exercise.

Include a capability/gap matrix identifying cagent-specific work, reusable destination facilities, missing common capabilities and work better deferred. Prepare representative request examples and paper walkthroughs so the Developer can review behavior, not only abstractions. Keep any proposed tool restriction or approval boundary tied to an explicit responsibility, demonstrated failure or stated human preference.

Deliverable: `options.md`, including the proposed boundaries, storage ownership, alternatives, recommendations and prioritized questions for the Developer.

## Step 3 — discuss the design with the Developer

Present the investigation and a concrete recommended design together. The Developer should not have to investigate code or invent an architecture from a list of unanswered questions.

Discuss in small, dependent groups:

1. **Purpose and responsibility:** what cagent manages, what it merely observes, reporting expectations, desired-state obligations and autonomous repair policy.
2. **Operating model:** workflow boundaries, state semantics, execution authority, external effects, recovery, concurrency and self-management.
3. **Migration scope:** backend reuse/replacement, common WFE changes, retained information, staged rollout and future phases.

Record each decision in `decisions.md`: question, evidence, alternatives, recommendation, Developer response/reference, chosen outcome and consequences. Distinguish agreed, provisional, deferred and unresolved items. An assistant recommendation, silence or elapsed time is not agreement. Continue independent research while questions are pending; reopen a decision if new evidence materially changes its basis.

Use the ordinary human conversation for these discussions; this plan does not require creating an in-system discussion or posting messages to other agents. If the Developer needs to verify a prototype, prepare it and its inspection path before asking, under the separately agreed trial scope.

Deliverable: `decisions.md` and an updated proposal reflecting actual discussion. Resolve decisions that determine authority, storage ownership and execution lifecycle before treating the design as ready for implementation.

## Step 4 — write and review the agreed design

Write `design.md` from the decisions, with concrete responsibilities, data ownership, interfaces, workflow/run boundaries, effective tool access, operational records and recovery behavior. State which existing components remain, which change, and which decisions belong to a later phase.

Walk the design through at least these cases, combining cases where useful:

- a reported installation with purpose information, duplicate delivery, and later conflicting observation;
- an unreported service, an unreachable node and a stale record used to answer a question;
- a one-time operation and a request to maintain a lasting state;
- a request from another managed project, including questions and result return;
- concurrent changes to the same target and a changed precondition before execution;
- a completed external action whose response is lost, partial success, restart and cancellation;
- work affecting cagent/WFE or their state/transport dependencies;
- cutover with an open legacy request and later discovery of a missing caller.

For each case state who acts, what is read/written, what constitutes sufficient authority, what proves the result, and what remains uncertain after failure. Identify which cases can be checked from code/tests and which need bounded trials in the implementation phase. Paper walkthroughs are design validation, not operational proof.

Review the complete draft with the Developer, including any consequence not already covered by Step 3. Update the decisions and design together; do not silently turn an unresolved choice into an implementation detail.

Deliverable: `design.md`, scenario review notes, and explicit agreement or a precise list of remaining decisions.

## Step 5 — hand off an executable migration plan

Prepare `../plan.md` from the agreed design. Separate common-system changes, cagent project/workflow construction, backend changes if any, trial preparation, deployment, operational proof and legacy retirement. For each step name affected repositories, dependencies, deliverables and acceptance evidence. Sequence any backend replacement as its own phase when agreed; do not couple it to migration merely because both are possible.

Specify realistic trial targets, operation ownership, recovery/cleanup and evidence for external effects. Define how the human reviews prepared results with minimal manual work. Include LAN access for any services introduced, fresh Nautobot/nctl checks for operational decisions, and deployment before requesting human behavior verification where applicable.

Write `report.md` summarizing findings, decisions, unresolved/deferred work and the implementation handoff. Review all authored documents for consistency, unsupported claims and local environment details. Commit and push the intended documentation changes in their owning repository, preserving unrelated work. Plan publication does not itself execute the migration.

Deliverables: agreed design, `../plan.md`, and `report.md`.

## Completion criteria

- Current entrances, state paths, execution paths and dependencies have a source/evidence-backed inventory; unknowns are visible.
- The design assigns state ownership and operational responsibility without assuming complete reporting or global prevention of out-of-band changes.
- External effects and their recovery are explicitly related to WFE's run, acceptance and integration lifecycle.
- Stack choices and migration boundaries are justified by responsibilities and discussed with the Developer.
- Consequential decisions have actual human discussion records; deferred decisions have a destination and do not conceal implementation blockers.
- The agreed design and staged implementation plan are reviewable, committed and pushed. No migration or operational success is claimed from this design phase alone.
