# project_migration p7 — add one PC execution node

## Objective and scope

Implement [braindump.md](braindump.md) with the scope selected in discussion: add the GPU development PC as a working WFE execution node, prove real playprey development there, and leave a reproducible example of adding a PC. The existing execution host continues serving other work concurrently. The management UI and conversational entrances remain unified.

Only this first additional PC is in p7. Adding the always-on VM host or any other PC is deferred until this case is demonstrated successful. No VM rollout, long-running CPU experiment migration, automatic placement optimization, live run relocation, automatic failover, or control-service high availability is part of this phase. The central WFE, Git and shared-storage services remain dependencies.

The immediate predecessor is [retire/old_autolab](../retire/old_autolab/report.md). Its removal of the legacy listener, gateway, missions and deployment role is the starting point. Extend the retained WFE executor; do not restore those paths.

Follow `devpolicy/styles.md`, `devpolicy/terms.md`, `devdocs/README_DEV.md`, applicable project instructions, ignored environment notes and `localrule.md`. Generated files are English. Backward compatibility is not required: migrate retained data explicitly and remove superseded active behavior rather than maintaining two execution systems. Services and review surfaces are accessible from the home LAN. Agents perform setup, deployment, verification and ordinary operational decisions; approvals and result decisions are records, not new permission gates. Omni has the Developer's standing authority to answer in-system operational permission requests. Human feedback concerns the delivered experience after agent-performable preparation is complete.

This plan does not claim implementation or deployment. Physical node names, addresses, filesystem paths, credentials and raw operational evidence belong in ignored local notes. Here, **coordinator** means the existing WFE management service, **existing worker** its current execution host, and **PC worker** the additional PC selected by the Developer. These names describe roles, not new service products.

## Starting facts to recheck

- WFE currently owns a host-local execution database, one execution slot, local PID reconciliation and run copies whose `run.json` is the live record. Claims return local paths. Cloning a workspace on another host does not distribute this state.
- Run-owned branches, nested-submodule handling, shared input/output manifests, context snapshots and harness session identities already exist and should remain the foundation.
- The retained `agautolab.wfexec` launches agents; intake and notification currently run alongside it. Additional workers need execution capability, not additional conversational entrances.
- The planning-time Nautobot query shows one retained executor placement. The PC already serves image-generation workloads. Registered convergence is not proof of current GPU capacity, a usable interactive desktop or a working Godot development session.
- The retirement report names 13 pre-existing notifier/conversation test failures. Establish their current disposition before changing those paths, so new failures are not dismissed as inherited.

## Architecture and ownership

| Owner | Responsibility |
| --- | --- |
| WFE coordinator | Project identity, request receipts, run allocation, authoritative run state, assignment, attempt ownership, context, questions/answers, integration records and unified reads |
| Worker on each host | Local workspace/cache, run working copies, agent and child-process lifecycle, local files, checkpoints, publication and durable reporting of execution outcomes |
| Gitea and existing shared areas | Versioned source/run branches and portable large-input/output delivery respectively |
| Nautobot, nctl, deployment backend and managed cagent | Placement, installation, configuration and observation of workers and their infrastructure dependencies |
| Managed playprey | Actual development task, portable project requirements, usable result and reusable project knowledge |

Use one coordinator with workers claiming assigned work through its API. The existing worker uses the same contract as the PC worker. Local IPC or local calls may be implementation details, but state transitions have one implementation and one authority.

The coordinator becomes authoritative for run mutations. Worker-side run records are identified snapshots for tools, checkpoints and publication; they are never an independently writable second authority. The existing `wfe` agent interface routes state operations to the coordinator and file operations to the owning worker as appropriate. Preserve discoverable CLI usage and the current agent contract; agents should not need to invent HTTP calls or synchronize records manually.

Keep source, imports, build products and intermediate assets on each worker's local disk. Git and `shared://` carry published material. Do not share the execution database or a writable working tree over NFS.

Project/workflow defaults and request-time overrides select an execution node separately from the model profile. Define and document precedence. A run keeps its assigned worker across attempts; a request-time choice is recorded. Start with one slot per worker, allowing the two workers to run concurrently. Missing capability or an unavailable worker leaves work waiting with an actionable reason, rather than dropping it or silently sending it elsewhere.

## Step 1 — inspect the PC and specify the execution contract

1. Read repository instructions and record source/deployed revisions and unrelated working changes. Consult Nautobot/nctl for the actual PC, existing services, placements and observation ages; collect fresh observations where needed. Keep infrastructure changes out of this inspection step.
2. Inspect the PC's actual OS and execution environment, supervisor, agent harnesses, Git/submodule access, available disk and memory, graphics/display access, Godot tooling and existing image-generation load. Choose the real supported execution environment from evidence; do not assume a particular OS, container or desktop arrangement from CUDA readiness.
3. Trace WFE request creation, run allocation, claims, run operations, context/detail reads, project access, delegation, publication, integration, presentations, notification and observation. Identify which operations currently rely on direct access to the owning working tree.
4. Specify the minimum coordinator/worker protocol: registration and identity, supported profiles/capabilities, assignment, preparation, claim/start/finish, heartbeat, stop, file/log access and durable retry receipts. Include per-worker slot ownership and one active attempt per run.
5. Define the run-state cutover and the relationship between authoritative state, local snapshots and Git publication. Name where frozen definitions, adjusted definitions, authored guides, run documents and context snapshots are read; preserve their actual revisions rather than substituting the coordinator's unrelated checkout.
6. Select one bounded, useful playprey task using available input assets. Include a visible Godot result, a question/resume exchange and a code or configuration change that can be integrated. Select a small concurrent task for the existing worker.

