# project_migration p8 — Front as a managed project

## Objective

Implement [braindump.md](braindump.md) and the subsequent discussion: Front is the Developer's conversational entrance to the managed system. It understands requests, carries work through to usable results, coordinates projects, creates missing projects when useful, and performs the operations available through agdevworld. The Developer normally expresses a desire to Front and uses agdevworld when detailed editing or direct inspection of results is useful.

Implement Front as an ordinary managed project, executed by the existing WFE workers. Its guides, workflows, run history and durable knowledge use the common project facilities. autolab remains the execution and transport infrastructure. Front's broad operator authority does not require a separate execution engine or a special project type.

This document is an implementation plan, not an implementation or deployment report.

## Working policy

Follow `devpolicy/styles.md`, `devpolicy/terms.md`, `devdocs/README_DEV.md`, applicable repository instructions, ignored environment notes and `localrule.md`.

- Generated files are English; communication with the Developer is Japanese.
- Backward compatibility is not required. Migrate useful data and remove superseded active behavior instead of maintaining two Front implementations.
- Agents perform implementation, deployment, operational preparation and verification. Routine choices do not require another approval. Approvals and result decisions are records, not process gates. Omni has standing authority to answer in-system operational permission requests on the Developer's behalf, with the actual actor recorded.
- Deploy the completed experience and prepare direct review entry points before requesting experiential feedback. Services and review surfaces are accessible from the home LAN.
- Consult Nautobot or nctl for service placement and state before operational work. Distinguish recorded observation ages from fresh verification.
- Keep physical host names, addresses, credentials, machine-specific paths and raw operational evidence in ignored local files. Tracked documentation uses portable project and run references.
- Preserve unrelated working changes. Commit/push intended changes and update relevant submodule pins as appropriate without adding a human approval checkpoint.

## Architecture and responsibilities

| Owner | Responsibility |
| --- | --- |
| Managed Front project | Understand the request, choose useful work, answer from existing evidence, delegate, judge results, handle questions, coordinate projects, remember preferences and explain outcomes |
| Shared conversation facilities | Durable conversation identity, incoming message receipts, links to Front runs, delivery of replies and continuity across restarts |
| WFE | Projects, workflow definitions, run state, revisions, delegation, context, operator operations, artifacts and recorded effects of its operations |
| autolab and workers | Execute attempts, transport coordination events and release execution resources while work is waiting |
| Other managed projects | Their own specialist work, deliverables, operational effects and project knowledge |
| agdevworld | The conversational surface and direct access to the same operations and records for detailed use |

The conversation facilities route and persist messages; Front decides their meaning. Do not move intent interpretation into a fixed keyword router or introduce a second ledger of project execution state.

Front can operate the system with the Developer's authority. Express that authority through the execution principal and common operation interfaces rather than scattered `project == front` exceptions. Record Front as the actor and the Developer as the person served; do not represent Front's judgment as words spoken directly by the Developer. Do not build a general permission administration product in this episode.

## Behavioral design

### Requests and work

Front may answer immediately from evidence, continue an existing run, start new work, revise a run's work, delegate, or establish a project. Not every message requires a new workflow definition or a long-running task.

Start with a small ordinary workflow based on the existing freework pattern. Use talk, do and delegate nodes as useful, with run adjustments for work discovered during the conversation. Persist reusable workflows when a recurring need justifies them. Do not require a fixed proposal/approval/delegation sequence.

Front remains responsible for returning the result to the Developer. Posting a request or creating a project is an intermediate outcome unless that was the whole request. Specialist agents retain autonomy over how they perform their work. Front answers their questions when the request and known preferences suffice and asks the Developer when a material choice remains unresolved.

### Conversations and runs

A continuing conversation can refer to multiple bounded Front runs. A run represents a coherent piece of work with a meaningful completion state; the entire relationship with the Developer is not one permanent run.

Maintain durable links among the original message, Front's interpretation, the selected or created run, subsequent steering, delegated runs and delivered replies. Preserve the original words separately from rewritten requests. A clarification about completed work can cite its run without silently reopening execution.

