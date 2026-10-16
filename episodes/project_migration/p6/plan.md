# project_migration p6 — shared data and evolving workflows through playprey

## Basis and intended result

Implement [braindump.md](braindump.md): bring `pj-playprey` into the managed project system and use real model-onboarding work to prove large-file exchange and evolving, visible workflows. The project's durable product is reusable methods, tools and knowledge for producing interactive scenes from supplied models, including delivery to `pj-protoprey`. Finished scenes are deliveries and evidence, not the purpose of an ever-growing Git asset collection.

The discussion selected cagent as the operator for shared-storage installation. Prepare the missing deployment tools, then let cagent investigate placement, propose the concrete change, install it and verify it through its managed `change` workflow. Do not have Omni install the service in advance and describe the subsequent registration as a cagent installation.

Follow `devpolicy/styles.md`, `devpolicy/terms.md`, `devdocs/README_DEV.md`, the project's ignored environment notes and `localrule.md`. Generated documents are English. Do not preserve backward compatibility by default during structural changes. Services and review surfaces must be usable from the home LAN. Complete agent-performable preparation before requesting human verification. Commit and push changes in each affected repository, then update and push parent submodule pointers where applicable.

This document is an implementation plan, not evidence of deployment or agreement to a particular cluster change. Host names, addresses, actual mount paths, storage inventory, credential locations and raw operational evidence belong in ignored environment notes. Published episode reports use logical resource names and portable references.

## Findings to verify at execution

- The inspected playprey source has about 2.93 GB of file contents in its current Git tree, including textures, screenshots and large diagnostic JSON. This is neither history size nor compressed transfer size. Its ignore rules exclude GLB and Blender files, so the repository alone does not establish that the original inputs or required build checkpoints are available.
- `devdocs/episodes/casestudy_and_toolify/guide.md` describes onboarding through inspection, construction, registration, fitting and handover, with lessons from wolf, cheetah and eagle cases. It explicitly permits judgement and iteration. Model configuration, presentation, anatomy/capabilities and human adjustments are established contracts to understand before migration.
- WFE supplies isolated run copies, frozen definitions, definition revisions for future work, dependency branches/joins, delegation and result confirmation. Its graph is acyclic; dependency branches are not conditional routing. Current adjustment rules restrict changes to started work, bindings/execution and delegates.
- Run copies do not carry large ignored resources automatically. Existing artifact attachments are project-relative; presentations copy their content into published run records and currently have a 200 MiB limit. These are not suitable as the transport for large external assets.
- cagent has a plan-gated, host-scoped Ansible playbook path and an effects ledger. The inspected deployment had no configured playbook entries and no NFS-specific deployment implementation in the searched backend sources. Tooling preparation is therefore part of this phase.

## Scope and ownership

| Owner | Responsibility |
| --- | --- |
| pj-clusterintent and relevant submodules | Reusable NFS server/client deployment, inventory integration and the observations needed to explain service state |
| Managed cagent | Placement investigation, concrete change plan, installation, verification and operational records |
| WFE / pj-agdev | Portable shared-resource references, run input/output metadata, external result identity and usable review links |
| Managed playprey | Model-processing tools and configuration, workflows, reusable knowledge and delivery contracts |

Start with one NFS-backed shared area. Keep physical mounts host-local and expose a logical resource name to projects. Google Drive mounts, a general storage-provider framework, distributed execution scheduling, transparent migration of a live run between hosts, automatic garbage collection and a generic looping workflow engine are outside the initial implementation. Two-host storage verification is required; it does not require a second WFE executor.

NFS provisioning follows cagent's existing Developer plan-agreement contract. The actual host/disk choice is made after investigation, not inferred from this plan. Existing destroy-class rules remain applicable if such work becomes necessary; ordinary installation is agent-operated. Unknown external effects are investigated from records and observations rather than blindly repeated. Dependency hosts use the ordinary change path.

## Step 1 — inventory source assets, runtime requirements and implementation seams

