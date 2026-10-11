# Workflow editor p5 — plans, reports, and Zulip coordination

## Goal and scope

Implement [braindump.md](braindump.md) and the implementation proposal agreed upon in the discussion.

Each default-policy run preserves the person's input, an agreed plan, per-node reports, and a final report in devdocs. Zulip carries concise coordination updates that tell a person or another agent what happened, what is uncertain, and who should act next. Readers can move from Zulip to devdocs and then to actual source code or generated artifacts when more evidence is needed.

The proposed 7:2:1 reading frequency is a usability goal, not a posting quota or a measurable completion threshold for one trial. Assess whether routine decisions can be made from the summary while detailed evidence remains easy to find.

P5 includes:

- A workflow documentation policy with `default` and `manual` values.
- Generated plan/report steps, persistent execution state, and their browser presentation.
- Agent instructions for writing the documents and short coordination summaries.
- Project folders, workflow channels, and run topics in Zulip.
- Outgoing notifications with durable delivery tracking and retry.
- Validation through isolated tests and a real RTS run.

P5 does not include inbound Zulip answers or commands, multi-agent execution, delegate execution, manual plan/report node authoring, legacy mission/task migration, or a new AI service for document generation or summarization. Human answers remain in agdevworld. Manual policy leaves document creation to explicit instructions.

The editor remains agdevworld system code, following [p4/ex1](../p4/ex1/plan.md). Develop and deploy it through the system lifecycle; do not restore an editor project or self-development workflow to the ordinary catalog.

## 1. Starting point and implementation ownership

The current implementation provides:

- `agdevworld/workflow_editor/shared/model.ts`, YAML parsing/rendering, validation, and semantic approval digests for definitions.
- `shared/run.ts`, a shared reducer with graph readiness, history replay, questions, answers, attempts, and holds.
- `server/runs.ts` and CLI/API operations with cross-process locking.
- `server/exec.ts`, local execution management and publication checkpoints.
- `src/canvas.ts` and the definition/run views for graph display.
- `agautolab.wfexec` and `agent/guides/wfrun/guide.md` for agent launch and execution instructions.
- `notableEvents()` and `wfe run events`, which select events but currently send nothing.
- Existing Python Zulip client and channel-folder helpers used by the legacy autolab path.

Keep workflow compilation and execution rules in the editor's shared implementation. Extend autolab's integration layer to deliver committed events through the existing Zulip client. Reuse transport and folder operations without importing the legacy mission/task state model or triggering its listeners.

Inspect the current installed release and repository state before implementation. Earlier reports are evidence of prior checks, not a substitute for testing the new code.

## 2. Documentation policy and definition contract

Add a workflow field:

```yaml
documentation:
  policy: default  # default | manual
```

Expose it in workflow creation and workflow-level editing. New workflows default to `default`. Validate unsupported values and define missing-field behavior explicitly in the updated schema; do not silently change an existing run's execution rules.

Include the policy in the definition approval projection and update the projection version. Changing the policy makes the definition approval stale; it does not independently change intent approval. Layout and presentation remain outside semantic digests.

Persist the author's original nodes, edges, and policy in the editable YAML. Do not insert generated nodes into that source file. One shared compiler produces the effective execution graph for both editor preview and run creation.

At run creation, freeze the source definition, policy, effective graph, and compiler/contract version needed to interpret it. Run history and readiness use that fixed graph. Later source edits or software updates must not silently regenerate an existing run differently.

Use stable generated step identifiers, distinguish them from author-defined node IDs, and avoid collisions. Record each generated step's purpose, associated original node where applicable, output path, and execution instructions. The generated plan/report kinds are internal execution steps; p5 does not add them to the manual node palette.

No general backward-compatibility layer is required. Update active definitions explicitly, diagnose unsupported records, and retain prior real-run evidence without rewriting its meaning.

## 3. Effective execution graph

### Default policy

For an authored graph such as `study → implement → human-check`, generate:

```text
plan talk
  → study → report(study)
  → implement → report(implement)
  → human-check → report(human-check)
  → final report
```

Compilation rules:

1. Add one initial plan talk before all authored entry nodes.
2. Add one report step after each authored node.
3. Replace every dependency `A → B` with `A → report(A) → B`.
4. After every authored terminal node's report succeeds, enable one final report step.
5. Do not generate reports for generated steps. There is no report-of-report recursion.

For a branch, each branch's report gates its own successors. A join waits for every predecessor report. The current single execution slot still performs eligible branches sequentially; this feature introduces no additional agents or processes.

