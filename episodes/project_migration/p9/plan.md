# project_migration p9 — Front onboards lowerpoly

## Objective

Fulfill [braindump.md](braindump.md): bring the existing Gitea repository
`trustedagent/pj-lowerpoly` under managed project execution, with Omni giving
the request and Front performing the onboarding. Preserve the existing
repository history and its local `study/`, `wedo/`, `lab/` and `devdocs/`
directories. Prove that Front can obtain a useful result through the
onboarded project and return inspectable evidence.

Start with `freework`. Derive a reusable workflow from the actual work and
its evidence once the inputs, execution requirements and output checks are
understood.

This is an implementation plan. The preliminary observations below are
read-only findings, not evidence that onboarding or a Front trial succeeded.

## Working policy

Follow `devpolicy/styles.md`, `devpolicy/terms.md`, `devdocs/README_DEV.md`,
applicable repository instructions, ignored environment notes and
`localrule.md`.

- Generated files are English; communication with the Developer is Japanese.
- Agents carry out implementation, operational preparation, commit/push,
  deployment and verification without an additional human approval gate.
  Omni may answer in-system permission requests on the Developer's behalf.
  Record the actual actor; do not represent an agent's judgment as the
  Developer's own words.
- Workflow approvals, decisions and confirmations remain records rather
  than execution permissions. Ask the Developer only for genuinely missing
  intent that cannot be resolved from the request and existing evidence.
- Backward compatibility is not required. Keep the implementation small and
  use existing project, area, run and delegation contracts.
- Prepare a deployed result and direct LAN-accessible review links so the
  Developer can inspect it with few actions. Do not restrict new review
  services to loopback.
- Consult Nautobot or nctl for service placement and state before operational
  work. Distinguish the age of stored observations from fresh runtime checks.
- Keep host addresses, machine-specific paths, credentials and raw
  operational evidence in ignored local files. Tracked files use portable
  repository and run references.
- Preserve unrelated working changes. If Omni performs work belonging to
  Front or a specialist, record: "did X for agent Y — handoff candidate".
  Such assistance diagnoses a gap; it does not prove autonomous completion.

## Preliminary findings

Read-only inspection on 2026-10-20 established:

| Subject | Finding |
| --- | --- |
| Existing repository | `trustedagent/pj-lowerpoly`, populated `main`, not yet in the managed registry |
| Root layout | `devdocs/`, `lab/`, `study/`, `wedo/` and `.gitignore`; no `project.yaml` |
| Knowledge | Existing research notes and experiment results, including a mesh simplification/LOD pipeline; some claims and human evaluation remain explicitly unverified |
| Local areas | Supported by `ag.project.v2`; top-level `study` and `wedo` are categorized correctly |
| Registration | `wfe project register` requires a published `ag.project.v2` definition before it can register the repository |
| Front authority | Its guide authorizes project creation/configuration and workflow/run operations; Git and shell are available |
| Current WFE authentication | The configured Gitea principal is `developer`, which has access to the target |
| Agent authentication | An existing `trustedagent` credential authenticated successfully and reported pull/push/admin access to the target; no branch protection was listed |
| Services | nctl reported Nautobot reachable and authenticated; WFE reported both configured execution workers available |

Recheck mutable facts at execution time. API permissions and Omni's reads do
not establish that Front's attempt can use the credential or successfully
publish a change.

Relevant implementation references:

- `pj-agdev/agdevworld/workflow_editor/docs/contract.md`: project definitions,
  local areas and repository bindings.
- `pj-agdev/agdevworld/workflow_editor/server/agdev.ts`: existing-repository
  registration and workspace acquisition.
- `pj-agdev/agdevworld/workflow_editor/server/gitea.ts`: credential loading
  and authenticated Git environment.
- `pj-agdev/agdevworld/workflow_editor/server/basics.ts`: basic `freework`
  placement.
- `pj-agdev/agdevworld/workflow_editor/docs/delegation.md`: specialist work
  and continuation.
- `devdocs/episodes/project_migration/p8/` and `p8/ex1/`: managed Front,
  operator capabilities and releasing slots while waiting.

## Design decisions

### Preserve local knowledge areas

Declare the existing directories in the root project definition:

```yaml
schema: ag.project.v2
id: lowerpoly
name: Lowerpoly
devdocs: directory
areas:
  study: Research findings and references
  wedo: Lessons learned through experiments
  lab: Experiments and tools
```

Front adds intent, goals and a concise project guide after reading the
existing material. Preserve the distinction between recorded findings,
working assumptions and pending human evaluation.

