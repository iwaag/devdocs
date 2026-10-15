# project_migration p4 — archsage and shared project access

## Goal and agreed scope

Migrate archsage into a managed project after forge. Its responsibility is to collect and understand knowledge accumulated in projects' study and wedo areas, and use it to advise people and other agents.

Basis: [braindump.md](braindump.md), the p3 migration reports, and the Developer's clarification that access to arbitrary managed projects must be a general system capability, usable by future Front as well as archsage.

Archsage:

- answers and advises from existing knowledge;
- performs study when requested, with an agreed scope;
- supplements wedo from execution records only where the owning project is not maintaining it itself;
- does not perform another project's practical work to generate or update that project's wedo.

**Project discovery and access belong to WFE, not to archsage.** Provide a common CLI/API surface for any system agent or operator using the existing operating context. Front must be able to discover projects and read their files, workflows, runs and results without archsage, its index, or its configuration. Knowledge interpretation and advice remain archsage's responsibilities.

Follow the workspace's `localrule.md`: authored files in English, steps starting at 1, no backward compatibility requirement, deployment and preparation before requesting human verification, LAN accessibility for services, and commit/push of every changed repository.

Out of scope:

- implementing Front or its request-routing policy;
- preserving the old listener, addressing syntax or logical-sage orchestration for compatibility;
- introducing a vector database, a second project registry or a general cluster synchronization service;
- replacing the existing run, acceptance and integration lifecycle;
- automatically accepting results on the person's behalf.

## Design

### Shared project access

Extend the existing WFE project registry, repository model, workspace resolver and Git facilities. Do not make agents read internal registry files or scrape the dashboard to perform supported access. Reuse existing public operations where adequate; name new commands and API routes after inspecting those operations in step 1.

The shared surface must support:

1. **Discovery:** enumerate managed projects with their purpose, root repository, available workspaces and observation status.
2. **Structure:** inspect project definitions, workflows, declared local areas, submodules and repository relationships. Cover devdocs, study, wedo and ordinary project files. An undeclared directory can still be read as a project path; do not infer that every directory is a declared knowledge area.
3. **Content:** list, search and read files at a specified project/repository revision, with bounded output and structured results suitable for agents.
4. **Execution:** find workflows and runs, inspect current run state, and locate execution records, reports and saved result files. Use the live run resolver for current state; a published devdocs snapshot is not necessarily the live run record.
5. **Provenance:** return stable project/repository identifiers, resolved commit, path and observation time. Distinguish absent, inaccessible, uninitialized and failed-to-fetch sources.

Default authoritative content reads to committed sources. Expose working-tree content only as an explicit source selection and label it as such. Fetch into system-managed caches or isolated copies, without pulling into another project's active workspace. Never fall back from a failed refresh to cached data without reporting its revision and stale status.

For submodules, distinguish the commit pinned by the project from the current upstream branch tip. Default project-content traversal to the pinned commit; allow explicit upstream inspection for knowledge refresh. A request that requires a particular commit must check that the retrieved source contains it and report failure if it does not.

Use existing authentication and operating boundaries. General agent access does not mean exposing credentials, ignored host configuration or arbitrary host paths as project content. No new approval flow is required for ordinary supported reads. Keep repository credentials and cache locations out of portable records.

### Knowledge ownership and write-back

The source project or shared repository remains the authority. Archsage maintains a small portable index of useful domains, project/repository/path references, applicability, observed revisions and refresh outcomes. It does not maintain a second authoritative copy of all knowledge. Local fetch caches are disposable.

Treat devdocs as evidence about intent, decisions and execution. Distinguish findings from unaccepted runs, accepted results, and integrated knowledge. An agent's report alone does not prove a practical result was verified.

All changes use isolated run copies and the existing save, acceptance and integration path:

- **Shared study/wedo repositories:** bind the selected repository as editable in the appropriate workflow; save on run branches and integrate after acceptance.
- **Study/wedo inside another project's root:** use a run owned by that project, with archsage's task guide and execution configuration where supported. Verify how the requested task reaches that run and how its result is returned. If a common capability is missing, add it to WFE rather than bypassing integration with a direct push from archsage.