An authored node's work can be complete while its report is running. Show those facts separately. The run cannot become completed until its final report step completes. Count authored work and documentation steps separately where a combined count would mislead the person.

Keep delegate execution refused during preflight even if the compiler can display its documentation preview. Handle empty or invalid author graphs explicitly rather than allowing generated steps to make an invalid workflow appear executable.

### Manual policy

The effective graph is the authored graph. Do not insert plan/report steps, require their files, or add generic instructions that make the agent create them automatically. An explicit node instruction or human request can still ask for documents.

Run state, questions, outcomes, and event summaries remain available under manual policy. They are ordinary execution records, not an implicit requirement to write Markdown plans or reports.

## 4. Document lifecycle and agent behavior

Keep all documents inside the existing run folder:

```text
devdocs/runs/<workflow-id>/<run-id>/
  braindump.md
  plan.md
  report-<authored-node-id>.md
  report.md
  definition/
  run.json
```

Preserve the existing distinction between a human braindump and an agent-authored request. A planning summary never replaces the original input.

### Plan talk

The agent drafts `plan.md` before asking the person to agree to the plan. It describes the intended result, work sequence, verification, relevant restrictions, and open questions. The question includes enough context to answer and a reference to the document.

Use the existing ask → process exit → answer → new attempt → take-up sequence. Discussion can revise the plan and ask a follow-up question. Record the plan revision associated with the question and agreement so a later edit cannot appear to have been approved already. Complete the plan step only after taking up agreement to the applicable plan. A withdrawal alone does not establish that agreement.

This agreement concerns the proposed work. It is separate from workflow-definition approval and result acceptance.

### Per-node report

After the authored node completes, the agent starts its report step, writes `report-<node-id>.md`, and records completion with a short outcome and artifact reference. A report should state:

- What was done or decided.
- What evidence or checks support the result.
- Remaining uncertainty, limitations, or blockers.
- Relevant artifacts and the next action, if any.

A talk report can be brief and record the question, answer, and resulting decision. It need not duplicate an entire conversation. Do not invent tests or imply that an unchanged artifact was modified.

### Final report

After all authored work and its reports finish, write `report.md` summarizing the request, delivered result, verification, remaining work, and any human decisions already recorded. Link to detailed reports rather than copying all of them.

Final report completion establishes execution completion. Keep the existing explicit result decision separate: do not turn a document, a node's completion, or an unrelated answer into `accepted`. If acceptance was expressed in a talk answer, preserve that evidence accurately without fabricating a separate decision.

### Shared mechanics

Document prose is written by the executing agent. Extend the execution brief and guide with the current step's purpose, output path, and content expectations. No separate model call or deterministic document generator is required. The agent can continue from work into reporting within the same attempt.

The phase transitions must nevertheless be explicit: start and complete generated steps through the shared run operations so the UI can show reporting and history can replay it. Extend attempt fencing, receipts, stopping, and resumption to generated steps.

Before completing a documentation step, the file layer verifies that the required file exists, is nonempty, and is confined to the run's reporting area. Record the evidence needed to identify the checked document. The pure reducer validates the transition and evidence shape; it does not access the filesystem. File presence is not a claim that the prose is correct.

If reporting is interrupted, resume the report step without repeating completed implementation. If authored work fails or is cancelled, normal successors remain blocked and the run is not successful. When the agent can still write, allow it to attach a partial report as evidence. If it cannot, show the missing report; do not invent a successful report or a final report after a crash.

## 5. Editor and run presentation

Use the shared compiler for the definition preview and the frozen effective graph for the run view.

- Display the initial plan talk as a normal-sized talk card because it requires human interaction.
- Display each per-node report as a small, lower-saturation circular icon at a fixed offset from the original node's lower-right corner.
- Move the report icon with its parent card. It has no independently editable position or authored dependency edges.
- Display the final report as a compact endpoint with its own status.
- During work, highlight the authored node. During reporting, highlight the report icon and label the phase “Writing report.”
- Show pending, running, completed, failed, and interrupted/held context without relying only on color. Provide an accessible label and selectable detail view.
- Open the relevant document from its icon. A missing or unfinished file must be shown as such.
- Keep the primary dependency lines readable between authored nodes, while detail views explain dependencies on report completion.
- Fit, zoom, selection, and layout calculations must include the auxiliary icons so they are neither clipped nor confused with other cards.
- Manual policy shows no generated plan/report steps.

