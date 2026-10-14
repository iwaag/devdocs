# project_migration p3 — forge as a managed project

## Goal and agreed scope

Move forge's two responsibilities into a project managed by the workflow editor:

1. Create or modify media assets, including text, using existing knowledge. When a request cannot be fulfilled with the available capability, explain why and preserve the demand.
2. Discuss those demands with the person, then extend capability through research, tested procedures, workflow design and proposals for resources.

Basis: [braindump.md](braindump.md), the p1/p2 migration reports, and the person's scope clarification on 2026-10-14. The first project is provisionally named `pj-forge` (project id `forge`). Existing forge structure and compatibility are not requirements; useful generation tools and proven knowledge can be reused.

**This phase proves forge's work, using local files and direct tools where necessary.** A manually maintained local source map, a local environment document, explicit file handoffs and host-operated request/collection commands are acceptable. They must be documented and reproducible by the next operator; they need not become a general system.

Out of scope:

- implementing `delegate` nodes or generic parent/child workflow orchestration;
- designing cluster-wide context discovery, distribution, synchronization or access control, including shared ComfyUI context;
- Front's production request routing, callbacks, cross-project acceptance and other operational integrations;
- migrating archsage, Front, autolab or cagent;
- a general external-job scheduler, a new notifier service or a second execution host;
- backward-compatible old forge APIs, topic names or listeners.

Delegate, shared context and Front integration belong to separate episodes developed alongside Front. Record concrete requirements found here for those episodes, without implementing them as prerequisites to forge's trial.

## Working design

### Project and workflows

Use the installed project/workflow contract, run copies, plan agreement, questions, result acceptance and integration. Do not recreate forge's old state machine in parallel.

Proposed project contents:

```text
project.yaml
agent/guide.md
agent/provide.md
agent/develop-capability.md
agent/refresh-catalog.md
catalog/                     capability descriptions and source references
requests/                    one unmet-demand record per originating run
tools/                       reusable generation, collection and delivery tools
devdocs/workflows/            provide, develop-capability, refresh-catalog, freework
devdocs/runs/                 system-managed execution records
.local/localenv.md            ignored host facts and resource instructions
.local/sources.toml           ignored source checkout and tool locations
```

These are proposed areas, not a new storage schema. Validate their declarations against the installed editor before finalizing the layout. Specialized projects remain the authority for their knowledge and implementation; forge indexes and uses them. Do not copy every source repository into forge.

| Workflow | Work | Result |
| --- | --- | --- |
| `provide` | Assess the request and available capability; create/modify and check an asset, or record the missing capability; present the outcome | Delivered asset and usage information, or an explicit explanation and demand record |
| `develop-capability` | Review recorded demand with the person; research and implement an agreed improvement; verify and update the catalog | A demonstrated capability, or an evidence-backed limitation and next proposal |
| `refresh-catalog` | Read selected source projects and update references, applicability and verification evidence | A reviewed catalog update |
| `freework` | Use the standard basic workflow for other discussion and maintenance | Request-specific result |

Use ordinary study/do/talk nodes and the system's plan step. Exact node definitions follow validation in step 2. Result confirmation must distinguish acceptance of an explanation from successful asset delivery. Providing no asset is not automatically an execution failure; an interrupted process is not an intentional unsupported outcome.

`provide` does not silently turn into unlimited research or environment construction. Missing capability is recorded and discussed in a separate `develop-capability` run. A later improvement links back to the original demand; retrying that demand is a new explicit request.

### Knowledge and local context

Start with readable catalog entries and a small source map. Each capability names its supported outputs and edits, required inputs, limitations, source project/path/revision, execution instructions, resource requirements and verification evidence/date. Separate knowledge of a method from proof that it works in this environment. The agent chooses the relevant capability; the index is a discovery tool, not an automatic routing policy.

Host-specific endpoints, paths, model/workflow availability and credential references belong in ignored local files. Commit only portable descriptions and examples. Guides explain how a run reads the project workspace's local context; run copies must not assume `.local/` was copied by Git. Record source revisions used, and avoid claiming a historical verification proves current availability.

Start with selected relevant sources, including existing forge/mediagen knowledge and the migrated asset project where useful. Read sources directly through the documented local map. Refresh explicitly through `refresh-catalog`; no automatic subscription or synchronization service is required.

### Demand records and delivery

A demand record holds the originating run reference, requested outcome, relevant constraints, what was tried, missing capability, evidence and subsequent improvement/retry references. Use separate files per request to reduce concurrent editing conflicts. Keep the originating run as evidence; shared records become ordinary accepted project changes. Before integration, operators can find the record from the run and its saved branch. Do not require a new cross-run index service.

Reuse working generation, transformation, upload, fetch and URL-renewal code where practical. Separate those tools from old listener/role-runner dependencies. Delivery records include a durable object key or artifact path, content hash, generation conditions and a way to obtain a current link; an expiring URL alone is not a durable delivery.

For an external job, save its job id and output references before collecting results. Prefer bounded foreground execution for the initial trial. If work must outlive an attempt, use existing run waiting/resumption plus a documented operator collection/resume step. Do not promise automatic callbacks. Check the saved job before resubmitting after interruption. Any new helper is a narrow tool for this trial, not a generic scheduler.

## Step 1 — inspect the migration boundary