Resolve the concrete route in step 1. The route must work with current run allocation and immutable workflow bindings; do not assume a running consultation can acquire new editable repositories. A new task or workflow definition may be needed before allocating the writing run. Do not implement a parallel cross-project job lifecycle.

An update to a shared repository and an update to consuming projects' gitlinks are separate results. Archsage must report which occurred. Do not silently advance every consumer to a new revision.

### Archsage project and workflows

Create a managed project provisionally named `pj-archsage`, project id `archsage`, with a project guide, task guides, a knowledge index, and normal devdocs/workflow/run storage. Finalize paths against the installed project contract.

| Workflow | Responsibility | Mutation boundary |
| --- | --- | --- |
| `consult` | Read relevant sources and give advice through talk, citing evidence and identifying unknowns | Consultation records only; source knowledge is readonly |
| `study` | Agree a bounded investigation, research, record findings and present the result | Selected study destination through its owning run |
| `backfill-wedo` | Check eligibility, inspect execution evidence, supplement missing practical knowledge and present the change | Selected wedo destination through its owning run |

The last two are logical task types; their concrete workflow owner follows the destination's storage form. Avoid creating two independent runs that both claim ownership of the same deliverable.

Knowledge gaps found during consultation become explicit proposed research questions. Do not turn a consultation into unbounded research. Refresh the index explicitly or as bounded preparation for a task; a separate refresh workflow is optional if actual operation needs it.

Write all reviewed artifacts before the final result-confirmation talk. Configure documentation generation so no report is created after acceptance. Preserve the distinction between completion, acceptance, saving and integration demonstrated in p3.

Initially use one agent with direct access to relevant knowledge. Preserve useful existing domain guides and unanswered questions, but do not require the old logical-sage selector or nested agent calls.

### Wedo supplementation

Make maintenance responsibility explicit for each managed target: project-maintained or archsage-supplemented. Unknown ownership is not permission to edit. Do not infer abandonment from the age of the last commit alone. Store this policy with the target's portable project documentation or configuration; decide the smallest sufficient contract in step 1.

Before supplementing:

- identify the relevant completed execution records and their acceptance/integration state;
- check whether the project has already recorded the same knowledge;
- extract only procedures, constraints, failures and verification supported by those records;
- record originating run references, source commits and the target base revision;
- identify processed evidence so repeating the task does not duplicate content.

Recheck ownership and current target content before integration. A project may update wedo after archsage prepared a change; a clean Git merge alone does not detect semantic duplication. Establish a common pre-integration check or equivalent target-version precondition if current integration cannot enforce this condition. If the prerequisite changes, stop publication, reassess the difference and present a revised result as needed. Never overwrite project-maintained knowledge merely because the merge is technically possible.

## Step 1 — inspect current contracts and migration sources

- Inspect repository status, applicable local instructions, the installed release and current source contracts before making changes.
- Inventory existing archsage definitions, study sources, useful guides, unanswered queues, active conversations, scheduled synchronization and callers of the old listener. Record a disposition for each active dependency.
- Inspect registry/dashboard, local-area discovery, content reads, run resolution, storage, delegation if relevant, and integration. Identify exactly which CLI/API operations can be reused and which are missing.
- Select representative managed projects covering directory-mode devdocs, submodule-mode devdocs, local study/wedo, and shared study/wedo repositories. Include an unavailable source and a project without a local workspace.
- Resolve writing-run ownership for both shared and root-local knowledge. Resolve maintenance-policy storage and the integration-time supplementation check.
- Use current operational information for deployment decisions; consult Nautobot/nctl before changing or retiring registered services.

Deliverable: `report1.md`, with the capability/gap matrix, concrete API/CLI proposal, write-back route, migration inventory and bounded trial cases. Host details remain in ignored local notes.

## Step 2 — implement and verify common project access