1. Record repository state, source/deployed revisions and applicable instructions. Use Nautobot/nctl for current cluster facts, including observation ages and unavailable hosts. Keep raw evidence under the phase's ignored local evidence directory.
2. Inspect playprey's current tree and Git history separately. Classify source, authored settings, reusable tools, original inputs, necessary intermediate checkpoints, regenerable outputs, large evidence and obsolete cases. Check ignored files in the original working environment; do not conclude that they are recoverable from Git. Identify missing assets and their owner.
3. Read the onboarding guide, current model registry, generators, launch paths and recent handovers. Identify one reproducible existing case and one structurally different case. Preserve the project's distinction between mechanism correctness, human tuning and later optimization; do not make this migration an exhaustive profiling exercise.
4. Inspect cagent's deployed tools, execution environment and playbook configuration, including inventory selection, privilege requirements and how new tools/configuration reach a run copy. Identify the smallest backend changes required for NFS installation and observation.
5. Inspect WFE's run copies, request intake, artifact paths, presentation manifests, confirmation subjects and integration. Specify where external references enter the request and how all UI/CLI/agent readers will resolve them without loading asset bytes into the editor.

Deliverable: `report1.md` with a portable asset classification, capability/gap matrix, proposed trial cases and repository ownership. Acceptance: necessary inputs are located or explicitly missing, current behavior is distinguished from proposed behavior, and the storage and review seams are concrete.

## Step 2 — prepare NFS deployment tools for cagent

1. Implement rerunnable Ansible server/client roles or playbooks in the existing deployment repository. Cover the selected server OS and actual client OSes identified in Step 1; do not add hypothetical platforms.
2. Express the backing directory, exported share, client scope, ownership/group mapping, mount options and persistent or automatic mounting through inventory/configuration. Preserve existing exports and mounts outside the change's scope. Keep large intermediate work local to each run.
3. Add discoverable usage, preview/check instructions and verification commands. Verify actual inventory target resolution and required privileges before dispatch. A successful empty target selection is not a successful installation.
4. Configure the playbook entry points used by cagent. Use the existing agreed-plan and ledger path. Record the deployment source revision and effective settings for reproducibility; if those can change between agreement and dispatch, bind or revalidate the approved inputs instead of executing different content under the same filename.
5. Provide the minimum observation support needed to record the NFS service and client mounts honestly. Distinguish configured, running, mounted and successfully usable states. A service declaration alone is not an end-to-end health check. A dedicated automatic NFS reconciler is not required for this phase.
6. Run relevant role/backend checks, including repository-mandated gates for touched modules. Commit/push and deploy the tools and configuration before starting the installation trial.

Deliverable: `report2.md` and usable server/client deployment tools. Acceptance: cagent can discover and execute the prepared operations through its existing change path; no production NFS installation has been hidden in tool preparation.

## Step 3 — have cagent plan, install and verify the shared area

1. Request the shared-storage outcome through managed cagent. Have it investigate suitable placement, persistent capacity, client availability and existing storage. Compare an existing host with a dedicated VM only as relevant to the actual environment; do not assume a new guest is necessary.
2. Have cagent prepare the concrete change record and plan: server, backing storage, client hosts, exports/mounts, permissions, declared state, ordered operations, verification and recovery. Preview applicable desired-state changes and name the deployment inputs.
3. Present the complete plan through the normal cagent plan question for Developer agreement. This is the agreement to actual infrastructure effects; do not request a second generic permission for preparatory reads or already authorized work.
4. Let cagent apply the agreed operations, record effects and verify the result. Omni repairs demonstrated tool defects if needed, but attributes those interventions explicitly rather than claiming autonomous success.
5. Test from at least two client PCs, using the actual agent execution identity on the execution host: directory traversal, read/write/rename, a representative large-file transfer with matching hashes, visibility from the other client and remount behavior. Exercise automatic mounting in the service context that will use it, not just an interactive terminal.
6. Record the installation and purpose in cagent and the appropriate Nautobot desired/observed state. Explain any observation gap. Retain the agreed service for p6 use; clean up disposable transfer probes.

Deliverable: `report3.md`, cagent run/ledger references and fresh verification. Acceptance: the shared area is genuinely usable from two hosts, the installation was performed through cagent, and operational records identify what was configured and what was tested.

## Step 4 — implement portable input and external artifact references

