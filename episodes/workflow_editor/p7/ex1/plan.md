# Workflow editor p7 ex1 — user-designed nodes and visible work

## Goal

Implement [braindump.md](braindump.md) by giving the person an editable catalog of reusable node templates, making that catalog available in workflow editing and run execution, and enabling the agent to represent requested work at the granularity the person expects.

The catalog states the person's expectations and gives the agent tools to realize them. Whether a task deserves its own node remains a design judgment. This phase does not introduce keyword-triggered graph expansion or require every applicable template to become a node.

## Evidence and agreed direction

In the RTS `improve-game/run-004` trial, the person explicitly requested Web research before implementation. The first attempt performed one Web search and received results, then summarized approaches in `plan.md`. It did not add a study node or retain sources in a research report. The plan promised research details in the implementation report, but that report omitted them. This trial supports two separate improvements:

- Tell agents which activities the person recognizes as distinct workflow steps, and how to add them.
- Preserve evidence and findings for requested research, whether it is a separate node or part of another step.

Represent user-designed nodes as templates over existing execution types, with a description for the agent performing the work and an expectation for the agent designing the workflow. Reuse p7 appearance and adjustment mechanisms. Keep original run definitions, started work, questions, decisions, and immutable revision evidence intact.

Start with one catalog per agdev operating area for its current user. Account management, per-project inheritance, template dependencies, a general resource system, and changes to the semantics of `study`, `do`, `talk`, or delegation are outside this phase.

Experimental breaking changes are allowed; backward compatibility is not required. Inspect affected records and active processes before cutover, preserve useful evidence, and document any explicit conversion. Keep host facts and credentials out of tracked documents and project definitions.

## Step 1 — define catalog, instantiation, and snapshot contracts

Inspect the current editor, p7 implementation, executor guide, and relevant trial records. The editor is system code in `pj-agdev/agdevworld/workflow_editor`; `pj-agdev/agautolab` supplies the run executor and common guide. Read the applicable development policies. For local service observations or deployment, consult Nautobot or `pj-clusterintent/nctl` rather than relying on old environment notes.

Define a versioned catalog stored as user data beside the operating area's configuration, outside managed project workspaces and outside the installed release. Its contents are portable preferences; keep machine-specific runtime configuration separate. UI and tools use the same persistence and validation operations.

Each template has:

- Stable ID and revision identity.
- Name and existing execution type.
- Description: what the agent performing this activity should do and record.
- Expectation: how the person expects the activity to appear in the workflow.
- Optional appearance, using p7's icon and color fields.

Do not bind project repository names in the global catalog. Instantiation selects bindings available in the target workflow and cannot grant additional access. Define how type-specific fields are supplied and validated; exposing a template must not imply that unsupported delegate execution becomes available.

Instantiation copies template fields into an ordinary editable workflow node, recording its template ID and revision as provenance. Subsequent catalog edits do not modify instantiated nodes. Distinguish provenance, semantic node fields, and appearance in digest/approval behavior: descriptions and other work changes retain semantic approval effects; provenance and appearance alone do not change executable meaning.

Define run catalog snapshots, what gets delivered to an agent, and how unavailable or malformed catalogs are reported. Save an explicit empty snapshot when the person has no templates rather than confusing an empty catalog with a failed read.

Deliverable: contract documentation and concrete schema, ownership, digest, and cutover decisions.

## Step 2 — implement the user-node catalog and editor

Add a “User nodes” entry from the agdev dashboard. Provide a list, create/edit/delete operations, and card previews. The editor exposes name, type, description, expectation, icon, and color without requiring YAML editing.

Persist catalog changes through a shared service layer, with validation, atomic writes, and stale-edit detection. Assign a new revision when template content changes. Distinguish an intentionally empty list from a failed load; do not overwrite unreadable user preferences with defaults.

Move the current presentation and cleanup presets into catalog entries using their existing descriptions and appearance. Record the one-time initialization so deletion or customization is not undone by a later release. Offer a preliminary-research example that the person can choose and edit; do not silently declare new personal preferences on their behalf.

Provide API/CLI operations to inspect and manage the same catalog. Help describes what each field means and how to instantiate a template. Resolve the operating area through the established explicit registry configuration rather than an implicit alternate catalog.

Deliverable: the person can design reusable nodes, and the UI and tools read the same persisted catalog.

## Step 3 — use templates in workflow editing

Extend “Add node” to show catalog templates alongside existing basic execution types. Selecting a template creates an ordinary node, with a unique workflow node ID, copied content and appearance, and template provenance. Let the person choose repository bindings and edit the resulting node independently.

Show the originating template's expectation and revision when inspecting the node. Preserve this information in YAML round-trip, node copy operations, and definition snapshots. Editing or deleting a catalog entry never silently changes an existing workflow, run, or approval.

Provide a shared instantiation operation callable from the UI and CLI/API so an agent editing workflow definitions can use the same template contract. Existing descriptions, repository access checks, validation, compilation, and approvals still determine the resulting node's behavior.

Deliverable: user-designed nodes are available to human and agent workflow authors through consistent interfaces.

## Step 4 — deliver the person's expectations to run agents

At run creation, save the selected operating-area catalog as immutable run context with its content hash. Deliver that snapshot to every attempt, including resumed attempts. A subsequent catalog edit affects new runs, not an existing run's expectations. Do not rewrite completed runs to claim they used a catalog they never received.

Include the expectations in the agent's execution information with a concise statement of their meaning:

> The person recognizes these activities as distinct steps. When relevant to this request, they generally expect them to be visible as workflow nodes. Choose whether and where to represent them using the request and the current workflow.