- Add the missing discovery, structure, content and execution-read operations in WFE, using shared server operations behind CLI and API.
- Support normal directories and submodules, remote-only registered projects, explicit revisions, pinned versus upstream content, and reliable provenance/error states.
- Document the public contract and provide discoverable CLI help. No archsage-specific source map is required to find managed projects.
- Implement only the common write-path or integration prerequisite gaps established in step 1. Reuse existing ownership, acceptance, non-forced push, integration journal and recovery behavior.
- Add focused tests for revision resolution, local-area discovery, remote-only access, unavailable sources, live-run versus published-record selection, and changed supplementation prerequisites. Verify reads do not alter another workspace.
- Run required WFE checks for changed components. Prove the common read interface from a plain CLI/API client with no archsage project or index configured.

Deliverable: `report2.md`, public interface documentation and a tested common capability usable by future Front.

## Step 3 — build archsage and migrate useful knowledge references

- Create the managed project and its guides, index and workflows. Import useful existing domain descriptions and unresolved questions with provenance.
- Declare source knowledge readonly for consultation and narrowly bind writing destinations for study/backfill runs. Ensure general maintenance workflows do not accidentally become an alternate unrestricted wedo-writing route.
- Record maintenance responsibility for trial targets and implement evidence-based, repeatable supplementation.
- Ensure agent guidance distinguishes sourced facts, inference, unknowns and verification limits. Include how to use shared project access and how to report missing/stale sources.
- Validate definitions against the installed contract or the release candidate containing step 2 changes. Test reference lookup from a run copy without relying on Git to copy ignored local files.

Deliverable: `report3.md`, valid project definitions, seeded references and a reproducible migration procedure.

## Step 4 — deploy and prove the workflows with real runs

- Commit and push the required changes, deploy the WFE release, and verify that the service and CLI use the same release. Back up operating stores before any required migration.
- Register/obtain archsage and prepare execution context and trial targets. Prepare direct result-viewing links before asking the person to verify anything. Services must be accessible from the home LAN.
- Execute bounded trials through the actual executor:
  1. Cross-project advice citing both study and wedo, with source revisions and an honest unknown.
  2. Requested study that reaches its destination through acceptance and integration; a subsequent consultation sees the integrated revision.
  3. Wedo supplementation from actual execution records for an explicitly eligible project.
  4. Repeated supplementation producing no duplicate knowledge, and a project-maintained target producing no edit.
  5. A target update or ownership change between preparation and integration causing reassessment rather than stale publication.
  6. Write-back coverage for both shared repositories and root-local knowledge. Cases may be combined with the study/backfill trials.
- Verify results from files, revisions and integration records, not only agent claims. Keep the person's decisions as their own. Document operator-issued requests and manual steps accurately.
- Recheck the general CLI/API independently of archsage after deployment. Front implementation is not needed for this proof.

Deliverable: `report4.md`, with runs, source/result revisions, acceptance/integration evidence, observed limitations and prepared human review paths.

## Step 5 — retire obsolete paths and report completion

- Apply the dispositions from step 1 after accounting for outstanding work. Preserve historical conversations, useful evidence and source repositories.
- Retire obsolete archsage listeners and synchronization jobs when their callers are handled. Update registered desired service state and verify actual state; do not claim retirement from a stopped process alone.
- Update documentation and callers that must change for this migration. Record any remaining Front dependency explicitly without adding compatibility shims by default.
- Write `report.md` describing the common access contract, archsage's demonstrated responsibilities, write-back ownership, operating procedure, remaining limitations and future Front usage.
- Review diffs for unintended changes and host details. Commit and push all changed repositories, including changed superproject pointers where applicable, while preserving unrelated work.

Deliverable: a functioning managed archsage, a reusable system project-access capability, and an explicit legacy disposition.

## Completion criteria

- Any existing system agent/operator can discover and inspect managed projects through the documented CLI/API without archsage configuration.
- Content reads identify source revisions and failures; directory and submodule storage both work, and active workspaces are not mutated by reads.
- Archsage provides evidence-backed cross-project advice and performs requested study through normal managed runs.
- Wedo supplementation is limited to eligible targets, traceable to execution evidence, repeatable without duplication, and revalidated before publication.
- Both shared and root-local knowledge have a demonstrated accepted write-back route; no direct-push exception bypasses result ownership.
- The required release and project are deployed, verification evidence is retained, and old services/dependencies have a documented disposition.