Let Front resolve conversational references using the visible history and linked work. Ask a concise clarification when the target genuinely remains ambiguous. Preserve the unresolved message so it cannot silently modify the wrong run. Avoid asking the Developer to learn run IDs for ordinary conversation.

New messages remain serviceable while earlier work is delegated or queued. Waiting must not occupy an execution slot. Inspect the current scheduler before choosing a bounded solution for conversational responsiveness; do not assume releasing a slot alone prevents a long child attempt from blocking the next conversation turn.

### Knowledge

| Information | Durable home and use |
| --- | --- |
| General preferences | Appropriately scoped system context blocks, available when projects are used directly |
| Project-specific preferences | That project's context blocks |
| Lessons from coordination | Front's versioned knowledge, with evidence and useful summaries delivered to affected projects |
| Active work and results | Authoritative run records, read through common project/run tools |
| Conversation continuity | Durable conversation records and links; summaries are derived aids, not replacements for source messages |

Distinguish explicit enduring preferences, Front's tentative inference and instructions for this request only. Keep evidence and correction history; do not turn every incidental instruction into a system-wide rule. Reuse context blocks and on-demand details instead of injecting all conversation history into every attempt. Context updates affect future attempts; use ordinary run instructions to steer current work.

### Operations

UI and Front use the same underlying operations and state transitions. Expose discoverable CLI/API capabilities where UI functionality is missing from agent tools. Front should not have to simulate browser clicks for routine system administration.

The coverage audit includes discovery and reads, project creation/configuration, workflow creation/editing, requests, instructions, questions/answers, run revision and lifecycle, delegation, context, execution settings, result review, publication/integration and artifact delivery. Include additional meaningful UI operations found during inspection. Explain any genuine unsupported operation explicitly; do not silently reduce the braindump's goal to delegation alone.

## Step 1 — inspect the current system and record the gaps

1. Read current instructions, source and deployed revisions, and relevant migration reports. Check working changes and use Nautobot/nctl to inspect Front, WFE, workers and their dependencies. Record observation ages and confirm uncertain runtime facts as needed.
2. Trace the current human conversation path through agdevworld, the relay, legacy Front, Zulip and callbacks. Identify retained user-facing behavior and the actual authority for each record.
3. Audit legacy Front responsibilities beyond direct chat: routines, argues, presentation/rendering, settings, memories and outstanding requests. Assign each an explicit migration destination or a documented retirement outcome; stopping the old listener must not silently strand another feature.
4. Trace common project creation, freework, run adjustment, dynamic delegation, context, project access and operator APIs. Check implementation and focused tests where older documentation conflicts with newer contracts.
5. Produce a UI-to-tool operation matrix and identify missing capabilities. Separate shared infrastructure work from Front's ordinary project content.
6. Specify conversation/run association, actor identity, operation authority, reply delivery and responsiveness. Decide how short conversational work is scheduled while specialist work runs. Use the existing worker architecture; extend only what the measured scenario needs.
7. Select two real managed projects and a bounded useful request whose combined result can be inspected. Include one missing-project scenario separately so creation is exercised without making every request create a project.

Deliverable: `report1.md` with current facts, operation coverage, ownership decisions, retained behavior, migration inventory and trial scope. Resolve ordinary design choices without an approval checkpoint.

## Step 2 — complete common operator capabilities

1. Implement missing CLI/API operations from the coverage matrix using the same domain operations as the UI. Give tools sufficient help, examples, target identity and actionable outcomes.
2. Wire Front's execution principal to the required operator capabilities. Keep credentials and host overlays outside project Git history and run-published files. Reuse existing identity/configuration machinery rather than introducing a parallel authorization framework.
3. Ensure creation and writes return durable project/run/operation identities. Replayed transport messages and retried operations must not create duplicate projects, runs or external effects.
4. Preserve conflict detection for concurrent workflow/context changes. A stale edit reports current state so Front can reassess; it must not overwrite an unrelated UI edit.
5. Make existing runs, outputs and source references readable through common tools, including when execution occurs on another worker. Preserve live versus published-state distinctions.
6. Verify the lifecycle of delegate nodes added by run adjustment, including target resolution, questions, result decisions and continuation. Fix actual gaps in the shared contract.

