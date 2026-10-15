# Workflow editor p10 — delegation and inspectable execution

## Objective and scope

Implement the direction discussed from `braindump.md`: a workflow can entrust
work to another managed project's workflow, answer its questions, receive its
artifacts and continue with the result. People can follow both the overall
progress and the detailed execution from the originating Zulip conversation.

Keep `delegate` as a distinct node type and reuse ordinary run execution,
attempts, sessions, questions, isolated copies and publication. Agents decide
what to request, how to answer and whether a result meets the request. The
system owns durable relationships, delivery, execution ownership and resumption.

The existing mechanical `Request received` notification is sufficient for this
phase. The Developer clarified that the reported absence may have been observed
on an older version. Preserve and verify that notification; do not add an early
receipt stage, new acknowledgment protocol or notification-latency project.

This document plans implementation and deployment. Creating it does not start
implementation, send requests or change running services.

## Starting points

- `agdevworld/workflow_editor/shared/model.ts` and `shared/validate.ts` define
  delegate targets within the current project. `server/runs.ts` and
  `shared/run.ts` refuse delegate execution, including graph adjustments.
- `shared/catalog.ts` provides descriptive templates over executable node
  types. A delegation relationship needs more than a template's instructions.
- `shared/intake.ts` currently permits configured agents to open requests in
  named workflows, but ignores their subsequent answers and instructions.
- `shared/run.ts` has request provenance (`requester`, `onBehalfOf`,
  `entrustedBy`), but its run entrustment reference lacks cross-project identity.
- `server/notify.ts` queues the existing request receipt after run creation;
  `shared/notify.ts` renders it mechanically with progress/run links.
- `agautolab/src/agautolab/wfexec.py` launches one attempt at a time and keeps
  attempt logs. Waiting must release that slot for the child to execute.
- Observer consumes WFE observations. Normal delegation delivery and resumption
  belong to WFE; Observer detects failures of that path.

Paths above are relative to `pj-agdev`. Recheck the implementation before each
step. Consult Nautobot or `pj-clusterintent/nctl` whenever inspecting deployed
services. Keep host addresses, credentials, live run identifiers and absolute
local paths in ignored environment notes.

## 1. Specify delegation identity and decision ownership

Write the delegation contract before changing the execution guards.

- Define a portable target with project and workflow identity. Resolve its
  workspace explicitly at dispatch, reporting ambiguous registrations rather
  than guessing. Freeze the resolved child definition through normal run creation.
- Give each delegation a stable ID bound to the parent run UID and node. Record
  the request text/digest, requester, entrustment, transport references and child
  UID. Reuse the observation contract's creation-qualified run identity.
- Preserve original human input separately from the parent's constructed request.
  Extend entrustment provenance to identify the parent across projects.
- Initially allow one child run per delegate-node instance. Delivery retries
  retain the same delegation ID. A replacement after failure is explicit new
  work with a new identity and a predecessor reference, not a silent replay.
- Record who may answer the child's questions and decide on its result. The
  parent agent handles decisions within its entrusted scope; matters beyond that
  scope return to the person. Preserve the actual actor and supporting evidence.
- Distinguish delivery, child execution, result availability, parent receipt,
  child acceptance and integration. Define whether the entrusted scope permits
  child acceptance that queues integration. Consuming an artifact alone must not
  silently authorize changes to the child's shared branches.
- Keep acceptance bound to the reviewed revision. Parent acceptance of a child
  result does not accept the parent's final result for the person.
- Check cross-project reference cycles for declared targets and active ancestor
  cycles at dispatch, including definitions changed since the parent was created.

**Completion evidence:** a documented transition/ownership table covers normal
delivery, questions, results, rejection, failure, hold and cancellation, including
the existing plan and result confirmation steps used by forge.

## 2. Implement durable dispatch and two-way communication

Add delegation operations through shared server logic, CLI and API. Keep Zulip
as the request/conversation interface and run records as the state authority.

