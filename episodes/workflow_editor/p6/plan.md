# Workflow editor p6 — a project-based PPT agent driven through Zulip

## Goal

Implement [braindump.md](braindump.md) by introducing a new `pj-ppt` project whose workflows can be requested and completed through Zulip. Complete one real `study` run and one real `create` run, including planning, questions, answers, and human result confirmation, without requiring the person to use the agdevworld UI for normal progress.

Use the existing project/workflow execution model as the authority. Zulip is its communication interface: a request becomes a braindump, discussion becomes recorded workflow input, and execution proceeds through the existing run operations. Devdocs remains the authoritative work record; Zulip carries concise coordination and links to records and artifacts.

## Compatibility and scope

**This is an experimental phase. Breaking changes are allowed; backward compatibility is not required.** Prefer a clear new contract over compatibility adapters, dual schemas, legacy command aliases, or automatic migration of obsolete experimental records. Change schemas, APIs, configuration, and guides directly when needed. If existing experimental data prevents progress, an explicit reset or replacement is acceptable; document what is affected. Do not spend this phase building a general migration framework.

This does not expand p6 into converting existing agents. Enable the new inbound workflow interface for `pj-ppt` first. Leave the legacy agents on their existing execution paths; changing shared infrastructure must not accidentally route their conversations into the new receiver. Preserve unrelated user changes and useful evidence. Before deploying a breaking contract, identify affected processes and active runs and coordinate their cutover explicitly.

Included:

- A new project-based PPT agent with `study` and `create` workflows.
- Workflow-scoped execution profiles, additional guides, and Zulip intake configuration.
- Durable inbound messages, request/run bindings, question answers, and additional instructions.
- Additional-instruction handling at node boundaries, with no interruption of running work.
- Request-topic conversation, run-topic progress, and accessible deliverables.
- Isolated failure-path checks and two human-led end-to-end runs.

Deferred:

- Migration of existing agents, legacy missions, tasks, or conversations.
- Cross-agent delegation and concurrent agent execution.
- Immediate interruption and cleanup triggered by a Zulip message.
- Dynamic graph rewriting, automatic rollback of completed nodes, and automatic reopening of completed runs.
- Multiple runs in one request topic and interpretation of edits to previously ingested messages.
- A general-purpose agent framework or support for multiple PPT generation backends.

## Step 1 — define the inbound and conversation contracts

Inspect the current code and installed releases before implementation. Use the existing run reducer, request receipts, execution queue, answer/resume operations, publication checkpoints, and notification delivery protocol. Previous reports describe prior evidence, not verification of this change.

Define the following rules before adding the receiver:

1. A configured Zulip workflow channel maps explicitly to a project and workflow. Routing uses the recorded realm/channel identity, not a profile inferred from an arbitrary topic prefix.
2. Within that channel, a `request-…` topic represents one request and one run. The first eligible post by an authorized requester becomes the immutable original braindump. Later posts are answers or additional instructions.
3. Record the original text, sender identity, message ID, time, and source location. Do not replace the braindump with an agent summary. Corrections arrive as new posts; message edits do not retroactively change accepted input in p6.
4. Ignore bot-generated posts as execution input. Posts in `run-…` topics do not answer questions, start runs, or resume work.
5. Questions sent to the request topic identify their question ID and the action expected from the person. Bind an answer to a specific question using the reply/reference or an unambiguous current pending question. Persist that binding when classifying the message. Do not later reinterpret an old message as an answer to a newer question.
6. Ambiguous replies require clarification. Late replies to closed questions remain recorded and receive an explanation; they are not silently applied elsewhere.
7. After completion, a new request needs a new request topic. A further post does not silently create another run or reopen the old one.

Separate message receipt, application to the run, and agent acknowledgment. Specify the transition rules and their durable evidence. A transport receipt does not mean the agent has read or acted on an instruction.

Define topic rename/move handling explicitly: retain the recorded binding when identity can be established, otherwise report a routing problem. Do not guess a different project or run from a matching topic name.

Deliverable: implemented contract types and focused tests for routing, classification, identity, and state transitions.

## Step 2 — create pj-ppt and its two workflows

Create and register `pj-ppt` through the existing project/repository mechanisms. Use directory-mode devdocs unless a concrete requirement calls for a separate repository.

| Resource | Purpose |
| --- | --- |
| `pj-ppt` | Project definition, workflow definitions, and agent configuration |
| `study-ppt` | Existing knowledge, sources, techniques, and tool comparisons learned through research |
| `wedo-ppt` | Locally exercised examples, reproducible procedures, failures, and improvements |
| `devdocs/runs/<workflow>/<run>/` | Original request, plan, dialogue evidence, node reports, and final report |

Register `study-ppt` and `wedo-ppt` as shared repositories using the established binding mechanism. Declare their read/write scopes in each workflow. Keep generated artifacts identifiable by run, and record reusable examples and their reproduction instructions in `wedo-ppt`.

