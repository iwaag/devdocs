# Workflow editor p13 — scoped context and visibility

## Objective

Implement [braindump.md](braindump.md): make the context used when authoring
workflows and executing runs explicit, editable and inspectable across system,
project, workflow and run scopes. Separate repository-owned defaults from
local overrides, and show both the current configuration and what an actual
attempt received.

The Developer accepted the proposal discussed alongside this braindump. This
plan makes it concrete; writing the plan does not start implementation or runs.

## Working conventions

- Follow `localrule.md`: generated files are English and communication with the
  Developer is Japanese. Agent-operable implementation, verification, deployment,
  commits and pushes need no additional human approval checkpoint.
- Deliver a deployed result on the home LAN, with direct inspection links and
  few actions needed to try it. Do not finish with source changes alone.
- Backward compatibility is not required. Update current consumers and remove
  obsolete duplicate paths instead of maintaining two context systems.
- Keep host paths, addresses and raw local evidence in ignored files. Tracked
  documentation uses portable paths and identities.
- Inspect current service state through pj-clusterintent's Nautobot or `nctl`
  before runtime work, using `pj-clusterintent/nctl/README.md`.
- Preserve unrelated changes and explicitly held runs. Use fresh, bounded trials
  rather than resuming historical work for verification.
- Context informs agents; it introduces no acknowledgment, approval or execution
  gate. Keep tool usage in help and shared facts in one authoritative place.

## Context contract

### Scope and storage

| Scope | Repository-owned default | Local override / run input |
| --- | --- | --- |
| System | Common execution and authoring contracts maintained by agdev | Developer-wide preferences and local environment guidance |
| Project | `project.yaml` intent/goals and explicitly declared project guides | Instructions for this project across workflows |
| Workflow | Workflow intent and existing `execution.guides` | Instructions for runs and authoring of this workflow |
| Run | No independent pre-authored default; resolve inherited sources | Original request, subsequent answers and instructions, using existing records |

Run input remains its own category. A braindump is the original request, not a
private preference store: existing run documents can be committed. Do not change
that contract or introduce a redundant run-override editor in this phase.

Default/override describes ownership and storage, not an unconditional priority
order. Present scope and provenance explicitly. Applicable explicit requests and
specific instructions refine general guidance; a broad “English unless otherwise
requested” preference must not erase a project's Japanese-language deliverable.
Do not parse prose into key/value overrides or claim to automatically resolve all
semantic conflicts. Clearing an override removes that contribution; it does not
mask inherited defaults.

System execution and authoring contracts have different consumers. Show their
applicability instead of passing the complete executor contract to an IDE agent.
Other sources use the same resolver in both contexts.

### Local storage and source identity

Use the WFE area's ignored local storage as the authority for overrides. A
portable logical layout is:

```text
<area>/.local/contexts/
  system.md
  projects/<project-id>/project.md
  projects/<project-id>/workflows/<workflow-id>.md
```

Keep revision history and update metadata beside these sources or in an existing
local store, behind one storage API. GUI and CLI edits use that same API. Each
revision records its author, time and exact content, including removal. Use stable
project/workflow identities and detect concurrent edits rather than overwriting
an unseen revision. Validate identifiers and resolve files within the context
store. The initial system scope is one WFE area, not an inferred OS account or a
new multi-user preference service.

Zulip is not the authority in this phase. A future Zulip editing entrance can
call the same API without adding a second source of truth.

Do not duplicate operational configuration in prose. Use existing share settings
and `shared://` references for shared storage; override prose explains purpose and
usage. Nautobot/nctl remains the source for cluster state.

### Resolution, delivery and history

- Produce one structured context bundle with scope, category, applicability,
  source identity, revision/content hash, content or reference, and delivery mode.
- Inject short instructions and override bodies directly. Mark reference-only
  material explicitly; “reference supplied” does not mean “agent read it.”
- Resolve a bundle once per attempt claim, persist its exact local snapshot, and
  pass those same bytes to the executor. A concurrent edit cannot make the preview,
  recorded snapshot and launch payload disagree about the attempt's input.
- Defaults for a run come from its own project tree and active frozen workflow
  definition. Cross-project guides retain the current committed-read mechanism,
  with the resolved commit recorded. Overrides come from the owning WFE area's
  current local settings. Authoring uses the selected editable project/workflow.
