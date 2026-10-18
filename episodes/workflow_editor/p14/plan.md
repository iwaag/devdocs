# Workflow editor p14 — agent-maintained environment context

## Objective

Implement [braindump.md](braindump.md): let cagent publish concise cluster
knowledge into local system and project context, with visible ownership,
traceable evidence and a practical route to details. Newly started projects
should discover useful existing resources without repeating basic questions.

Extend p13's context store, resolver, editors and attempt snapshots. Keep
Nautobot/nctl authoritative for cluster state; context is a selected explanation
of that state and its relevance. Writing this plan does not start implementation.

## Working conventions

- Follow `localrule.md`: generated files are English; conversation with the
  Developer is Japanese. Implementation, trials, deployment, commits and pushes
  proceed without additional human approval checkpoints. The Omni Agent may
  answer in-system agents' permission requests on the Developer's behalf, using
  its actual identity. Do not add approval gates or restrictive classifiers.
- Finish implementation with a deployed, verified home-LAN result and direct
  inspection links. Perform setup and verification the agent can do itself;
  leave the Developer only a few inspection actions.
- Backward compatibility is unnecessary. Update active consumers together and
  retire duplicate paths. Preserve existing human content and historical
  evidence through a bounded data conversion where needed.
- Keep host addresses, machine paths, raw observations and runtime evidence in
  ignored local storage. Tracked code and documentation describe portable
  contracts and examples.
- Before runtime work, inspect service state through pj-clusterintent's
  Nautobot or nctl, following `pj-clusterintent/nctl/README.md`. Local environment
  notes are useful pointers, not proof that a service is running now.
- Preserve unrelated edits and explicitly held runs. Use bounded fresh trials.
- Give cagent tools and sufficient help, with autonomy to select useful facts.
  Keep tool syntax in help, shared contracts in one place and role-specific
  responsibilities in cagent's guides.

## Contract

### Independently managed context blocks

Support multiple blocks within each existing local override scope: system,
project and workflow. Run input retains p13's separate request/instruction
contract. System scope means one WFE area.

A block has a stable identity within its scope, title, manager, semantic kind
(instruction or environment information), body, revision and last editor.
Evidence metadata records source references, observation times, verification
status and relevant coverage. A manager identifies who maintains the block;
the last editor identifies who actually changed it. Neither is an authentication
claim or an exclusive write permission in this phase.

For example, the system scope can contain a Developer instruction block and a
cagent cluster-summary block. The forge project can contain its human guidance
and a separate cagent resource-advice block. Other agents can use the same model.

- Save, remove and inspect history per block, with optimistic concurrency and
  atomic writes. An update to one block never rewrites its siblings.
- Preserve human edits inside an agent-managed block in revision history. An
  agent reads the current revision before updating; an unseen edit causes a
  conflict, not an automatic overwrite. A later deliberate revision remains
  possible without an approval workflow.
- Keep blocks, histories, detailed local references and snapshots outside
  repository/run trees. Existing overrides become Developer-managed blocks;
  do not retain a second independently writable store.
- Preserve p13's scope semantics: breadth and provenance do not imply absolute
  precedence. Label environment information as facts/advice, distinct from
  instructions. Do not attempt to resolve conflicting prose automatically.

### Content selection and detail access

The system summary should normally occupy three to five short lines: major
resource capabilities, frequently useful services, and how to read details or
ask cagent. Treat this as an editorial target, not a silent truncation rule.
Show size in the editor so the cost of system-wide injection is visible.

Project blocks add relevant information without repeating the system summary.
For forge, this might cover usable generation services, model availability and
resource constraints supported by evidence. cagent reads project intent and
goals through the existing `wfe projects` interface; no separate project catalog
or mandatory per-project configuration is needed.

Provide both a non-conversational detail-read route and a cagent consultation
route. Reuse suitable existing read APIs; if they cannot expose the required
local evidence, add a narrow logical-reference read surface in the owning
service. References must work from the executor and the GUI, without requiring
knowledge of a host filesystem path. Full inventories are fetched on demand,
not automatically expanded into every prompt.

Operational facts remain rooted in Nautobot/nctl and cagent's evidence. Do not
copy machine configuration into tracked guides. Existing `shared://` contracts
remain authoritative for shared storage references.

### Evidence, age and failure

Distinguish publication time, source observation time and later verification
checks. Record whether a claim is reported, observed or a recommendation, and
preserve partial coverage and failed observations. A fresh summary must not make
old evidence appear newly observed.

