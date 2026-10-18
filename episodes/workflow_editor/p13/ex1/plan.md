# Workflow editor p13 ex1 — context inside the editors

## Objective

Improve the GUI delivered by [p13](../report.md). The Developer found the
separate Context editor disconnected from project, workflow and run editing,
and context unavailable in important Simple-mode paths.

Make context an ordinary part of each editor. People can read and change the
instructions for the object they are working on without leaving that object or
switching to Detail mode. Keep the scope matrix as a secondary overview.

This plan covers implementation, verification and deployment. Creating this
plan does not start those activities.

## Working conventions

- Follow `localrule.md`: generated files are English; Developer communication is
  Japanese. Agent-operable work, commits, pushes and deployment proceed without
  additional human approval checkpoints.
- Deliver a deployed result accessible from other PCs on the home LAN, with
  direct inspection links and few actions required to try it.
- Backward compatibility is not required. Reuse p13's resolver, override store,
  conflict handling, snapshots and delivery semantics; remove redundant UI paths
  rather than maintain competing editors for the same content.
- Preserve unrelated work, existing preferences and explicitly held runs.
  Machine addresses, paths and raw runtime evidence belong in ignored notes.
- Before runtime inspection or deployment, obtain current service state through
  pj-clusterintent's Nautobot or `nctl`; use `pj-clusterintent/nctl/README.md`.

## Interaction contract

### Placement and scope

| Surface | Primary content and actions |
| --- | --- |
| Home | Visible summary and inline editing of instructions shared by all projects |
| Project editor | Project purpose, goals, guide references and local instructions; inherited system guidance available in place |
| Workflow editor | Workflow purpose, guide references and local instructions; inherited system/project guidance available in place |
| Run editor | Original request, subsequent instructions and context used for execution; differences expected on the next attempt |
| Context overview | Scope matrix for cross-scope inspection and navigation to the relevant editor |

Use a consistent Context section near the object's purpose or request. It must
have a visible entry and useful summary in Simple mode. Opening or editing it
must preserve the current display mode and editor location. A collapsible section
can limit page height; do not require every inherited document to stay expanded.

Use the object's current selection to determine scope. Do not ask users to select
the project or workflow again after entering its editor. Editing system guidance
from Home must work without choosing an arbitrary project.

### Simple and Detail

Simple mode includes instruction text, editing, save state, inheritance and
meaningful application status. Detail adds source paths, revisions, hashes,
delivery evidence and attempt history. Display mode changes the amount of
technical information, not access to core context operations.

Prefer labels that describe the user's choice:

- **Shared with this project** for repository-owned purpose and guidance.
- **Only in this environment** for local project/workflow instructions.
- **Inherited instructions** for wider-scope content, with its scope named.
- **Used for this attempt** and **Changes for the next attempt** on runs.

Refine exact copy during browser verification. Keep repository/local ownership
visible because it affects where instructions travel. Put storage paths and
implementation terminology in Detail.

### Editing and saving

- Integrate existing purpose/goals/guide controls into the context section or
  reuse their existing draft model. One value has one editable control and one
  draft; avoid a second intent editor with independent state.
- Edit guide declarations in place. Show referenced content and a clear source
  link where supported. This episode need not build a general file editor for
  arbitrary guide files or external repositories.
- Keep default edits in the project/workflow's existing save transaction. Give
  local instructions a clearly labelled save action backed by the p13 override
  API. Do not imply that one save atomically commits both storage kinds.
- Preserve unsaved text across display-mode changes, panel expansion, ordinary
  live refreshes and unrelated editor actions. Navigation protection accounts
  for both definition and local-instruction drafts.
- Retain concurrent-edit detection and useful recovery. Save failures leave the
  draft intact; partial success across separate saves is visible.
- A preview incorporating unsaved definition edits must say it is a draft.
  Otherwise label the preview as saved context and indicate pending edits.

Inherited sources are readable in place. An explicit **Edit project instructions**
or **Edit instructions for all projects** action may open an embedded editor or
drawer, with the affected scope visible. Saving there must clearly describe the
wider scope; it must not silently create a narrower override or require an extra
approval dialog. Users can return to their original editor without losing drafts.

### Run behavior

Keep the original request immutable and use existing additional-instruction
records for new run-specific guidance. Bring these controls into the primary
run experience without duplicating their records or creating a run override store.

The context summary follows the current/latest attempt, with older attempts
available through Detail. Distinguish a prepared snapshot, recorded delivery and
unavailable history truthfully, using concise language in Simple mode.

Show when inherited settings have changed since that attempt. Editing project or
workflow instructions from a run states that the change applies to future
attempts of all affected runs. Saving starts no run and does not alter a live
attempt. Offer the existing additional-instruction action for steering this run;
do not automatically send an instruction when a preference is saved.

The overview matrix remains useful for cross-scope comparison, but routine
editing should be complete within Home/project/workflow/run views. Reuse the
same UI components if editing remains available from the overview.

## Starting points

Within `pj-agdev/agdevworld/workflow_editor/`:

| Area | Files |
| --- | --- |
| Existing context UI | `src/contextView.ts` |
| Main editors | `src/homeView.ts`, `src/projectView.ts`, `src/workflowView.ts` |
| Run surfaces | `src/runView.ts`, `src/runSide.ts`, `src/runCommon.ts` |
| Display and navigation | `src/displayMode.ts`, `src/main.ts`, `src/styles.css` |
| Shared API and resolver | `src/api.ts`, `server/context.ts`, `shared/context.ts` |
| Verification | `checks/context.ts`, existing display-mode/editor interaction checks |