1. Define a small reference contract, for example a logical share plus relative path. Treat the example spelling `shared://playprey/inputs/case-a/` as a proposal, not an existing API. Resolve it through ignored host configuration; tracked records contain no physical mount location or credentials.
2. Preserve the human interaction: place a folder in shared storage and describe it in the request. Do not require a rigid per-file upload form. Add only the UI needed to show references and availability; metadata reads are bounded and heavy previews load on explicit request.
3. When a run selects its inputs, generate a manifest of the actual consumed files with relative paths, sizes and content hashes. Keep the original folder reference and explanatory request. Establish stable input bytes through an immutable published input set or verified local staging; record missing or changed inputs instead of silently consuming a different set during resume.
4. Stage expensive processing under the run's `.local/`. Publish outputs and selected large evidence under a run/candidate-specific destination. Incomplete copies must not appear as completed publications: finalize the manifest only after content verification, and make retries find the same publication or create a clearly distinct candidate.
5. Add a first-class external artifact reference to run records and reads. Keep manifests and concise reports in Git; keep payloads outside Git. Do not turn the existing project file API into arbitrary host filesystem access.
6. Bind result confirmation to the exact external output manifest/content identity alongside the existing result subject. Verify availability and integrity when preparing the result for confirmation. A replaced output must be presented as a new revision, and an unavailable or altered artifact must not be described as the verified result.
7. Provide LAN-usable review/download access resolved from runtime configuration. Avoid routing multi-hundred-megabyte payloads through editor state or copying them into the existing Git-backed presentation bundle. Small previews can continue using the existing presentation mechanism.
8. Test the meaningful boundaries: different physical mount paths for the same logical reference, missing mount, changed input, interrupted publication, concurrent candidates, external-result revision changes and unchanged ordinary project artifacts. Deploy the common-system changes and verify the CLI/UI/agent paths against the running service.

Deliverable: `report4.md`, the reference/manifest contract and deployed operations. Acceptance: a request can name a folder, the run records exactly what it used, and a reviewer can reach and identify the delivered bytes without a heavy editor or Git payload copy.

## Step 5 — migrate playprey into a knowledge-centered managed project

1. Register the project through the supported project workflow. Keep source, authored `config.json` and `presentation.json`, generation recipes, reusable tools and knowledge under Git. Declare local knowledge areas where sufficient; separate repositories only when sharing requires them.
2. Move original inputs, large outputs and selected bulky evidence to shared storage after verifying copies. Keep provenance and update references. Put intermediate files, caches and temporary diagnostics under `.local/`. Preserve small evidence where it helps explain a method; do not classify every JSON as source or every image as disposable.
3. Resolve Git history size explicitly. Removing files from HEAD does not remove old blobs. Based on the inventory, choose a clean managed import with preserved archival history or a coordinated history rewrite. Prepare and record the concrete retention/mapping decision before any destructive source-history change; do not silently force-push the original repository.
4. Update generators, resource lookup and launch paths to use staged or resolved assets. Preserve declared model frames/capabilities, reproducible parameter inputs and human-authored adjustments. Regeneration must not overwrite a person's tuned configuration without an explicit operation that makes that consequence clear.
5. Extract the working onboarding guide and reusable tools into discoverable maintained locations, retaining links to the cases that justify each lesson. Do not turn case-specific instructions into universal anatomy rules or maintain retired/dormant features merely because they were imported.
6. Define a portable delivery manifest for a consumer such as protoprey: required assets, configuration, code/tool revision and launch/use instructions. Delivery must not depend on paths inside the producer's run copy.

Deliverable: `report5.md` and the managed project. Acceptance: a clean run copy can obtain its declared inputs and reproduce the selected baseline; required data and knowledge survive the migration, with an explicit history disposition.

## Step 6 — author visible, adaptable workflows

1. Create a model-onboarding workflow from the existing guide: inspect inputs and applicability, construct, register, fit requested motion/interactions, verify, record reusable findings, present and confirm. Static construction and full interaction transfer are different scopes; the workflow must allow the requested scope to determine the work.
2. Provide an exploratory workflow for developing or comparing methods and feeding proven lessons back into tools and knowledge. Reuse an ordinary freework workflow where it already expresses this purpose; avoid proliferating workflows without a distinct outcome.
3. Use visible nodes for meaningful investigations, alternative approaches and decisions. Keep fine parameter iteration inside a node with evidence. Use current adjustments at node boundaries to add repair/comparison/reverification work. Do not claim conditional execution, arbitrary loops or concurrent executor processes from dependency edges alone.
4. Establish version handling: normal definition improvements apply to new runs; deliberately coexisting approaches have distinct workflow IDs; request-specific changes remain run revisions. Record base versions and candidate lineage. Use successor runs when a change exceeds adjustment capabilities, rather than rewriting completed work.
5. Test existing adjustment/delegation limits before proposing common-engine changes. Add only a narrowly evidenced capability if the actual case cannot be expressed adequately; document the limitation and the chosen implementation.
6. Record applicable anatomy/input conditions, failed assumptions, candidate comparison, adopted configuration, unresolved limits and evidence references. Complete knowledge/report writes before final result confirmation. Later human tuning is a subsequent recorded input or follow-up run, not an unrecorded alteration of an accepted delivery.

