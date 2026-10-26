# Project cooperation p1

Status: in execution. This document plans the implementation and trials; step reports (`reportN.md`) record what happened.

Step 1 findings (report1.md): the selected case is protoprey's forest. gimmickimg's waterfall-loop animates the Mushroom Waterfall location (its source image is byte-identical to protoprey's location view), and playprey's `wolf` profile becomes the `mouth` step of the wolf predation event. Gaps G1–G10 are listed there.

## Objective

Establish a practical cooperation pattern using protoprey, gimmickimg and playprey, then document the parts that should become the system-wide standard. Protoprey owns this integration; no new aggregation project is needed.

A protoprey scene-making run can discover useful work in the other projects, reuse an existing result or commission a workflow, receive a verifiable delivery, and integrate it into the game. Each producer retains its broader purpose. Protoprey can replace a producer or stop using it without losing the ability to maintain and run the adopted result.

Independence means independence from a producer's working checkout, running services and future changes. Adopted code and formats still have explicit, versioned dependencies.

## Execution policy

Follow `localrule.md` throughout execution:

- Write maintained files in English and communicate with the Developer in Japanese.
- Carry out the necessary implementation, verification, commits, pushes, deployment and LAN-host operations autonomously. Do not introduce approval, agreement or acceptance gates between steps. Record decisions and evidence; human review is available without blocking technical progress.
- Backward compatibility is not required in this experimental phase. Update affected callers, data and documentation together when changing a contract.
- Complete agent-operable setup and checks before handing the result to the Developer. Leave the integrated result deployed or otherwise directly launchable with few actions. Serve any review service needed by this work on the LAN, with a usable address.
- Before reasoning about or changing local services, consult Nautobot or nctl using `pj-clusterintent/nctl/README.md`. Distinguish recorded observations and their ages from current service checks.
- Preserve unrelated work. Machine addresses, host paths and credentials belong in ignored local records, not tracked documentation.

The steps below describe outcomes and an initial order, not mandatory routes for in-system agents. Adjust the work when a trial supplies evidence. An existing mechanism that already meets a requirement should be demonstrated directly.

## Starting evidence

The proposal inspected published main through `wfe projects`, because some local workspaces lagged behind it:

- protoprey `de32d81fb3`: its project goals already require deliveries by name and hash that work without the producer checkout. Its workflow entry is `freework`.
- gimmickimg `35b73c0960`: experiments and a Godot landscape effect exist; the workflow entry is `freework`. A consumer-facing delivery contract was not found in the inspected material.
- playprey `c1112b64eb`: `onboard`, `compare`, `boast` and `freework` exist. `knowledge/delivery.md` describes a standalone delivery with code, configuration, assets and hashes. It explicitly leaves embedding its Forward Plus runtime into protoprey's gl_compatibility game unresolved.
- WFE already provides project reads, delegation with questions and corrections, shared input/output manifests, and scoped local context blocks. Its contracts are in `pj-agdev/agdevworld/workflow_editor/docs/`.

These are design inputs, not assumptions that the same revisions or service state will hold when execution starts. The playprey delivery document also contains conflicting statements about dotfile handling; reconcile its claims against current code and WFE behavior during implementation.

## Ownership and storage

| Information or artifact | Owner and location |
| --- | --- |
| General methods, applicability, failed approaches and lessons | Producer's maintained wedo/knowledge; use its existing layout |
| Offered capabilities, request inputs, delivery format, tool usage and limitations | Producer's version-controlled documentation and tools |
| Scene requirements, integration decisions, importer/adapter, game-specific editor and authored tuning | protoprey Git |
| Adopted delivery identity, producer revision, hashes, configuration and provenance | protoprey Git |
| Large source material, checkpoints and published deliveries | Shared storage, with references and manifests in tracked records |
| Service endpoints, credentials, host tool locations and share mount paths | Environment-owned configuration, scoped context and ignored files |
| Staged inputs, generated caches and working copies | Each run's ignored local storage |

Storage choices follow the artifact's purpose; this plan adds no universal file-size prohibition. Inspect actual repositories rather than assuming they already obey this layout.

`shared://` identifies a logical location, not globally portable storage. Keep content identity separate from retrieval location. An adopted delivery must be recoverable from an explicit local package or a remapped storage location without requiring the original environment. Published bytes are not edited in place; a revision is another publication. Retention of adopted material must survive run cleanup.

## Step 1 — Establish the current cooperation map

1. Read published project definitions, guides, workflows, delivery tools, relevant experiments and current run records. Record revisions and distinguish published state from local workspaces.
2. Inspect the actual storage and runtime requirements: tracked assets, ignored inputs, shared references, renderer assumptions, environment configuration and execution hosts.
3. Consult nctl/Nautobot for the relevant services and shares, then verify only the live capabilities needed for the trial. Keep machine-specific findings in an ignored episode note and publish a portable summary.
4. Select one existing landscape input/effect and one existing predator profile suited to a small protoprey scenario. Prefer available material so the trial measures cooperation rather than unrelated content production.
5. Identify which requirements are already met and which need code, documentation or environment changes. Update this plan with concrete findings where necessary.

Outcome: a bounded integration case, an ownership map and an evidence-backed list of gaps.

## Step 2 — Make each producer usable by a consumer

Provide one discoverable entry per producer covering:

- What it can provide and when it is useful.
- Inputs and constraints the requester should supply.
- Existing workflows and how to obtain or commission a result.
- Output format, revision, configuration, runtime requirements and verification.
- Known limitations and links to supporting knowledge and trials.

For playprey, build on the existing workflow and delivery documentation and tools. For gimmickimg, package one useful existing expression into a consumer-usable result and add the missing delivery/verification support. Do not generalize every experiment before the first trial.

Separate two contracts:

1. **Delivery:** content, format version, provenance, producer revision (including modified working bytes when applicable), hashes, runtime requirements and evidence. Reuse WFE manifests for transport identity and integrity; add domain metadata only where needed.
2. **Embedding:** scene entry and exit, inputs, camera ownership, state/events, resource paths and renderer requirements. These describe how a consumer uses the delivered result.

Keep reusable producer behavior in the producer. Put protoprey-specific conversions and lifecycle integration in protoprey. A new dedicated workflow is justified by recurring work or trial evidence; the existing `freework` entrance is sufficient for a first commission.

Outcome: each selected result can be requested, delivered and understood without private knowledge of its producer's workspace.

## Step 3 — Give protoprey an integration path

1. Add a maintained cooperation entry, such as `docs/cooperation.md`, linked from the project guide. Explain candidate producers, how to inspect their public contracts, and the scene-specific information a request needs.
2. Use an existing or narrowly added scene-making workflow. Its run may reuse a delivery, add a delegate node, implement locally or choose another approach. Do not force every scene through both producers.
3. Implement acquisition, integrity checks and import for the selected deliveries. Record adopted revisions and hashes; support explicit package input or retrieval-location remapping for use outside this WFE environment.
4. Implement protoprey-owned integration code and any editing support needed for the selected scenario. Preserve authored game tuning separately from regenerated producer output. Make replacement and deliberate re-import understandable operations.
5. Resolve the playprey/protoprey renderer mismatch through a concrete technical trial. Record the chosen approach and its implications for the existing game. A standalone producer demo or a separate window is not evidence of in-game integration.
6. Handle corrected deliveries through existing delegation revision and receipt assessment. Verify their usefulness inside protoprey, not only the producer's completion claim.

Outcome: the selected landscape and predator expression participate in a playable protoprey scenario, with reproducible provenance and explicit ownership.

## Step 4 — Connect environment facts to actual run workspaces

- Read other projects through `wfe projects`; request work through delegation. Do not assume a sibling checkout exists on an execution host.
- Resolve shared references with the executing host's WFE share configuration and stage inputs into that run's own workspace.
- Deliver environment information through the existing scoped context/configuration mechanisms. A file in a developer checkout's `.local/` does not automatically exist in a fresh run or on another worker.
- Give generation-service endpoints to the producer that invokes them. A consumer that delegates generation needs the producer's contract and delivery, not duplicated service configuration.
- Identify the owner and update mechanism for each environment fact, with observation time and provenance where relevant. Use cagent-managed resource information where it already fits; avoid adding a second source of truth.
- Extend WFE or the executor only for a demonstrated missing capability. If changes are necessary, implement focused checks, update affected callers and deploy the changes before the trial.

Outcome: a fresh run can locate its instructions, tools and inputs on its actual execution host without manual file placement by Omni Agent.

## Step 5 — Run the cooperation trial

Initiate the selected scene-making task through the normal in-system workflow entrance. Let protoprey's agent discover the producer contracts and conduct the work using the available tools. Do not hand it an undisclosed Omni-only route.

Record:

- What the agent discovered and which published revisions informed its choice.
- Delegation requests, questions, answers, outputs and result assessment.
- The exact deliveries adopted, their manifests and protoprey's integration changes.
- Any missing information, inaccessible resource, incorrect assumption or manual intervention.

If Omni Agent supplies a missing fact or repairs the path, record that intervention. Improve the owned contract/tool/context and repeat the affected portion from a fresh run context. Do not present an assisted outcome as proof of independent cooperation.

Verify the following:

| Check | Required evidence |
| --- | --- |
| Discovery and commissioning | An in-system protoprey run identifies and uses the producer's public entrance |
| Delivery integrity | Consumer verifies the selected bytes against recorded manifests |
| Actual integration | Both selected expressions are exercised inside protoprey, with visible behavior and working controls |
| Fresh-copy reproduction | A clean consumer copy obtains the declared artifacts and reproduces the integrated result |
| Producer independence | The prepared consumer runs without access to producer checkouts or generation services; do not stop shared production services just to prove this |
| Environment portability | A relocated copy uses an explicit package or remapped artifact source, without the original absolute paths |
| Revision and replacement | A corrected or alternative delivery can be adopted explicitly, preserving authored tuning and the identity of the previous result |
| Existing behavior | Relevant existing game checks pass; visual and interaction checks cover what headless tests cannot |

Use a second execution host where available and useful for verifying path resolution. A same-host relocation proves only the tested relocation behavior, not untested OS or renderer compatibility. Use focused tests for changed contracts and import behavior; do not add tests that merely mirror documentation or reversible text edits.

Outcome: recorded end-to-end evidence, with remaining limitations stated precisely.

## Step 6 — Deploy and extract the standard

1. Commit and push the implemented work in its owning repositories. Integrate completed run work through the existing mechanisms and confirm the published revisions contain it.
2. Deploy changed runtime/system components and prepare the integrated protoprey result for immediate use. Complete asset retrieval and setup on the review host. Any review web service must be reachable from the LAN.
3. Verify the deployed or prepared result, including a browser or interactive game check as appropriate. Provide the Developer with one entry point and a short route to the two integrated expressions. Keep local addresses and launch details in ignored environment notes and the user-facing handoff.
4. Extract the smallest supported cooperation standard into the existing shared development documentation and agent guidance: capability discovery, ownership, delivery identity, consumer integration, environment resolution and evidence.
5. Keep project-specific lessons with their owners. Add instructions to shared guides only where the trial demonstrates their value; do not turn one successful sequence into a mandatory procedure.
6. Write `report.md` here with implemented changes, revisions, trial/run references, checks, interventions, limitations and recovery information. Update relevant development/environment documentation so later runs use the delivered system.

Outcome: usable deployed work and a standard grounded in a demonstrated case, rather than a new coordination framework.

## Completion criteria

- Protoprey can discover, obtain and integrate the selected results from both projects through the normal system mechanisms.
- The integrated game works without producer checkouts or live generation services after its artifacts are acquired.
- Delivery identity, authored tuning, integration code, shared payloads and environment settings have explicit owners and appropriate storage.
- Fresh-copy reproduction, relocation and a delivery revision/replacement are demonstrated with stated limits.
- No unexplained Omni-only setup remains in the successful trial path.
- Necessary implementation is integrated and deployed; the Developer can inspect the result with few actions, without performing setup the agent could perform.
- The reusable standard and final report reflect actual evidence, including unresolved limitations. Approval requests are not completion prerequisites.

## Scope boundaries

Do not create a central cooperation project, a generic plugin registry, a replacement transfer system, or a mandatory producer chain for p1. Do not migrate every project or normalize every historical artifact. Additional projects, such as forge or archsage, may participate when the chosen case needs their existing capabilities; expansion is driven by the case, not by an obligation to include them.