Changing the policy in the editor changes the preview and approval status. It does not mutate saved runs or discard an unsaved definition draft when run events arrive.

## 6. Zulip structure and identity

Use the hierarchy proposed in the braindump:

| Zulip object | Corresponding entity |
| --- | --- |
| Channel folder | Project |
| Channel | Workflow within that Project |
| Topic | Run of that Workflow |

With Zulip integration configured, schedule folder provisioning when a project is created or registered. Provision workflow channels when saved workflows become known, and ensure the required structure before the run's first notification. Also reconcile externally edited definitions, since project files remain editable outside the UI.

Store project/workflow identities and Zulip folder/channel IDs in local integration state. Record the run topic binding and returned message IDs. Scope run identity by Project + Workflow + Run; reject ambiguous duplicate identities rather than mixing separate workspaces' runs in one topic.

Channel names must be unique across the realm, so include both project and workflow identifiers using a naming convention distinct from legacy autolab execution entrances. Validate supported name lengths and handle truncation without collisions. Display names are labels, not identity.

Reuse the existing Zulip transport and folder operations after inspecting their contracts. Do not infer ownership from an existing matching name or attach to a legacy mission topic automatically. Renames must retain recorded identity; deletion or loss of permission must produce a visible integration problem with an explicit repair path.

Credentials stay in system configuration, outside managed repositories. Link the run view to its Zulip topic once provisioned. Do not post secrets, full prompts, raw tool logs, or machine-local paths as coordination content.

P5 sends notifications only. Zulip replies do not answer a question, resume a run, or invoke the old listener. Question notifications direct the reader to the agdevworld answer operation. Test this separation against the actual legacy listener routing rules.

## 7. Notification content

Extend `notableEvents()` to select useful coordination events rather than copying every run history entry.

| Event | Content |
| --- | --- |
| Request received / execution started | Requested result, executor, and current next action; combine adjacent routine updates where useful |
| Plan or other human question | Concise question, who should answer, document reference, and answer route |
| Authored node's report completed | Outcome, main verification facts, remaining concern, and next action |
| Failure, interruption, stop, or resumption | What happened, what is known, and who must act |
| Final report completed | Overall result, limitations, and remaining review if applicable |
| Explicit result decision | Accepted or rejected, by whom, with evidence |
| Publication failure / recovery | Whether detailed records and artifacts are published and any action needed |

Do not post every tool call, heartbeat, generated-report start, or internal state transition. Combine routine adjacent events without losing critical questions, failures, or decisions. Use the same selection rules during live delivery and recovery.

Have the executing agent record a concise summary when reporting an outcome. The integration formats that summary together with factual run state and verified references. Existing event data suffices for mechanical events such as stopping; no new AI summarizer is needed.

Example:

```text
Implementation complete: added the heavy-warrior unit.
Verification: 14 headless checks passed; browser behavior checked.
Next: awaiting the person's result review. [Report] [Answer]
```

Enforce a short message budget and preserve actionable content when shortening it. Avoid unsupported success claims. Completion, acceptance, and publication are distinct facts and must remain distinct in notifications.

## 8. Durable delivery and evidence links

Devdocs remains authoritative. Save the run event first, then make it eligible for notification. Git publication and Zulip delivery are separate from the local record write.

Use a durable local outbox and delivery cursor, linked to immutable history entries or publication-checkpoint events. One history entry may produce more than one notification type; include event type or a stable discriminator in delivery identity rather than assuming sequence alone is unique.

- Reconstruct missed outbox entries after a crash between recording an event and queueing its notification.
- Retry transport failures without rerunning work or changing recorded node state.
- Retain sent message IDs and an event marker so a response lost after successful posting can be reconciled before retrying. Do not claim exactly-once delivery where the transport cannot guarantee it.
- Preserve per-run event order, and label delayed notifications so recovered old messages do not look like current questions after the run has moved on.
- Show provisioning errors, pending delivery, and last successful delivery in the run/integration view. Zulip failure must not make a completed run look failed or prevent human answers in agdevworld.
- Run delivery outside the agent session and independently of a blocking model invocation. It must continue during human waits and when the execution queue is idle.
- Ensure project/workflow provisioning is also durable and retryable. A successful Git project creation is not undone by a Zulip outage.

Prefer commit-addressed Gitea links for published reports and artifacts. Resolve the owning repository correctly for directory and submodule devdocs. Publish checkpoints currently occur after attempts, so a report may exist locally before Gitea can serve it.