- Read the current forge code, tool guides, knowledge configuration and the installed WFE contract. Inspect repository status before touching existing work.
- Use Nautobot or `nctl` to establish current service/resource state, then check only the actual generation routes needed for the trial. Historical environment notes alone are not proof of availability.
- Inventory unresolved forge requests, external jobs, artifact storage and callers of the old listener/API. Identify what must finish in place, what will be explicitly re-requested and what is historical.
- Select reusable tools, a small knowledge-source set, one inexpensive supported media route and one bounded capability extension. Record the execution budget and required resources in the trial plan.
- Identify any actual contract gap. Resolve it with local files/direct tools where possible; report excluded infrastructure needs as follow-up requirements.

Deliverable: `report1.md`, with the reuse decisions, trial cases and explicit disposition proposed for old work. Host details stay in ignored local notes.

## Step 2 — build the project and validate its definitions

- Create the project in an isolated migration checkout, preserving useful source history or provenance when extracting code.
- Add the workflows, guides, catalog, demand-record format, tool entry points and portable configuration examples.
- Seed the catalog from the selected sources with exact references. Unsupported or unverified methods remain visibly unverified.
- Ensure agents can learn tool usage from `--help`; guides describe responsibilities, available context and outcomes rather than copying CLI manuals.
- Validate with the installed release in a scratch registry/workspace. Verify workflow bindings, local-context lookup from a run copy and the supported/unsupported result paths.
- Run focused existing tests for reused or modified tool behavior. Add tests only for meaningful new failure boundaries, such as collecting the same external job after interruption.

Deliverable: a valid project ready to register, and `report2.md`. No delegate or shared-context infrastructure is added.

## Step 3 — register, prepare and deploy the trial

- Commit and push the project, register and obtain it through WFE, and verify its request channels and executable workflow configuration.
- Install the selected tools and prepare ignored local context in the obtained workspace before any run. Verify it is readable from the run execution environment without publishing its contents.
- Use the existing browser/Zulip request interface or an explicit operator CLI request. Do not add Front routing for this phase or impersonate the person's approval.
- Prepare a concrete result-viewing path. Any service started here must be reachable from the home LAN, following `localrule.md`.
- Deploy any necessary narrow system fix before asking the person to check it. If no system change was needed, use the existing release.

Deliverable: a usable forge workspace and `report3.md`; the person can inspect the prepared run/result with minimal operations.

## Step 4 — prove both responsibilities through real runs

Perform and retain evidence for:

1. **Supported requests:** a text asset and one available non-text media route, including an edit to a supplied or previously generated asset. Verify the delivered files, constraints and durable references, not just the agent's claim.
2. **Unsupported request:** explain the missing capability and save a discoverable demand record. End the request cleanly without pretending that an asset was delivered or leaving an unexplained running node.
3. **Capability extension:** review that demand with the person, agree a modest improvement, execute `develop-capability`, and record verification evidence in the catalog. Use a local tool/procedure improvement if the larger request requires excluded infrastructure.
4. **Subsequent provision:** explicitly request use of the newly verified capability and link the new outcome to the original demand. Demonstrate the actual improvement; document any remaining part of the original request that is still unsupported.
5. **Interruption boundary:** for the chosen external generation route, prove that a saved job can be inspected/collected without duplicate submission or delivery. A documented manual resume is sufficient.

Use normal run records for questions and acceptance. An operator may pass files or issue collection commands; record exactly what was manual. The Omni Agent must not silently create the agent's deliverable or claim a manual handoff was automated. When doing an in-system agent's work, add the required one-line handoff-candidate note to the episode report.

Deliverable: `report4.md`, linking runs, outputs, demand records, catalog changes and the person's actual decisions. Acceptance and shared-branch integration are checked separately.

## Step 5 — settle the old forge and report the boundary

- Apply the explicit dispositions established in step 1. Preserve old conversations, useful evidence and durable asset objects; do not reinterpret old requests as new runs without recording the handoff.
- Retire obsolete forge listeners/services when their outstanding work and callers have been accounted for. Do not create compatibility shims. If a caller must wait for the Front episode, document that dependency and whether its old route remains operational or is explicitly discontinued; do not claim full system cutover.
- Update the current development documentation and ignored environment notes to describe the new project, local context and any remaining old service.
- Write `report.md`: results, reused/deleted components, demonstrated limitations, manual operating procedure, remaining old dependencies and the next-episode requirements.
- Commit and push every changed repository, including changed superproject pointers where applicable. Review exactly the intended diff; preserve unrelated work.

Deliverable: a demonstrated managed forge, an explicit legacy disposition and a handoff suitable for the Front episodes.

## Verification and completion

This phase is complete when both responsibilities work through managed runs: supported creation/modification is delivered, unsupported demand is retained and discussed, and an agreed capability improvement leads to a demonstrated subsequent provision. Source evidence, local resource lookup, durable asset retrieval and acceptance/integration are traceable. Manual steps are acceptable and documented.

Before publishing, check newly authored files for host details and secrets. Run WFE validation for the new project; run the editor's required checks and affected-project validation only if the editor changes. Verify service state through Nautobot/nctl before and after relevant deployment or retirement. Do not launch unrelated migrations or repeat broad checks without a new reason.

The final follow-up list separates (a) delegate execution and parent/child result ownership, (b) shared cluster context including ComfyUI resources, and (c) Front routing and production operations. None is a completion condition for p3.