Local areas remain part of the root Git history and are published and
integrated with it. No shared knowledge repository or submodule conversion
is needed. Declare workflow bindings explicitly and inspect the generated
freework bindings before starting a run: run adjustments cannot add new
repository bindings later.

Keep knowledge maintenance with lowerpoly initially. If later work calls
for archsage supplementation, use the existing maintenance contract and
target-project workflow for root-local knowledge; this onboarding does not
require that additional arrangement.

### Front owns onboarding; specialists own product work

Omni supplies the request, ensures common capabilities are usable, observes
the execution and independently checks the result. Front reads the source,
prepares definitions, publishes them, registers the project, obtains a
managed workspace, establishes its workflow and coordinates verification.
The lowerpoly run performs the actual research or pipeline work.

Do not have Omni prepare the target project's definition or register it
before the Front trial. Fix missing common capabilities where a real gap
appears, then let Front continue using them. Put tool usage in help and
durable coordination lessons in Front's knowledge.

### Use trustedagent deliberately

Use `trustedagent` for Front's onboarding Git writes. Reuse the existing
credential and authenticated Git mechanism; give Front a discoverable
operation, not a token pasted into a prompt or guide.

Repository ownership, authentication principal, Git commit authorship and
WFE run actor are different facts. Record them separately where relevant.

The existing WFE service authenticates as `developer`. Do not globally
replace its credential as a side effect of onboarding one project. In the
capability audit, explicitly decide and document whether subsequent
coordinator publication/integration stays on that service principal or uses
a target-scoped `trustedagent` selection. The baseline scope permits the
existing service principal; do not claim every write is by `trustedagent`
when only Front's preparation uses it. Extend scoped credential selection
only if required by the actual execution or adopted identity contract.

### Learn the reusable workflow from freework

Use Front's existing conversational workflow for onboarding. After
registration, establish lowerpoly's basic `freework` and delegate a bounded,
useful request to it. The initial candidate is to explain the existing
pipeline, reproduce one representative case and deliver its outputs,
execution recipe and limitations.

Choose the case after inspecting available assets, dependencies and
existing records. Large inputs and outputs use the existing shared-area
facilities when applicable. A folder named in an old report is not evidence
that an execution worker can access it.

After the trial, extract the reusable work into an ordinary workflow with
the necessary bindings, execution settings, inputs and output checks.
Do not encode every incidental shell step or failed experiment as a
mandatory workflow node.

## Step 1 — verify the capability boundary

1. Re-read applicable instructions, inspect working changes and deployed
   revisions, and check service state through Nautobot/nctl and the relevant
   WFE read operations.
2. Confirm the repository's current branch, history, layout, registration
   state and existing project/workflow definitions, if any appeared since
   planning. Read enough existing experiment material to select a useful,
   bounded trial without expanding this into new algorithm research.
3. Trace Front's actual execution identity and access to the operating area,
   Git, Gitea configuration and registration/workspace operations.
4. Check authenticated clone/read, definition preparation, commit/push,
   registration, acquisition, basic-workflow publication and integration.
   Distinguish supported operations from missing documentation and genuinely
   missing tool capabilities.
5. Confirm that Front can use `trustedagent` without exposing its secret in
   published files, command output or run records. Record the intended
   authentication principal for each write path.
6. If necessary, provide a small common authenticated Git or onboarding
   operation through existing modules and CLI help. Avoid a lowerpoly-only
   script or a new general credential-management subsystem. Keep source
   inspection and content choices with Front.

Deliverable: `report1.md` with the capability matrix, identity decisions,
actual gaps and selected trial. Keep machine details in ignored evidence.

## Step 2 — complete and deploy only the missing capabilities

1. Implement demonstrated gaps from Step 1 using existing shared operations.
   Expose useful errors and retry behavior so Front can continue after a
   partial clone, an existing registration or a failed push.
2. Preserve existing history and concurrent edits. Preparation must not
   recreate a populated repository, overwrite an existing definition
   silently or force-push over unrelated work.
3. Make any new credential selection explicit for the operation or target;
   retain unrelated projects' existing behavior.
4. Add focused tests only for changed behavior: relevant examples include
   existing-repository handling, identity selection, publication failures
   and retry outcomes. Reuse existing local-area coverage.
5. Run required checks for touched components and deploy necessary changes.
   Verify the installed tool's help and behavior from the same execution
   context Front uses.

Deliverable: `report2.md` with changes, checks and deployed revisions. If no
code change is needed, record the existing capability evidence and proceed.