Deliverable: `report2.md`, the completed operation matrix and focused tests for newly exposed mutations, retry identities, stale writes and dynamic delegation. Use existing coverage for unchanged operations.

## Step 3 — implement shared conversation continuity

1. Connect the existing human-facing conversation surface to managed Front execution. Retain a single understandable Front entrance; internal Zulip channels remain transport and evidence rather than a requirement for the human to navigate.
2. Persist message receipts, original text, conversation identity, run associations and outgoing delivery records. Recover after restart without losing steering or delivering the same completion repeatedly.
3. Give Front enough history and linked-run evidence to distinguish a new request, a follow-up, a correction and an informational question. Record its chosen association and allow correction without rewriting the original message.
4. Support multiple active Front runs in the same conversation. Replies identify the relevant work naturally and include direct review links when useful. Concurrent completions must not be attributed to the last message merely because it is newest.
5. Carry human steering into the correct Front run and, when needed, its delegated child through existing instruction operations. Explain the actual scope and outcome of hold, stop and cancel actions: stopping a parent does not implicitly stop its children under the current delegation contract.
6. Release execution slots during waits and implement the bounded scheduling decision from Step 1. Verify a second conversation request receives a response while the first request's specialist work remains active.

Deliverable: `report3.md` and focused tests for message replay, restart recovery, concurrent work, ambiguous references, steering and reply routing. Test race conditions with fixtures before trying them on unrelated live work.

## Step 4 — establish the managed Front project

1. Create the managed project through the common creation path. Add its intent, project guide, small conversational workflow and versioned knowledge structure. Use ordinary profiles, workers, run copies and integration.
2. Rewrite the role around the new responsibility. Remove obsolete assumptions that only a human can create projects, that Front cannot read project records, or that every action requires a proposal and permission. Keep evidence-backed lessons that still apply, with their trial references.
3. Provide discoverable tools for system operations and specialist capabilities. Keep tool usage in tool help and shared facts in shared documentation; the guide describes Front's role and available evidence rather than prescribing a route for every request.
4. Implement preference and coordination-knowledge maintenance using existing context and repository facilities. Handle simultaneous runs updating knowledge through normal conflict/integration behavior rather than shared unrecorded writes.
5. Exercise immediate answers, existing-run steering, one-off work, delegation and new project creation. Make Front check whether an existing project serves the need before creating another.
6. Present actual delivered outcomes and outstanding limitations. Include usable artifacts or direct review entry points; do not call dispatch, setup or a child's claim of success the completed user outcome without examining the relevant evidence.

Deliverable: `report4.md` and an ordinary managed Front project usable through the conversation surface. No separate Front executor or special project schema is required.

## Step 5 — deploy and migrate

1. Back up affected conversation, delivery, configuration and knowledge stores. Record the deployed revisions and a coherent recovery procedure for code and data together.
2. Deploy common infrastructure and the managed Front project. Verify LAN access, execution identity, tool discovery and required project/artifact access.
3. Migrate useful preferences, knowledge and open conversation/work links from the Step 1 inventory. Keep historical records readable without continuing the legacy execution model. Do not restart held or completed work merely to migrate it.
4. Cut over between active servings. Ensure only one consumer owns each incoming Front message and each callback during the switch; reconcile messages received around the boundary through durable receipts.
5. Migrate or retire the other legacy responsibilities according to their recorded destinations. Remove superseded listeners, routing and guide/tool assumptions after verifying their replacements. Retained presentation or transport functionality belongs to the appropriate common service.
6. Update desired placement and observe runtime state through Nautobot/nctl. Verify the actual deployed behavior, not just source tests or a converged placement record.

Deliverable: `report5.md` with migration counts/outcomes, deployed revisions, observations, retained historical access and recovery instructions in ignored local notes where machine details are needed.