- Save overrides for the next attempt; do not interrupt or mutate an active
  attempt. Existing run instructions remain the route for steering current work.
  Saving preferences alone does not enqueue or resume a run.
- Include the effective context fingerprint in session configuration comparison.
  A changed bundle starts a new session on the next attempt using existing session
  replacement behavior. An unchanged bundle can resume; reconstruction after
  compaction includes the same applicable context.
- Keep exact override bodies and snapshots outside Git-managed run trees and
  publication/integration inputs. Portable run records can name an opaque snapshot
  id and fingerprint without embedding local paths or override text. If the local
  snapshot is absent on another host, report it as unavailable, not reconstructed
  from today's settings. This does not promise that an agent can never quote an
  instruction in an artifact it authors.
- Distinguish resolved/prepared context from context handed to a launched attempt.
  Record launch failure honestly; delivery is not proof of comprehension or use.

## Starting points

Paths below are relative to the projects directory unless stated otherwise.

| Area | Starting point |
| --- | --- |
| Models and definitions | `pj-agdev/agdevworld/workflow_editor/shared/model.ts`, `server/yamlDoc.ts`, `shared/validate.ts` |
| Default project guides | `workflow_editor/server/basics.ts` within agdevworld |
| Execution and session choice | `workflow_editor/server/exec.ts`, `shared/run.ts`, `docs/sessions.md` |
| Prompt construction | `pj-agdev/agautolab/src/agautolab/wfexec.py`, `agent/guides/wfrun/guide.md` |
| Authoring entry | `workflow_editor/templates/AGENTS.md`, `templates/START.md`, `cli/setup.ts` |
| UI and CLI | `workflow_editor/src/{homeView,projectView,workflowView,runView}.ts`, `cli/wfe.ts`, `server/api.ts` |
| Local storage examples | `workflow_editor/server/catalog.ts`, `server/registry.ts`, `docs/storage.md` |
| Release | `pj-agdev/agdevworld/scripts/wfe-release.sh` and agdevworld's `README_DEV.md` |

## Step 1 — establish sources and implement the resolver

1. Trace current bootstrap, continuation, reconstruction and authoring paths.
   Inventory direct text, referenced files, generated facts and harness-loaded
   repository instructions. State which WFE controls; do not label the WFE bundle
   as the complete model prompt when harness or IDE configuration adds context.
2. Add an explicit project guide declaration to `project.yaml`, using the existing
   guide-reference conventions. Retain workflow `execution.guides` for workflow
   additions. Resolve project intent/goals and workflow intent as distinct sources.
3. Update basic-workflow generation and current managed definitions as needed so
   project guides inherit once rather than being copied into every workflow.
   Preserve provenance when the same resolved guide is referenced more than once;
   deliver identical content once. Do not promote a workflow-specific guide to
   project scope merely because several workflows happen to reference it.
4. Implement the local override store, revisions, concurrent-edit handling and
   shared resolver. Missing optional overrides mean no contribution; unreadable
   configured sources produce explicit errors rather than silently disappearing.
5. Add CLI operations to inspect/edit/clear scoped overrides and resolve context
   for authoring or a run. Provide human-readable and structured output. Document
   source selection, applicability and delivery modes in command help.

Deliverable: one context model and resolver, usable through CLI and service APIs,
with a concise `workflow_editor/docs/context.md` contract.

## Step 2 — connect execution and record actual delivery

1. Resolve and persist context during claim, extending attempt settings and the
   executor payload. Keep local snapshot storage out of checkout/checkpoint paths.
2. Compose the bootstrap from the common execution contract and resolved sources
   without appending old project-guide injection a second time. Preserve run input,
   state, pending answers/instructions and existing generated resource information.
3. Extend session comparison and reconstruction. Include additions, removals and
   changed content; metadata-only edits with identical effective content need not
   discard a conversation. Record a useful replacement reason.
4. Expose an attempt's saved bundle and the prospective next-attempt bundle through
   read APIs/CLI. Show their differences without modifying either source.
5. Keep delegated runs scoped to their target project/workflow. Their request
   carries explicit delegated work; parent project overrides do not implicitly
   leak into the child's context.