Installed hardware is not free capacity; a desired placement or converged drift
is not a fresh application-level connectivity test. Model counts and endpoint
health require evidence of those specific properties. Mark unsupported claims
unknown rather than inferring them from service existence.

On refresh failure, retain the last successful information and show the failure
and its age. Do not publish an empty replacement or present old facts as current.
This is distinct from an unreadable context store, which follows the existing
explicit error contract. Context publication does not require a new host scan
unless the request or missing evidence calls for one.

### Delivery and session behavior

Use stable source keys per block and deterministic ordering. The same resolver
serves authoring, preview and execution. Resolve once at claim, retain the exact
snapshot and deliver its bytes through p13's executor path. Scope a delegated
run to its own target project and workflow.

Saving changes the next attempt, never an active attempt, and starts or resumes
no work. Effective content changes retain p13's session replacement behavior.

Separate verification bookkeeping from delivered guidance. Rechecking unchanged
facts can update evidence/history without changing every run's fingerprint.
Define this distinction explicitly in the resolver and renderer: metadata that
does not affect delivery must not appear as changing prompt text. Changes to
facts, source meaning or freshness status that affect decisions must change the
delivered content and fingerprint. Avoid a continuously changing age string in
the prompt; the GUI can calculate age from recorded times. Exact snapshots must
still explain the evidence available when an attempt was prepared.

Portable run records retain only logical source identities and hashes, not
local bodies or endpoint metadata. Historical snapshots stay readable locally;
an absent snapshot is reported unavailable. Keeping storage outside Git does
not promise that an agent can never quote local information in its own output.

### User interface

Reuse the embedded Context sections introduced in p13 ex1. In Simple mode, show
separate cards for human instructions and each agent-managed contribution, with
manager, last editor, applicable scope, concise content and meaningful freshness
or failure state. Editing remains available; ownership is visible rather than
enforced by an extra permission dialog.

Detail adds revisions, evidence references, hashes and full history. Support
block creation, editing and removal with independent save states, intact drafts
through refresh/navigation and useful conflict recovery. Inherited cards name
their actual scope. Run views retain “Used for this attempt” and “Changes for the
next attempt,” identifying the particular blocks that differ.

## Implementation steps

### Step 1 — extend the store and resolver

1. Inspect current source, repository instructions, installed release and local
   context data. Record the conversion and backup approach before changing data.
2. Extend `shared/context.ts`, `server/contextStore.ts` and `server/context.ts`
   for block identity, manager/editor separation, semantic kind and evidence.
3. Implement per-block revisions, conflict detection, deletion and listing.
   Preserve current human overrides and historical snapshots during conversion.
4. Connect stable source identities, rendering, snapshots and session comparison.
   Specify which evidence changes affect delivery and which are bookkeeping.
5. Update `workflow_editor/docs/context.md` as the authoritative contract.

Deliverable: one store and resolver supporting independent local contributions,
with meaningful concurrency and delivery tests.

### Step 2 — expose publication and detail access to agents

1. Extend CLI and HTTP operations to list, get, set, remove and inspect blocks.
   Make target area, project/workflow, block and base revision explicit.
2. Ensure an executor on another host can update the intended area's authoritative
   service rather than silently writing a local copy of its store. Reuse existing
   service discovery/transport and test the remote execution path.
3. Provide structured results with actual writer identity and revision. Record
   publication provenance, including the producing cagent run when available,
   without putting private payloads in published run records.
4. Establish a durable, logical route to detailed local evidence and a discoverable
   cagent consultation entrance. Verify both references from a consuming project.
5. Make repeated publication of unchanged guidance idempotent. A failed refresh
   retains the successful body and exposes the unsuccessful refresh separately.

Deliverable: tools that let cagent maintain its own contributions and let other
projects retrieve details without a conversation for every lookup.

### Step 3 — integrate the GUI

1. Extend reusable context components in `src/contextParts.ts`, `contextView.ts`
   and `runContext.ts`, and their Home/project/workflow/run integrations.
2. Show manager and last editor separately, with information/instruction labels,
   freshness, scope and size visible without switching to Detail.
3. Implement per-block editing/history and inherited-scope editing. Preserve
   drafts, focus and independent save outcomes during background refreshes.
4. Extend actual-versus-next-attempt inspection to block-level changes and
   evidence references. Keep prepared, delivered and unavailable states truthful.
5. Verify Chromium and WebKit journeys, including a narrow viewport, conflicts,
   human edits to a cagent-managed block and subsequent cagent updates.