Give the agent the catalog-inspection and instantiation tools, the overview of run adjustment, and the commands for obtaining detailed help. Keep tool usage in help rather than duplicating it across guides. Expose the same catalog information to workflow-authoring tools; do not assume an agent can see browser-only controls.

Record which snapshot was supplied and show it in run inspection. Ensure that the agent reads the run snapshot, not a live catalog that changes between attempts. Preserve the existing attempt setting/guide evidence conventions when modifying the execution brief.

Update the common guide with facts derived from the run-004 trial: requested research needs traceable sources, findings, choice rationale, and uncertainties in the records; the plan identifies where that evidence will live. A separate study node gets its normal node report. Integrated research still has identifiable evidence in an appropriate report. Summaries and final coordination should link to that evidence.

Deliverable: the person can inspect what expectations the agent received, and agents have both the context and tools to express those expectations.

## Step 5 — allow planning adjustments grounded in the original request

Extend adjustment provenance to support the immutable original braindump or entrusted request. The current operation accepts taken-up answers, acknowledged instructions, exact-proposal agreements, and person-applied changes; initial request-based planning needs a direct supported source as well.

Bind this new source to the run's actual input identity and immutable content evidence. It expresses why a change was made; it is not a claim that the person approved a particular graph. Existing attempt fencing, revision checks, locking, instruction boundaries, and adjustment restrictions still apply.

Expose this path in adjustment help and the agent's planning context. The agent can propose future study work from an initial request and apply the adjustment at a boundary, without a redundant permission request solely to express already requested work. Material changes of scope retain the existing agreement mechanism.

For the formation-movement example, the agent can include a preliminary-research template in its proposed plan, agree the plan, and add `study → implement` before either authored activity starts. The generated report after study provides the research evidence before implementation. Applying an adjustment while `_plan` is running remains invalid; use an eligible boundary, including after completing the agreed plan.

Avoid duplicate nodes when the current workflow already expresses the activity. If the agent integrates an activity instead, make that treatment visible in the plan and retain its evidence. Catalog preferences never automatically expand the graph or alter repository access, execution identity, or completed work.

Deliverable: an initial request can justify future-work adjustments with reproducible provenance, while plan agreement remains a separate record.

## Step 6 — verify behavior and deploy

Use isolated catalogs, registries, projects, and run records for deterministic checks. Run affected editor tests, required type/build checks, browser checks for the catalog and node-add flows, and executor checks for the changed brief. Test behavioral contracts rather than duplicating implementation details.

Required evidence:

1. Catalog CRUD, persistence across service restart, validation, and stale-edit handling work through the shared interfaces.
2. Templates instantiate editable ordinary nodes with valid bindings and provenance; catalog edits/deletion leave existing definitions unchanged.
3. Semantic changes retain approval effects; display/provenance-only changes do not acquire execution meaning.
4. The browser and an agent using CLI/API see the same catalog. New and resumed attempts of one run receive the same immutable snapshot.
5. Failed catalog reads are visible rather than treated as an empty list or overwritten.
6. Original-request adjustment evidence identifies the correct input; stale attempts, invalid revisions, mid-node changes, changed access, and pending instructions retain their existing refusals.
7. A study insertion yields the correct compiled dependencies and report steps, and the adjusted run replays with its original definition and evidence intact.
8. Templates do not trigger graph changes by keywords, imply new capabilities, or create duplicate mandatory work.
9. Existing presentation, result confirmation, repair adjustments, and cleanup remain functional.

Inspect active attempts before release. Deploy through the established agdevworld release mechanism; update the executor guide through its existing lifecycle. Record exact commits, any schema/cutover changes, service observations, and rollback limits.

Deliverable: a tested release and step report with concrete evidence and limitations.

## Step 7 — exercise the research request with an agent

Use the run-004 request as an isolated behavioral trial with a person-designed preliminary-research template. Compare the supplied execution information with the earlier trial. Do not alter run-004's completed history or attach retrospective evidence as if it had been recorded during the original research.

Check that the agent recognizes the preference, proposes a suitable sequence, can add a study node using the tools, and records sources, findings, the implementation choice, and uncertainties before implementation. Test the adjustment and report path deterministically in fixtures; evaluate the agent's interpretation separately rather than claiming a tool test proves it will make the design choice.

Also exercise a workflow that already has suitable research work and a case where integration is reasonable. Assess the stated plan, actual work, and evidence. Absence of an extra node alone is not a failure when the existing workflow already represents the activity or the integration is explained consistently with the request.

If a new live RTS improvement run is needed, use a new human-authored request and the person's actual planning/review answers. Do not manufacture agreement or send project-topic messages merely to produce a successful trial. Distinguish isolated evidence from a real human-reviewed run.

Deliverable: evidence that expectations reach the agent, node creation is usable, and requested research becomes understandable from the ordinary run records without inspecting private execution logs.

## Completion and reporting

Ex1 is complete when the person can edit their node catalog, use it in workflow editing, inspect the snapshot supplied to a run, and an agent can adjust future work from the original request with research evidence visible in the normal reports. The catalog conveys preferences while preserving agent judgment and existing execution constraints.

Write step reports and a final `report.md` with changes, verification, trial outcomes, limitations, and material departures from this plan. Reference the run-004 trial wherever guidance derived from it moves. Keep host-specific facts in ignored development documentation. Record outside-agent work performed on behalf of an in-system agent as a handoff candidate under the development policy.

Commit and push intended changes according to local rules, excluding unrelated user edits.