## Step 6 — prove the conversational experience

Run the following through the deployed entrance. Agents prepare inputs, answer routine operational questions and inspect results themselves.

| Trial | Evidence required |
| --- | --- |
| Existing information | Front answers a useful question from project/run evidence without unnecessary new specialist work |
| Two-project request | One human request leads to ordinary delegation to two projects, useful coordination and a combined delivered result |
| Specialist question | Front answers a question supported by the request/preferences and continues the work without forwarding routine choices to the human |
| Material ambiguity | Front asks a necessary concise question and applies the answer to the intended work |
| Mid-flight change | An additional human instruction reaches the correct active work and changes the result, with original and revised intent preserved |
| Conversation responsiveness | A second request is served while the first request's child is still active; neither reply is misattributed |
| New project | Front creates a useful managed project, establishes its work and obtains a result through ordinary facilities |
| Direct operator work | Front performs representative workflow, context and run-management operations and the UI shows the same result |
| Knowledge reuse | A subsequent request benefits from recorded preferences/coordination lessons; a project used directly receives its applicable context |
| Recovery | A bounded restart or replay recovers links and delivery without duplicate project creation, work dispatch or completion messages |
| Delivery | The Developer can inspect the actual combined output and relevant detail from prepared LAN review links with minimal actions |

Use private fixtures for hazardous timing cases; choose bounded live trials that do not interrupt unrelated work. Record run IDs, operation receipts, result/artifact identities, response timing, context revisions and actual interventions. When Omni performs work belonging to an in-system agent, record the required one-line handoff note. Such intervention diagnoses a gap; it does not prove Front performed the action autonomously.

Fix demonstrated gaps and repeat affected checks. Do not add blanket rules, forced workflows or additional permission prompts in response to hypothetical failures. Run repository-required checks for touched components; distinguish unrelated baseline failures from regressions.

Deliverable: `report6.md` with end-to-end evidence, meaningful failures and fixes, measured responsiveness and remaining limitations. Prepare the live review experience before requesting optional human feedback; report pending feedback separately from technical verification.

## Step 7 — document and close

1. Update shared contracts, CLI help, Front's guide, `devdocs/README_DEV.md` and ignored environment notes to describe the deployed ownership and conversation/run model. Remove obsolete active guidance.
2. Review the final UI-to-tool matrix. Resolve remaining gaps within the promised scope or explicitly identify a material unmet requirement; do not declare full UI substitution on the strength of a few demonstrations.
3. Review intended diffs, run appropriate checks, commit/push changes and update parent pins where applicable. Keep unrelated changes and machine details out of the commits.
4. Write `report.md` with delivered behavior, migration/deployment outcome, trial evidence, review entry points, interventions and named limitations. State which completion criteria are met and which, if any, remain open.

## Completion criteria

- Front is an ordinary managed project on common WFE execution, with no separate agent runtime or hardcoded Front project type.
- The Developer can express a request, steer it and receive a usable result through one continuing conversational entrance without navigating internal Zulip coordination.
- Bounded Front runs, their delegated work and concurrent conversation requests retain unambiguous identities across restart and replay.
- Front can discover and perform the meaningful agdevworld operations through common tools, including project creation, workflow management, run steering and context maintenance.
- Broad operator authority is configured for the execution principal, with truthful actor/provenance records and no new human approval gates.
- Conversation remains usable while specialist work executes, supported by measured evidence rather than an assumption about queue behavior.
- Preferences and coordination lessons persist and reach their appropriate scopes; transient run state remains authoritative in run records.
- Real two-project work, mid-flight steering, project creation, knowledge reuse and delivery succeed through the deployed system without Omni silently completing Front's responsibilities.
- Legacy Front responsibilities and outstanding work have explicit destinations; superseded active paths are retired without stranded callbacks or duplicate consumers.
- The result is deployed, LAN-accessible and prepared for low-effort human inspection, with checks, evidence and documentation supporting the claims.