Deliverable: `report1.md` with the gap/ownership matrix, protocol and migration decisions, PC requirements and trial scope. Resolve routine design choices during implementation without adding an approval checkpoint.

## Step 2 — separate authoritative state from local execution

1. Implement coordinator-owned run allocation, request deduplication and state transitions. Keep project/run identity global to the managed system; additional local checkouts do not create duplicate projects or competing run numbers.
2. Add execution-node defaults/overrides and per-worker queue/slot state. Match claims to assignment and offered capabilities. An incompatible worker cannot consume or drop another worker's job; waiting for one PC must not block the existing worker's queue.
3. Bind every worker report to its worker and attempt identity. Claims and starts are idempotent; duplicate or late reports cannot create an extra attempt, overwrite a later attempt or release another worker's slot.
4. Make central `wfe run` operations and worker file operations cooperate without competing writes to `run.json`. Define snapshot sequence/revision handling, including what gets committed and how the final integration result remains visible when publication precedes a later state event.
5. Preserve context resolution, immutable attempt snapshots, referenced detail reads and other-project reads on the PC. Resolve environment facts for the assigned host rather than presenting the existing worker's local paths as usable PC paths.
6. Maintain question/answer, hold/resume, stop/cancel, adjustments and delegation semantics through the central authority. Audit implicit single-slot assumptions, including instructions about when delegated work can execute. Do not expand this into a general scheduler.
7. Prepare an explicit migration of existing live/held/completed records, queues, receipts and integration journals. Back up the affected stores and copies; perform the cutover between attempts without resuming held work. Historical records remain readable. Remove superseded writable paths after migration.

Deliverable: `report2.md` and working state/assignment contracts, with focused tests for concurrent claims, unique allocation, report retries, snapshot authority, incompatible capabilities and retained run semantics.

## Step 3 — implement worker lifecycle and local materialization

1. Extend the retained executor to register, claim and report through the coordinator. Keep intake/notification as one centrally operated pair; PC execution does not provision a new Zulip entrance.
2. Materialize the selected source commit and initialized nested submodules into a run-owned local copy. Preserve run branch identity and integration targets. Make source commits required by the job reachable before dispatch; missing commits are a preparation problem, not a reason to use a different revision.
3. Separate portable project requirements from host overlays. Install harness/tool dependencies and configure local credentials and provider URLs through the existing deployment mechanisms. Do not copy the existing workspace's `.local/` wholesale onto the PC.
4. Stage shared inputs locally and publish outputs with the existing manifests. Record host-specific share availability. Keep Godot imports and other tools that write beside assets away from published input/output folders.
5. Launch and stop process groups on the owning worker. The coordinator never interprets a remote PID as a local process. Record actual worker, attempt, tool/profile and preparation failures in the normal run view.
6. Persist enough worker bookkeeping to reconnect after process/service restarts and retry outcome reports without relaunching completed work. Heartbeat loss means contact is lost, not that the process ended. Keep the run assigned, investigate actual process/outcome state on reconnect and do not execute a replacement on another host.
7. A requested stop is pending until the owning worker confirms its outcome. A worker unable to contact the coordinator must not start new work; define how a running attempt pauses or records pending reports when state-changing commands are unavailable. Retry receipts prevent duplicate questions, outputs or integration requests.
8. Preserve same-host harness-session continuation. If a session is unavailable, use the existing explicit replacement record and reconstruct from run records; do not imply that conversation files are portable merely because the source is in Git.

Deliverable: `report3.md` and usable workers on the actual two environments. Focused lifecycle tests cover duplicate claims/reports, disconnect/reconnect, worker restart and stop confirmation. Use fixtures for damaging timing cases rather than disrupting unrelated production work.

## Step 4 — complete unified review, publication and integration

1. Show selected worker, availability/last contact, queue reason, attempt state and pending stop in the existing UI. A live run remains distinguishable from its last published Git snapshot when its worker is offline.
2. Make logs, run documents, diffs, artifacts and presentations accessible from the normal central run view. Identify unavailable remote content honestly. Cache or publish bounded review material where useful; keep large payloads on the existing delivery path.
3. Bind confirmations to the exact candidate and output manifest. Keep decision recording independent of permission to publish or integrate, consistent with the current integration contract.
4. Integrate the actual worker-produced commits through one journaled operation, using deepest-first submodule handling and current target branches. Handle partial pushes and conflicts through existing recorded recovery, without a second integration owner on the worker. Parallel runs must not overwrite each other's target updates.
5. Provide a LAN-usable way to inspect the Godot result, including screenshots/capture and a prepared native launch or interactive session as appropriate. Verify graphical rendering and agent observation on the PC; a headless import or file download alone does not prove interactive development.
6. Verify that browser, CLI, Zulip transport and Observer read the same run identity and state. Correct the inherited notifier test expectations where the changed contract requires it, and document any genuinely unrelated remaining failures.