- Persist dispatch intent before sending. Journal delivery and bind the child
  created through intake to that intent. A crash after sending or child creation
  must recover the same request and child without duplicating work.
- Extend agent intake for answers and further instructions associated with the
  agent's recorded delegation. Validate the sender and relationship; merely
  posting a parent ID must not establish ownership.
- Correlate questions by child UID and question ID. Preserve message receipts,
  answer provenance, ordering, topic relocation and stale-answer behavior.
- Deliver question, result and actionable failure events to the parent using
  durable receipts. Progress updates remain visible without invoking the parent
  model on every update. Reconcile missed events from authoritative run records.
- Give agents inspect, request, answer and receive operations with sufficient
  `--help`. Extend the normal run brief with pending delegation events and child
  references. Keep procedural tool usage in help rather than duplicating it in
  every guide.
- Define a result manifest with outcome, report and retrievable artifact
  references, including hashes or commits where applicable. Child-local paths
  alone are not a delivery contract; use forge's durable delivery where suitable.
- Keep normal requests and delegated requests on the same creation and
  execution path, with explicit delegation provenance.

**Completion evidence:** duplicate dispatch and interrupted send/bind/application
produce one child; multiple questions and answers retain the correct identities;
an unrelated sender cannot answer as the parent.

## 3. Connect waits, resumption and result handling

- Allow delegate execution through the shared run state machine once dispatch
  and relationship recording are available. Update creation, adjustment,
  compilation and validation together; remove the unsupported warnings then.
- A parent can continue independent work. Once only delegated/external work is
  pending, it ends its attempt and frees the executor slot. Never poll for the
  child's completion inside an attempt that prevents the child from running.
- Queue a parent continuation for a child question, result or actionable failure.
  Coalesce pending work while preserving every event. Handle events arriving
  before the parent attempt ends without dropping them or starting a second writer.
- The parent reviews the delivered result and records receipt and disposition.
  Complete the delegate node only when its requested purpose is satisfied;
  dispatch or a child's terminal state alone does not complete it.
- Keep child plan agreement, result acceptance, publication and integration
  distinct. Exercise parent-agent answers against forge's real confirmation
  pattern, including the revision checks around acceptance.
- A parent hold retains incoming events and prevents automatic resumption.
  Stopping a parent does not implicitly cancel its child. Parent cancellation
  records the outstanding child's disposition; cancelling the child is a separate
  explicit action. Late results stay inspectable and do not revive cancelled work.
- Child failures remain visible and actionable. No automatic new child is
  created as a substitute for investigating a failed or uncertain dispatch.

**Completion evidence:** the single-slot executor completes a parent/child
question-and-result round trip, survives restart, respects a parent hold and
never runs two attempts for the same run simultaneously.

## 4. Provide detailed IO channels and direct links

Provide one Zulip IO channel per run, stable across attempts and session changes.
Use recorded channel IDs and run UIDs as identity, not channel names.

- Inspect existing logging utilities before implementing the relay. Derive IO
  from recorded harness output: public agent messages, tool calls/results and
  execution lifecycle events. Do not describe private model reasoning as available.
- Persist channel provisioning and log-delivery positions. Stream incrementally
  with batching, message-size handling and recoverable delivery. Long output
  needs a durable full-output link or attachment beyond its inline excerpt.
- Mark attempt boundaries and restarts in the same channel. Resume delivery
  without replaying the entire log; detect truncation or unavailable source data
  and report a gap instead of presenting an incomplete log as complete.
- Keep log delivery outside the agent's execution slot. A Zulip outage must not
  stop useful work; retain pending logs and expose delivery failures.
- Treat IO as output only. Its bot messages must not enter request intake or
  wake agents. Avoid publishing credentials from harness metadata/configuration;
  preserve useful tool output and use the project's established redaction rules.