Consult current source and repository development instructions before choosing
exact component boundaries and release commands.

## Step 1 — establish reusable embedded context components

1. Trace current purpose/guide editors, override drafts, route disposal, live
   refresh and display toggles. Identify where existing controls will move or be
   reused so the new section does not duplicate editable values.
2. Extract the override editor, inherited-source display, summary and attempt
   comparison from the standalone Context page into reusable components.
   Define lifecycle, dirty-state, refresh and save-result interfaces.
3. Support a system-only view independently of project selection. Add only the
   minimal API adaptation needed; reuse the same store and resolution rules.
4. Implement consistent summary and disclosure styling for desktop and narrow
   layouts. Keep technical metadata behind Detail or a secondary disclosure.

Deliverable: reusable components with explicit ownership of drafts and saves.

## Step 2 — integrate Home, project and workflow editing

1. Add the system instruction summary/editor to Home in both modes.
2. Integrate project purpose, goals, guide references and local instructions into
   the project editor. Show inherited system guidance in the same section.
3. Integrate workflow purpose, guides and local instructions into the workflow
   editor. Keep graph/node editing readily accessible and show inherited project
   and system guidance without navigating away.
4. Connect existing default save paths and local override saves. Implement the
   combined dirty-state/navigation behavior and avoid losing textarea selection
   or focus on background refresh.
5. Turn standalone Context entrances into an optional overview action. Update
   matrix links to reach the relevant editor section directly. Remove redundant
   primary controls and duplicate editors.

Deliverable: normal context changes are possible in Simple mode where the user
already edits the project or workflow.

## Step 3 — integrate run guidance and context inspection

1. Place the request, additional-instruction action and context summary in the
   primary run surface, accessible in both modes.
2. Reuse the p13 snapshot/comparison API for the current attempt and next-attempt
   changes. Keep full rendered text, source metadata and historical selection
   available without making them prerequisites to understanding the summary.
3. Provide in-place access to inherited settings with explicit scope labels and
   preserve the run view's node selection, scroll position and unsaved input.
4. Handle runs before their first attempt, pre-p13 runs, unavailable snapshots,
   failed delivery and completed/held runs without inventing evidence or actions.
   Show unavailable instruction actions according to existing run semantics.

Deliverable: a user can answer “What was this run told?” and add guidance without
switching display mode or visiting a separate settings screen.

## Step 4 — verify complete user journeys

Use browser interaction checks that exercise the user's task, not only the
presence of controls. Cover Chromium and WebKit, desktop and a narrow viewport.

1. Starting in Simple mode, edit a project-local instruction, save it and read it
   as inherited guidance in a workflow without leaving the relevant editors.
2. Edit workflow intent and a local instruction together. Verify independent save
   results, mode changes, live refresh, navigation and a concurrent-edit conflict
   preserve drafts and identify what is saved.
3. Edit system guidance from Home without selecting a project. Confirm the wider
   scope is visible when accessed from a child editor.
4. Inspect a run's request and actual attempt context in Simple mode, change a
   workflow preference and see the next-attempt difference. Verify the current
   snapshot stays unchanged and saving does not start work.
5. Exercise the existing additional-instruction route from the integrated run
   surface on a bounded trial. Verify the instruction is recorded once and is
   distinguishable from preferences for future attempts.
6. Check keyboard access, labels, focus, readable long text, narrow layouts and
   preservation of workflow graph/run selection during context interactions.
7. Run affected regression and required release checks. Backend execution behavior
   is already established by p13; add focused tests only where this change alters
   API behavior or exposes a new integration risk.

Deliverable: evidence that the ordinary editing journeys work end to end in
Simple mode, with Detail retaining diagnostic depth.

## Step 5 — deploy and provide immediate inspection

1. Read current Nautobot/nctl state and installed release information. Deploy
   through agdevworld's normal WFE release procedure, preserving local context
   data and unrelated work. Verify installed assets and CLI/service compatibility.
2. Inspect the actual published LAN route. Repeat representative project,
   workflow and run journeys there using bounded trial data; restore temporary
   preferences and leave existing user content intact.
3. Commit and push the implementation, documentation and required submodule pins.
   Update context documentation to reflect the integrated entrances and modes.
4. Write `report.md` with revisions, verified journeys, screenshots/evidence
   locations and remaining limitations. Keep machine-specific details in ignored
   notes and supply direct links to Home and representative project/workflow/run
   views in the handoff.

## Completion criteria

- Home, project, workflow and run context is discoverable and usable in Simple
  mode; opening or editing it does not force Detail mode.
- Ordinary editing stays within the current object's editor. Inherited settings
  name their scope and offer a practical path to edit at that scope.
- Shared defaults, local instructions and run input retain clear ownership,
  correct save semantics and one authoritative draft per editable value.
- Unsaved text survives mode switches and refreshes; navigation and conflicting
  saves do not silently discard it.
- Runs show actual versus next-attempt context clearly. Preferences and immediate
  run instructions remain distinguishable, with no implicit launch or resume.
- The matrix serves as an overview, and Detail supplies provenance/history.
- The deployed LAN GUI is ready for the Developer to inspect with direct links;
  the implementing agent performs setup and verification it can do itself.