Deliverable: `report4.md` and an end-to-end review/integration path usable without browsing the PC's private working directories or manually copying files.

## Step 5 — deploy and register the PC worker

1. Add reusable deployment/configuration support for the actual PC environment in the existing backend. Include supervisor installation, pinned runtime/tool versions where needed, worker identity, local storage, central endpoint, Git access and shared-area setup. Use parameters, not project-specific machine paths embedded in code.
2. Exercise deployment tooling through managed cagent's current development/change paths as applicable. Omni can handle authorized operational questions. Keep effect records accurate; do not introduce plan-agreement or human-only installation gates from older episode instructions.
3. Deploy the coordinator and existing worker together with the state migration, then provision the PC worker. Back up affected state and record a coherent rollback procedure; old code cannot simply be started against a newly authoritative store format.
4. Register/observe the retained executor service and new placement in Nautobot/nctl using the new executor's actual health semantics. Do not revive the deleted legacy gateway or deployment role to satisfy a health probe.
5. Verify the PC execution identity can fetch all required repositories, stage shared inputs, invoke its selected harness, run Godot and publish a result. Verify supervisor restart/reconnection and central reachability from the LAN.
6. Recheck existing image-generation services and record actual resource competition during the trial. Add scheduling/resource restrictions only for demonstrated problems; leave generalized GPU allocation out of scope.

Deliverable: `report5.md` with deployed revisions, migration outcome, recorded placement/observation and basic execution readiness. Store physical configuration and raw evidence in ignored environment notes.

## Step 6 — prove the additional node with real work

Run the selected playprey task through the deployed system, with agents performing preparation and operational verification.

| Trial | Required evidence |
| --- | --- |
| Ordinary request | The normal WFE request path selects the PC and creates one run; no manual SSH command starts its agent |
| Local development | The PC prepares a run copy, consumes verified inputs, edits/builds locally and launches and observes the Godot result |
| Question and resume | The central view receives a real question; an answer resumes the same run on the same PC with the recorded context/session behavior |
| Parallel work | A separate existing-worker run makes progress while the PC is executing; neither consumes the other's slot |
| Delivery and integration | The central view exposes the exact outputs and code changes; source/submodule changes integrate and the result is ready for human inspection |
| Availability and recovery | A bounded trial-worker interruption/restart leaves honest state and returns to usable service without duplicate execution or lost run identity |
| Repeatability | A second ordinary request uses the deployed PC without manual repair, ad hoc file transfer or a new one-off installation |

Record source/deployed revisions, run IDs, input/output identities, timing, available resource observations and every Omni intervention. Fix demonstrated defects and repeat affected checks. Keep any personal task-specific edits in the managed project's normal run branches and integration history.

Prepare the actual review links and minimal launch action before asking for experiential feedback. If human feedback remains pending, distinguish that from technical completion. Do not count an executor heartbeat, a synthetic echo, a clone or a headless import as the successful PC-addition case.

Deliverable: `report6.md` containing the real trial, recovery evidence, repeat request and any remaining limitations.

## Step 7 — document the example and close p7

1. Update executor/WFE documentation, agent tool help, `devdocs/README_DEV.md` and ignored environment notes for central state, worker assignment, local preparation, publication, availability and recovery. Remove obsolete descriptions of one global slot or local-only live records.
2. Write a short reusable PC-addition recipe: inspect the actual environment, provision the worker and dependencies, register/observe it, exercise the ordinary request path and verify delivery. Distinguish reusable machinery from facts specific to this first PC. Do not deploy another node as part of writing the recipe.
3. Run repository-required checks for touched components and the focused tests justified above. Review diffs for unrelated edits, local infrastructure details and unsupported completion claims. Commit/push intended changes and update parent submodule pins where applicable.
4. Write `report.md` with delivered behavior, deployment and migration outcome, reproducible trial evidence, review entry points referenced through local notes, interventions and named limitations. Explicitly judge whether adding this PC is now a successful, repeatable case.
5. Leave adding the VM host or another PC as a future decision after that finding. Do not automatically extend p7 when the first PC succeeds.

## Completion criteria

- The added PC is a persistent, observed WFE worker using the common execution contract, with no restored old-autolab entrance or gateway.
- A real playprey task completes the request, local development, question/resume, graphical verification, delivery and integration round trip on the PC through the normal UI and tools.
- The existing worker executes independent work concurrently, while run IDs, authoritative records and attempt ownership remain unambiguous.
- Connection loss, stop and restart are reported and recovered honestly; neither automatic duplicate execution nor manual state-file repair is needed.
- A subsequent normal request works without Omni installing or copying missing pieces by hand.
- The result is deployed and easy for the Developer to inspect from the LAN. Required checks and actual trial evidence support the claims; unresolved limitations are explicit.
- The reusable PC-addition procedure and final report establish one successful case. Additional hosts remain deferred.