Deliverable: traceable context across fresh attempts, resumed sessions,
reconstruction, delegation and service restarts.

## Step 3 — add the matrix and attempt inspection UI

1. Add a Context entry accessible from system/home, project and workflow views.
   Show the scope/storage matrix with inherited entries visible. Open defaults at
   their existing definition or guide source; edit overrides in place.
2. Show source, update metadata and revision history. Preview the full resolved
   bundle using the same resolver as execution. Clearly distinguish body delivery,
   references, absence and read failures.
3. In the run view, show “Used by this attempt” and “Next attempt with current
   settings.” Highlight added, changed and removed sources and label settings not
   yet applied. Older runs without snapshots remain inspectable with an honest
   unavailable state.
4. On save, explain that changes affect the next attempt. Provide access to the
   existing additional-instruction action for current-run steering without
   automatically sending instructions or starting work.

Deliverable: users can edit a preference and see exactly where it applies, then
inspect what a concrete attempt was given without reading raw records.

## Step 4 — connect workflow authoring

1. Add concise tool discovery to the authoring template: the context command shows
   the selected project's/workflow's guidance and where it comes from. Keep usage
   details in help and avoid copying override bodies into tracked `AGENTS.md`.
2. Update the installed authoring-area entry documents as well as generation
   templates. Reuse the same resolver for existing workflows and new workflows
   that inherit only system/project context until they have an identity.
3. Provide a convenient UI action to copy the authoring context or its retrieval
   command. External IDE sessions are not launched by the WFE executor: label this
   as available/copied context, not verified automatic injection.
4. Exercise a real authoring session using the command and verify that project
   preferences affect the edited workflow without becoming copied workflow text.

Deliverable: workflow authors and run executors consult the same scoped sources,
with the limits of external-session delivery visible.

## Step 5 — verify, deploy and demonstrate

1. Run focused tests for scope isolation, source selection, guide deduplication,
   override removal, concurrent edits and snapshot consistency during an update.
   Test context changes across fresh/resumed/reconstructed attempts and delegation.
2. Verify that checkpoint, publication and integration do not include local
   override files or snapshot bodies. Exercise missing snapshots and source errors.
   Validate changed managed project/workflow definitions and run required checks
   from the affected repositories' development instructions.
3. Use browser checks for matrix navigation, editing, unsaved changes, revision
   conflicts and the used-versus-next diff. Verify ordinary desktop and narrow
   layouts through the normal WFE route.
4. Inspect current Nautobot/nctl state and installed releases. Deploy the WFE
   service, CLI, executor changes and authoring entry updates through their normal
   release paths. Back up affected local stores and verify installed versions.
   Record local operational details in ignored notes.
5. Run bounded demonstrations on the deployed service: a system English-default
   preference, a project-specific instruction, a workflow-specific instruction,
   and an explicit run request that refines the general language preference.
   Verify actual generated outputs, not merely the presence of prompt text.
6. Change one override between attempts and remove it again. Verify the current
   attempt remains unchanged, the next uses the new bundle, session replacement
   is explained and earlier snapshots remain readable. Test persistence across a
   service restart and LAN access from the ordinary published editor URL.
7. Clean up trial settings without erasing trial evidence or user preferences.
   Commit/push implementation, documentation and required submodule pins. Write
   `report.md` with tested revisions, actual behavior and remaining limits. Supply
   direct deployed links to the matrix and representative attempt comparisons.

## Completion evidence

- System, project and workflow defaults/overrides are visible in one consistent
  model; run input retains its existing authorship and recording semantics.
- GUI, CLI, authoring and execution resolve the same applicable source content.
- A saved override reaches the next attempt, including after session reuse or
  reconstruction, with a truthful record of delivery and source versions.
- Earlier attempt context can be inspected locally after settings change; local
  context storage is not copied into Git-managed publication artifacts.
- Workflow editing has a practical retrieval path and no false automatic-delivery
  claim. Agents retain autonomy and no new permission process is introduced.
- The deployed LAN UI and real trial results are immediately inspectable with
  a small number of actions; completion does not depend on the Developer running
  setup commands that the implementing agent could perform itself.