Send urgent questions or blockers without waiting indefinitely for publication. Label unpublished details explicitly; do not provide a fabricated Gitea link. After publication succeeds, update the corresponding message's detail link or send one concise availability update, tracked through the same delivery mechanism.

Configure the browser/run-view base URL separately from repository URLs. A `localhost` address is meaningful only on the execution host and must not be advertised as a working link for remote readers. If no reachable answer URL is configured, state where to answer and expose that configuration gap. Remote UI hosting is not part of p5.

## 9. RTS rollout and existing definitions

Update the active RTS workflow deliberately to use the default policy. Its existing initial `confirm-scope` talk overlaps with the generated plan talk; incorporate the useful scope instructions into planning and remove the redundant authored step from the new definition. Do not automatically remove arbitrary user-authored talk nodes from other workflows.

Keep the human result-check step unless the agreed RTS workflow is explicitly revised. Its report and the final summary follow the normal default rules.

Existing RTS runs retain their historical meaning. Do not synthesize plans, reports, approvals, or Zulip conversations for old runs. Any required format changes must be explicit, with unsupported records diagnosed and prior evidence retained as files or an archive. Backfilling old Zulip activity is not required.

Deploy the tested editor through agdevworld's system release path and update the autolab guide/integration alongside it. Coordinate cutover with active attempts so one run does not cross incompatible execution contracts unexpectedly.

## 10. Implementation sequence

1. **Contract and compiler:** define policy fields, generated step identities, the graph expansion, snapshot/version behavior, approval digests, and document evidence. Update model, YAML, validation, and shared compilation tests.
2. **Execution and documents:** connect generated steps to run creation, reducer/CLI/API operations, prompt briefs, and file checks. Establish plan agreement, reporting, failure, and resumption semantics.
3. **Views:** add policy editing, preview, plan talk, attached report icons, final report, and document/status inspection. Verify branching, joins, layout, and draft preservation.
4. **Zulip integration:** add durable provisioning, bindings, event summaries, delivery state, retries, and published-evidence links using the existing client. Keep human replies in agdevworld and legacy listeners inactive for this route.
5. **Rehearsal and rollout:** test both policies and failure paths on isolated data, deploy through the system lifecycle, update the RTS definition, and perform a human-led real run.

Record each step's implementation scope, commands, results, limitations, and next entry point in `report*.md` in this directory. Summarize completion and operation in `report.md`.

## 11. Validation and completion

Run focused contract/integration tests, the editor's type checks/tests/build, agdevworld's required checks, and the affected autolab tests. Use dedicated repositories, registries, and Zulip test destinations for rehearsals; do not populate real project topics with synthetic progress.

Required evidence:

1. Default compilation produces one plan, one report per authored node, and one final report, with no collisions or recursive expansion. Manual produces none.
2. Branches and joins wait for the correct reports; run completion waits for the final report. Delegate execution remains refused.
3. The editor preview and run's frozen graph agree. Policy or source changes do not mutate an existing run, and policy changes invalidate definition approval.
4. The plan is written before consultation; disagreement/revision can lead to another question, and work waits for recorded agreement to the applicable plan.
5. Required reports exist and are nonempty before completion. Work completion and reporting are independently observable.
6. Interrupting during a report resumes reporting without repeating completed work. Failure/cancellation does not fabricate a report or successful run. Late updates from a replaced attempt remain refused.
7. Manual policy neither generates documents nor prompts for them implicitly, while explicit document requests still work.
8. Report icons remain attached, selectable, visible under fit/zoom, and distinguishable without color; reconnect and run updates preserve unsaved definition drafts.
9. Project/workflow/run mappings provision the correct Zulip hierarchy, survive restart, and do not trigger legacy listeners or merge distinct run identities.
10. Recording an event before a delivery crash, losing the response after posting, and a prolonged Zulip outage preserve work records and permit reconciliation/retry without repeating work. Delivery remains active during human waits.
11. Published links work in both devdocs modes. Pending publication and remote-unreachable UI links are described honestly, and eventual publication supplies a valid reference.
12. The real RTS run leaves `braindump.md`, the agreed `plan.md`, per-node reports, and `report.md` in Git/Gitea. Its Zulip topic contains concise, truthful progress, questions, and completion information linked to the evidence.

P5 is complete when both policies meet their contracts, a real default-policy RTS run has been reviewed by the person, and the person can use Zulip to identify current status and the next action, then reach the detailed records and artifacts when needed. Record any remaining explicit result-acceptance action separately from workflow completion. Synthetic tests alone do not establish the real coordination experience.