Both workflows use `documentation.policy: default`. Reuse the generated planning talk, per-node reports, and final report; do not author duplicates.

| Workflow | Authored nodes |
| --- | --- |
| `study` | Research → explain findings and ask permission to finish (`talk`) |
| `create` | Create slides → verify content and rendered appearance → present artifacts and ask for result confirmation (`talk`) |

The study workflow researches existing knowledge; it does not need to create a presentation. Its findings should select a practical generation and rendering approach for the real create trial. Use one approach in p6 rather than building a pluggable generation system.

The create workflow produces a PPTX, a reviewable preview, and sufficient source/configuration or instructions to reproduce the result. Verify the rendered slides, including obvious clipping, overflow, missing content, and layout defects. Do not claim PowerPoint-specific validation unless PowerPoint was actually used.

Deliverable: registered project and repositories, valid workflow definitions, and declared artifact locations.

## Step 3 — make workflow execution settings project-based

Add the minimum declarative configuration needed to select an execution profile and additional guide for each workflow. Keep credentials in host configuration, outside managed repositories.

The current `agautolab.wfexec` implementation and `wfrun` guide contain autolab-specific assumptions. Make the necessary parts configurable while retaining the existing queue, attempt fencing, process lifecycle, publication, and backend recording. Reusing the implementation does not make the PPT agent a legacy autolab mission.

Compose the execution instructions from the common workflow contract, the project's/workflow's additional guide, and the run brief. The guide must explain:

- The original braindump and later input are separate records.
- Questions and answers are recorded through workflow operations; Zulip is their delivery interface.
- Every node boundary requires checking pending instructions.
- Receiving, taking up, and acting on input are distinct steps.
- Planning agreement, permission to finish, execution completion, and result acceptance have distinct meanings.
- Publication remains the execution host's responsibility.

Record the effective execution settings or their versioned references with the attempt so its behavior can be explained later. Do not add a compatibility layer for the old hardcoded configuration.

Deliverable: the new workflows execute with their intended identity, profile, and guide through the existing execution host.

## Step 4 — implement durable Zulip intake

Add a receiver that reuses the existing Zulip client but does not invoke the legacy mission/task listener. Keep routing and run-state decisions in the workflow editor; the Python integration handles transport.

Persist incoming messages before applying them. Scope deduplication by realm and message ID. Store request/run bindings and processing state durably, with a visible failure reason and retry path.

Connect intake to existing operations:

- First request: create a run with the original braindump and enqueue execution using a stable receipt derived from the source message.
- Answer to an open question: record the answer with the real sender and source, then use the existing answer/resume mechanism.
- Other input for an active run: append an additional instruction without interrupting its process.
- Input for a completed run: retain the receipt and explain that a new request topic is required.

Account for crashes between inbox persistence, run mutation, queueing, and completion of the inbox record. The run operation must be idempotent by source receipt, or carry equivalent durable evidence that permits reconciliation. An inbox flag alone cannot prevent duplication after a successful run mutation followed by a crash.

Recover messages missed during receiver downtime with a durable cursor and history reconciliation. Advance the cursor only after durable receipt. A network or authorization failure must remain visible and retryable; it must not drop input or rerun work.

Keep intake and notification delivery available while an agent runs or waits for the person. Initially enable intake only for the registered `pj-ppt` workflow channels and configured requester identities.

Deliverable: repeated delivery and receiver restart do not create duplicate runs, answers, or instructions.

## Step 5 — consume additional instructions at node boundaries

Implement additional instructions as run history, with source identity and explicit acknowledgment by the executing attempt. Include pending instructions in run inspection and execution briefs.

The execution host can run several nodes in one attempt. Checking messages only when a process starts is therefore insufficient. Add a server-enforced boundary protocol:

1. While a node is running, store incoming instructions without changing that node's execution or injecting a mid-node response requirement.
2. At the next boundary, expose all pending instructions to the agent in a stable order.
3. The agent records that it took them up and records its intended response: incorporate them into upcoming work, ask for clarification, or explain why the request requires a new run.
4. Refuse the next node's start while received instructions remain unacknowledged. Return actionable guidance so the agent can read and acknowledge them within the current attempt.
5. Guard final completion as well, so the last node cannot bypass pending input merely because no successor starts.

Use the run's serialization/locking rules for receipt, acknowledgment, node start, and final completion. Define the race boundary precisely: input durably applied before the transition must be considered; input applied after it belongs to the next boundary or the completed-run handling. Do not promise that a message still in transit has already reached the run.

Apply the boundary contract consistently to generated documentation steps as well as authored nodes. Do not require an agent to abandon already finished work or leave an otherwise healthy attempt marked as interrupted merely to check its inbox.