- Add Progress, Detailed IO and Run links to the existing mechanical receipt.
  If IO provisioning is pending, send the existing receipt without waiting and
  enrich it once the destination exists. Do not claim a working link prematurely.
- Show the same IO link in the run view. A parent delegation event and node card
  link to the child's progress, IO and run view; do not duplicate child logs in
  the parent channel. Keep questions and responses in the request conversation.

**Completion evidence:** a person reaches current tool activity from the request
receipt and follows a delegated child without searching channels. Multiple
attempts, long output, restart and interrupted delivery retain that navigation.

## 5. Add editor views and delegation-aware observation

- Provide project/workflow selection and validation in the delegate inspector.
  Show child identity, current responsibility, pending questions, result artifacts
  and disposition in the node/run views. Reuse common run operations and states.
- Keep a custom-node template as an optional way to describe a delegation task;
  do not build a generic plugin execution framework for this phase.
- Extend observations with delegation relations and pending event/delivery facts.
  Observer can distinguish a healthy child, a child's human wait, a child failure,
  an unbound dispatch and a delivered event with no eligible parent continuation.
- Preserve human holds in recovery. Correlate parent dependency symptoms with
  child incidents so the same child failure does not cause duplicate recovery runs.
- Keep normal resumption in WFE. Observer recovery remains an exceptional path,
  not the mechanism that makes every successful delegation continue.

**Completion evidence:** CLI, API, browser and Observer agree about which run is
waiting for whom; child progress is visible without unnecessary parent attempts.

## 6. Verify, deploy and report

Use focused fixtures for durable boundaries and a real forge round trip for
end-to-end evidence. Run the workflow editor's required checks and affected
executor/Observer tests, plus browser checks for the new navigation.

| Scenario | Required evidence |
| --- | --- |
| Normal delegation | Parent requests forge work, answers its question, receives a usable artifact and continues |
| Decisions | Plan agreement, artifact receipt, result acceptance and integration retain their distinct meanings and actors |
| Recovery | Crash after send, child creation, event application and parent queueing does not duplicate work or lose continuation |
| Scheduling | One slot suffices; independent runs progress while a parent waits |
| Controls | Parent hold/cancellation, child failure and late results preserve explicit decisions |
| IO | Stable channel across attempts, long-output access, recoverable delivery and parent-to-child navigation |
| Existing intake | Mechanical receipt still arrives without AI-generated text; ordinary requests and answers still work |
| Observation | Healthy waits do not trigger recovery; broken continuation is detectable without duplicate child recovery |

Before deployment, inspect service state through nctl/Nautobot and current runs
through the installed WFE CLI. Version affected contracts and migrate explicitly
where needed; do not retain obsolete behavior merely for backward compatibility.
Preserve historical evidence and pending receipts. No migration may resend old
requests, revive held runs or manufacture IO logs that were never retained.

Commit and push changed repositories and applicable superproject pointers. Build
and deploy matching editor/executor/Observer versions, coordinating writers when
store changes require it. Keep local recovery copies and machine-specific facts
in ignored notes. Do not reintroduce removed automatic release rollback behavior.

Verify the deployed receipt, detailed IO and delegation round trip before asking
the person to inspect it. Leave normal LAN-accessible entry points and direct
links ready so human verification requires minimal navigation. Record results,
source/deployed versions and remaining limitations in `report.md`, update shared
developer documentation and local environment notes, and commit/push the reports.

## Definition of done

- A delegate node completes a real cross-project request/question/result cycle.
- Durable identity and receipts survive retries and restart without duplicate work.
- Waiting releases execution capacity and child events resume only eligible parents.
- Decision ownership and integration authority are explicit and evidenced.
- The existing mechanical receipt leads to progress, detailed IO and the run view.
- A person can follow delegated work down to the child's detailed output.
- Observer understands delegation waits and detects broken delivery/resumption.
- The deployed system is verified, documented and ready for immediate inspection.