Deliverable: ownership and update boundaries are apparent in the ordinary
editing surfaces, with no technical setup required from the Developer.

### Step 4 — implement cagent publication and prove project relevance

1. Inspect the current managed cagent project and its observation, reporting and
   consultation contracts. Add an explicit context-refresh workflow or equivalent
   existing entrance with system and selected-project targets.
2. Give cagent the new tools and access to project intent through `wfe projects`.
   Let it select and explain useful evidence; keep WFE responsible for storage,
   routing, provenance and delivery rather than domain-specific summarization.
3. Publish a short system summary and a forge-specific contribution from actual
   evidence. Keep generated local content and detailed observations outside Git;
   tracked workflow definitions/guides contain only portable behavior.
4. Demonstrate explicit refresh without requiring a scan, and a refresh after a
   relevant observed change. Exercise unchanged facts, partial observations and
   failed detail retrieval without manufacturing successful verification.
5. Start a bounded consumer trial that receives the contributions, retrieves
   details and identifies relevant resources. Verify the actual attempt snapshot
   and delivered context rather than relying on the final answer alone.

Deliverable: cagent, rather than an Omni-written substitute, publishes useful
system and project information that a consuming run actually receives.

### Step 5 — deploy, verify and hand off

1. Run focused tests and the repositories' required release checks. Back up the
   local context store and verify the conversion preserves human content and
   existing snapshot evidence.
2. Inspect current Nautobot/nctl state and deploy through the standard WFE release
   procedure, updating active CLI/executor consumers and managed cagent definitions
   as needed. Serve the normal LAN route, not a localhost-only preview.
3. Verify on the installed release: separate human/cagent cards, publication from
   cagent, detail links, next-attempt delivery and unchanged historical snapshots.
   Leave useful cagent contributions in place; remove disposable trial content.
4. Commit and push implementation, documentation and required submodule pins.
   Preserve unrelated work. Update development notes and local environment notes
   in their appropriate tracked/ignored locations.
5. Write step reports as useful and a final `report.md` with revisions, verification,
   limitations and rollback notes. Keep actual LAN URLs, machine details and raw
   evidence in ignored notes, and provide direct inspection links in the Japanese
   handoff. The Developer should be able to inspect Home, forge and the consumer
   trial without running setup commands.

Deliverable: deployed functionality ready for immediate inspection, not merely
source code or an unexecuted trial recipe.

## Focused verification

- Two managers update separate blocks without overwriting one another; two
  writers of the same block receive a conflict for an unseen revision.
- Human edits retain the block's manager and record the human as last editor.
  Subsequent cagent publication reads the current revision and preserves history.
- System/project/workflow inheritance is correct, including target-project
  delegation and explicit remote-area writes.
- A claim uses one immutable snapshot. Later changes affect the next attempt;
  metadata-only checks retain the session, while meaningful changes replace it.
- Old evidence never acquires a new observation time merely through publication.
  Partial/failed refreshes remain visible and do not erase successful information.
- Local bodies, endpoint metadata and histories do not automatically enter
  checkpoint, publication or integration artifacts. Detail references remain
  usable without relying on accidental access to host files.
- The deployed GUI supports the ordinary Simple-mode journeys and accurately
  identifies what the consumer trial was actually given.

## Scope after the first delivery

The first delivery includes explicit refresh, the generic block mechanism,
system and forge contributions, and deployed end-to-end evidence. It does not
require refreshing every project or introducing automatic recurring agents.

After the trial establishes useful content and update cost, add refresh triggers
for relevant scan/change results, new projects and meaningful project-intent
changes. Coalesce repeated events, refresh only affected contributions and avoid
publishing from a publication-triggered loop. Existing observation jobs should
not acquire an unconditional LLM call. A run starts with saved context rather
than waiting for a new cagent consultation.

## Completion criteria

- Human and agent contributions are independently stored, revised and visibly
  distinguished, including manager versus last editor.
- cagent can publish a concise system summary and relevant project information
  through the supported interface, including from the executor environment.
- Context includes useful detail/consultation routes and honest evidence ages;
  publication is never mistaken for a fresh service-health check.
- Existing p13 snapshot, delivery, inheritance and session guarantees hold, with
  no session churn caused solely by unchanged verification bookkeeping.
- Local environment content stays outside automatic Git publication paths.
- The installed LAN UI and a real consumer trial prove the behavior with direct
  links and minimal inspection effort for the Developer.