## Step 3 — ask Front to onboard the repository

1. Post one request through the managed Front conversation entrance,
   attributed to Omni acting for the Developer. Include the target
   repository, preservation of local areas, authority to complete the work,
   the initial useful trial and the required result links.
2. Front reads the repository, creates the project definition and guide in
   its preparation checkout, and commits/pushes them using `trustedagent`.
   It records the base and resulting commits.
3. Front registers the published project with
   `wfe project register trustedagent/pj-lowerpoly`, then obtains a managed
   workspace with `wfe project obtain lowerpoly`.
4. Front places the basic workflow, verifies its bindings and execution
   settings, validates the definitions and publishes the changes. Saving
   a definition file alone does not establish that Gitea contains it.
5. Front checks that project discovery and the editor expose lowerpoly and
   its local areas, and records what remains before the useful trial.
6. Omni observes through conversation and run records, answers ordinary
   operational questions under standing authority and fixes infrastructure
   gaps if necessary. Record all interventions and let Front resume the
   target work.

Deliverable: `report3.md` with Front conversation/run references, published
commits, project/workflow identities and intervention history.

## Step 4 — obtain a useful result through lowerpoly/freework

1. Front verifies the chosen case's inputs, dependencies and suitable
   execution node. Infrastructure changes, if needed, go through the
   managed cagent workflows and their normal state checks.
2. Front delegates the bounded request to lowerpoly/freework. Its own
   attempt releases the conversation slot while waiting, using the existing
   delegation lifecycle.
3. The lowerpoly run reads existing study and wedo, performs the case and
   records the method, results and limitations. Add substantive new
   findings to the appropriate local area, with evidence; do not invent
   knowledge edits merely to exercise a path.
4. Front handles questions settled by the request and examines the child's
   actual outputs before calling the work complete. Distinguish technical
   measurements and agent assessment from pending human visual judgment.
5. Complete result decisions and integration through ordinary WFE
   operations. Verify that intended local-area changes, run records and
   project changes reached their published destinations.
6. Front returns usable links to the run, results and any review surface.
   Prepare LAN access and keep the result available for inspection.

Deliverable: `report4.md` with the execution node, exact inputs and outputs,
run/commit references, verification results and review entry points.

## Step 5 — extract and verify the reusable workflow

1. Front and the lowerpoly project use the successful freework history to
   identify recurring work, required inputs, dependencies, meaningful
   research/implementation/check steps and delivered artifacts.
2. Create a focused ordinary workflow for that recurring request. Document
   how its steps derive from the trial. Retain freework for exploratory
   requests.
3. Validate and publish the workflow, then exercise it on a bounded case
   that tests reuse. Reuse the original fixture when that is sufficient;
   do not require a large second research campaign.
4. Confirm that a later request through Front discovers and uses the
   workflow, and that its result is inspectable through the same delivery
   path.
5. Store coordination lessons in Front's knowledge and project-specific
   operating knowledge in lowerpoly, with the trial references.

Deliverable: `report5.md` with workflow provenance, reuse evidence and any
remaining constraints.

## Step 6 — verify and close

1. Independently check the matrix below against deployed behavior and
   published state. Fix demonstrated gaps and repeat affected checks.
2. Update shared help/contracts only where behavior changed; update
   `devdocs/README_DEV.md` and ignored environment notes where needed.
3. Review intended diffs, run checks appropriate to changed components,
   commit/push and update relevant parent submodule pins as applicable.
4. Write `report.md` with delivered behavior, evidence, direct review
   entry points, actual authentication identities, Omni interventions and
   explicit remaining limitations.

| Completion criterion | Required evidence |
| --- | --- |
| Existing project preserved | Original history retained; no duplicate replacement repository |
| Front performed onboarding | Conversation/run records and published changes identify Front's actions |
| Local knowledge supported | Editor shows study/wedo as local areas; the run reads them and substantive updates integrate with the root |
| Authentication works | Front's preparation push succeeds as trustedagent; later publication/integration principals are stated truthfully |
| Managed operation works | Registry, workspace, definitions and published workflows agree |
| Useful freework completed | Reproducible bounded case, actual outputs, checks and limitations |
| Workflow extracted | Published workflow derived from trial evidence and successfully reused |
| Delivery is usable | Front's reply links to available run evidence and LAN-accessible results |
| Authority is exercised | Routine operational decisions handled by agents without new human approval gates |
| Claims remain accurate | Pending human evaluation and Omni assistance are identified separately from autonomous technical success |