If an instruction requires changing completed work or the frozen graph, ask the person how to proceed. Do not silently reopen completed nodes. Preserve the original braindump and record any agreed changes separately. A changed plan requiring renewed agreement must use the existing document-evidence contract.

Deliverable: an instruction arriving during one node is considered before the next node starts, and an instruction arriving before final completion cannot be silently lost.

## Step 6 — deliver conversations and results to the right topics

Extend the current durable notifier, including its frozen deliveries and uncertain-send reconciliation, rather than adding an independent posting path.

| Destination | Content |
| --- | --- |
| Request topic | Request acknowledgment, planning questions, clarification, result presentation, acceptance, and the person's next action |
| Run topic | Concise execution progress, node reports, failures, publication status, and links to detailed records |

Persist both destinations and their relationship to the run. Link the request and run topics. Ensure delivery identities distinguish destination and purpose when one run event creates several messages. Freeze the destination with each delivery and include it in reconciliation evidence.

Replace the old instruction to answer exclusively in agdevworld for runs bound to a request topic. Keep UI/CLI operations available for inspection and recovery.

For final human confirmation, define an explicit mapping from the designated result-confirmation talk to the existing result decision. Record the answer, the confirmed artifact revision, and the decision evidence. A planning answer or a generated final report is never sufficient evidence of result acceptance. Ambiguous approval requires a follow-up question. Keep execution completion separately visible, even when result acceptance has already been recorded.

Publish the PPTX and preview through an existing reachable artifact path, or add the smallest necessary delivery mechanism. Verify that the requester can actually retrieve them from Zulip. A machine-local path or a link to unpublished content does not satisfy delivery. Use commit-addressed references where applicable and clearly report publication failures.

Deliverable: normal conversation and result review can take place entirely in the request topic while detailed work remains discoverable through the run topic and devdocs.

## Step 7 — verify failure paths and deploy

Use isolated repositories, registries, and fake transport for deterministic checks. If a live rehearsal is needed, use a dedicated test destination rather than real project topics.

Required checks:

1. Only configured channels, request topics, and authorized senders initiate work; bot posts and run-topic posts do not.
2. Duplicate receipt, restart after run creation, and restart after answer application preserve exactly one corresponding run operation.
3. Downtime recovery retrieves missed messages without reapplying previously handled ones.
4. Answers preserve sender and question identity; ambiguous and stale replies cannot approve another question or plan revision.
5. An answer arriving before the asking attempt exits resumes correctly through the existing execution lifecycle.
6. Additional input during a node does not interrupt it, but blocks the next start until taken up. Test the final-completion boundary and races with input receipt.
7. Acknowledgment survives restart; a stale attempt cannot acknowledge input or change execution state.
8. Zulip outages, uncertain sends, and retry retain the existing notification guarantees across both destinations.
9. Planning agreement cannot become result acceptance. Final approval records the correct result evidence without requiring duplicate human approval.
10. Published presentation and preview links are usable by the requester.

Run the affected editor and integration tests, required type/build checks, and relevant UI checks if the run presentation changes. Update schema/configuration documentation directly for the new contract. Check active processes before deploying through the established system lifecycle, and record any reset required by breaking changes.

Deliverable: tested and deployed implementation, with exact verification evidence and known limitations in the step report.

## Step 8 — complete the two real runs

First, the person starts a `study` request in Zulip. The agent writes a plan, obtains agreement there, researches PPT creation, records useful knowledge in `study-ppt`, explains its findings, obtains permission to finish, and completes the run and final report.

Second, the person starts a `create` request in Zulip. The agent agrees the intended audience, purpose, scope, and verification in the plan, creates and checks the presentation, records exercised procedures and lessons in `wedo-ppt`, and presents the PPTX and preview. The person reviews the result and provides final confirmation through Zulip.

During at least one run, exercise an additional instruction sent while a node is running and verify its boundary handling. Do not fabricate human agreement or count synthetic answers as the real trial.

Record the run IDs, request and run topics, published documents, artifact revisions, actual verification, and human decisions. Distinguish execution completion from any unresolved acceptance or publication issue.

P6 is complete when:

- One study run and one create run have completed through Zulip, including their human consultations.
- The study knowledge and exercised creation knowledge are stored in their intended repositories.
- The requester can retrieve the created PPTX and inspect its preview.
- Devdocs preserves the original requests, plans, input/decision evidence, node reports, and final reports.
- The new path works without migrating existing agents or invoking their legacy listeners.

## Reporting

Write `report1.md` through `report8.md` for the corresponding steps, recording changes, checks, evidence, limitations, and the next entry point. Adjust implementation details when evidence requires it and explain material departures from this plan. Finish with `report.md` summarizing the delivered behavior and both real runs.

Commit and push changes according to the repository's local rules. Keep unrelated edits out of implementation and documentation commits.