Deliverable: `report6.md`, workflows and agent guides. Acceptance: a reviewer can see what was tried and why the plan changed, while the agent retains judgement within the requested scope.

## Step 7 — run end-to-end trials and prepare human review

| Trial | Required demonstration |
| --- | --- |
| Existing case | A human-style request supplies multiple large input files by folder reference; a clean managed run reproduces an evaluable baseline and publishes portable outputs. |
| Different anatomy | A structurally different case exposes assumptions, records method choices and adds meaningful future work through visible revisions or a justified successor run. |
| Coexisting approaches | Two named approaches/candidates remain separately identifiable; compare results without overwriting inputs, outputs or execution history. |
| Cross-host delivery | Another PC resolves the same logical references and verifies/uses the output without access to the producer's `.local/` tree. |
| Resume and publication | A bounded interruption or test fixture proves that resume preserves input identity and does not expose partial or mixed output. |
| Knowledge reuse | A later run uses a lesson/tool produced by an earlier case, citing its evidence and applicability instead of merely copying its settings. |

1. Prepare the requests, observation/evidence collectors and expected outcomes before trials. Use the deployed WFE executor and record source/releases, run IDs, costs, elapsed time, defects and Omni interventions.
2. Run automated model/runtime checks appropriate to the cases, including reset, configuration persistence and required mechanisms. Preserve the source project's distinction between provisional tuning and broken functionality. Fix demonstrated defects and repeat affected checks.
3. Prepare a lightweight LAN review page with explanation, candidate identity, screenshots/video where useful and output access. The existing application uses a browser editor plus native Godot; shared storage alone does not make it a browser-playable application. Prepare the actual launch environment and minimal launch action on the review PC before asking for experiential feedback.
4. Ask the Developer only for the human judgement that remains: whether the mechanism is evaluable, which candidate to prefer and what should change. Keep operational tests and setup with the agents. Record feedback and acceptance through the existing run conversation and confirmation path.
5. Demonstrate the delivery contract in an isolated consumer check or actual protoprey integration as appropriate to the agreed scope. Verify that source/run checkout paths are not needed. Do not claim production consumer integration from a download-only test.

Deliverable: `report7.md` with trial evidence and dispositions. Acceptance: all trial behaviors have evidence, any remaining limitations are explicit, and human review is prepared against deployed results.

## Step 8 — document operations and close the phase

1. Document how a person provides a folder, how an agent resolves it, where intermediates/results/evidence belong, how to retrieve an accepted output and how to report an unavailable share. Explain who owns retention; do not delete accepted outputs or irreplaceable inputs as incidental cleanup.
2. Update project guides, shared development documentation where appropriate and ignored host environment notes. Record NFS server/client lifecycle and recovery in cagent's operational records. Keep physical infrastructure details out of shared episode documents.
3. Write `report.md` with delivered behavior, deployed revisions, asset/history disposition, trial results, costs, interventions, cleanup and named follow-ups. Distinguish proven two-host storage access from deferred distributed execution.
4. Review diffs for unrelated edits, local environment information, credentials and unsupported claims. Run required checks for changed components, commit/push every intended repository change and verify remote branch correspondence. Report blockers specifically rather than claiming an unpushed or undeployed change complete.

## Completion criteria

- NFS deployment tools are available, and managed cagent performs and verifies the actual installation under its agreed change plan.
- Two client PCs can use the shared area under the intended execution identities, with recorded content verification and mount behavior.
- Large inputs and outputs travel outside Git while portable manifests tie them to requests, runs, candidates and accepted results.
- playprey is a usable managed project; clean run copies have a reproducible route to required resources and retain human-authored settings.
- Real onboarding work demonstrates changing and coexisting workflows, with visible decisions and reusable knowledge supported by evidence.
- Review is usable from the LAN, the native runtime review is prepared, and human feedback concerns the result rather than unfinished setup.
- Relevant changes are tested, deployed where required, committed and pushed; operational records and final reporting match what actually happened.
