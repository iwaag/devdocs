# Workflow editor p8/ex1 — Simple and Detail views

## Objective

Implement [braindump.md](braindump.md): provide a one-click switch between a genuinely minimal Simple view and a Detail view with the existing inspection and adjustment capabilities.

Simple should communicate the purpose, current situation, and person's next action through prominent titles, recognizable icons, and readable graphical objects. Reducing small text is a requirement, not merely a side effect of adding collapsible sections.

Use [Take 3](../../design/simpler/image_suggestion/take3/suggestion.md) for the direction of visual hierarchy. Adapt the layout to actual operations and graph structure. The p8 screenshots are the implementation baseline, including the information still visible when every detail section is closed.

## Scope and implementation boundary

Implement in `pj-agdev/agdevworld/workflow_editor`, using its current TypeScript/DOM architecture. Cover Dashboard, Project Editor, Workflow Editor, and Run view: commit IDs and remotes remain prominent in the Project Editor, so limiting this iteration to the latter two screens would leave the reported problem unresolved. Apply the same switch to the User nodes library with a minimal presentation pass.

Keep the existing data, API, execution semantics, approvals, and persistence operations. This work requires no generated summaries, new node types, node-only execution, or new agent-control primitives. Detail uses the existing capabilities; avoid maintaining two independent editors or duplicate forms.

Inspect the current checkout, installed release, and p8 checks before implementation. Use isolated services and fixtures for mutations and browser verification. Production data is only for read-only comparison. Implementation and deployment status are reported separately.

## 1. Define the display contract before changing the layout

Create a shared display preference, for example in `src/displayMode.ts`, with `simple | detail`. The initial default is Simple. Remember the person's choice locally across navigation and reload; malformed or unavailable local storage falls back safely to Simple. This is a viewer preference, not project or workflow data.

Place the same labeled Simple/Detail control in each screen's top bar. Expose its selected state accessibly. It must take one click to switch and must not navigate, submit, save, or change execution state.

Map all existing information and operations to one of three places: Simple's primary content, context shown after selection, or Detail. Use the following contract:

| Screen | Simple primary content | Detail content |
| --- | --- | --- |
| Dashboard | Project names, short purposes, meaningful status icons, attention requests, and project-opening actions; one entry to create/add a project | Registrations, repository catalog and management, workspace provenance, service and registry details |
| Project | Large project name, readable purpose, workflow choice and existing request form, recognizable recent runs and their status | Editable project settings and goals, repository bindings/management, Git status, commits/remotes, workspace inventory, workflow creation and technical metadata |
| Workflow | Name, short purpose, readable graph, Add node, Save/unsaved state, concise approval/validation status | Full intent and separate approval controls, documentation policy, bindings, detailed validation, appearance/provenance and precise editing controls |
| Run | Outcome or workflow title, current human task, relevant question/review/report actions, concise progress and availability, graphical workflow | Exact references and revisions, timestamps/backend, resolved conversations, all reports/files, execution/publication/coordination records, history, decisions and run adjustments |
| User nodes | Name, large recognizable icon, short purpose, selection and add-template entry | Full expectations, revision/provenance, template editing fields |

Detailed operations remain reachable from explicit entries, such as “Project settings,” “Intent & approvals,” or a selected object's “More details.” Activating one switches to Detail and reveals the relevant section. These contextual entries complement the global toggle; do not reproduce the complete list of closed technical sections in Simple.

Simple excludes raw commit IDs, remotes, internal paths, backend/model identifiers, exact timestamps, routine technical explanations, and lengthy per-step outcome lists. Avoid repeated status/count displays in the title, focus card, graph header, and sidebar. Keep a concise navigation breadcrumb, but use human-readable names rather than technical paths.

**Completion criterion:** Every existing capability has a destination, and the initial Simple screen has a clearly smaller visible information set even before anyone collapses a section.

## 2. Establish the visual direction on real data

First implement a Simple Project view and a Simple completed Run using representative existing records. Capture them before extending the design to every state. These two screens expose both the Git-information problem and p8's remaining dense outcome/detail presentation.

Use a clear type hierarchy: prominent page title, readable node/object names, and secondary text used sparingly. Increase the size and prominence of meaningful existing icons. Allocate recovered space to those objects instead of leaving the previous tiny content inside larger empty panels.

For a completed Run, show the actual completion and decision states, an available result title, and the final-report/request actions. Keep stopped presentation availability explicit. Place the full recorded-outcome list in Detail; do not replace it with invented or model-generated summary prose. For Project, show the purpose as text in Simple and edit it through settings in Detail.

Compare these captures with p8 and Take 3 at the same viewport. Correct the hierarchy before propagating shared styling. Functional correctness alone is insufficient evidence that the visual goal has been achieved.

## 3. Apply Simple to graphs and contextual editing

**Main files:** `src/canvas.ts`, `src/workflowView.ts`, `src/inspector.ts`, `src/styles.css`.

Simple cards emphasize a recognizable icon, readable name, and concise current state where applicable. Hide full descriptions, repository chips, type/provenance text, and routine report details until selection or Detail. Preserve type accessibility and meaningful distinctions using icons, accessible labels, and selection details. Keep a recognizable, accessible report-step entry without adding a paragraph or numerous badges to each card.

Selecting a node in Simple reveals a compact explanation and the relevant existing action: edit its name/description in the definition view, or inspect its recorded outcome/report in the Run view. “More details” opens the full inspector in Detail. Edges and generated steps retain selection and an explicit path to their inspection controls. Do not turn generated reports into authored nodes.

Simple/Detail is independent of Compact/Mini. Keep Compact/Mini as an optional graph display setting in Detail; Simple uses its own minimal card presentation, retaining the previous Detail choice. Never overload the existing Compact/Mini persistence key with the new preference.

Preserve the actual DAG and stored positions. Improve readable initial scale and framing as needed, without requiring a two-row chain or automatically rewriting layout. Preserve node selection and pan position across display switches; accommodate changed card/container sizes and reveal the selection when necessary. Avoid automatic Fit on every switch or background update. Report icons, edges, dragging, connecting, and hit targets must remain aligned.

## 4. Preserve essential state distinctions in Simple

**Main files:** `src/runView.ts`, `src/runSide.ts`, `src/runCommon.ts`, affected Project/Dashboard displays.

Keep the state-dependent human task introduced in p8. Display only the information required for that task, with an explicit route to supporting detail:

- Open question: the question and its one canonical answer form; relevant plan or review link.
- Answer awaiting take-up: receipt and who holds the next move, without offering it as unanswered.
- Working: current work/report step and a concise executor observation; existing Stop remains reachable without searching through technical records.
- Held or failed: short reason and the supported recovery action, including Resume's instruction where applicable.
- Accepted with cleanup pending: acceptance and unfinished cleanup remain separate.
- Completed without acceptance: distinguish completion from the decision and expose the supported decision action.
- Cancelled or historical: explicit status; historical views have no live observations or mutation controls.

Show disconnection, stale/missing observations, errors, and unavailable artifacts with recognizable icons and short text. Missing information must not look like healthy progress. Preserve the distinction between recorded node progress and actual executor activity without repeating the long explanatory note everywhere; full explanation belongs in Detail.

Several questions or failures may need attention at once. Provide a concise count/list and access to each; do not hide all but one because Simple has a primary task. Routine Git metadata stays in Detail, while a resource problem that prevents the current task can appear as a short actionable status.

## 5. Switch presentation without losing user state

Reuse mounted forms and existing keyed sections. The switch changes visibility and presentation, not the owning draft or underlying view lifecycle. Preserve workflow drafts, actor/approver fields, answers, resume instructions, decision evidence, adjusted YAML/reason, selection, disclosure states, and unsaved-navigation guards.

Keep a single instance of each interactive control. Do not clone forms for both modes or rebuild them during SSE/host updates. Explicit mode switching can change focus and visibility, but it must retain text and provide predictable keyboard focus. If switching would hide or move a composing control, defer that operation until composition ends. Background state changes retain p7/ex2's protection of invalidated active drafts.

Remember Detail's expanded sections and scroll positions while Simple is shown. Restore them on return, adjusting only if the content or scroll range changed. Adapt reading anchors to the new containers rather than relying on p8's old geometry. File contents retain their internal offsets when returning to the same file.

Keep the refresh coordinator, scoped host updates, stale-response guards, and server preconditions. Preview/Apply validity cannot be restored from display preference: adjustment previews remain bound to their exact input, sequence, and revision. Dispose mode listeners with the view and verify navigation does not accumulate handlers.

## 6. Validate both the visual reduction and operation continuity

Add a focused browser check, for example `checks/displayMode.ts`, using private state fixtures. Extend existing checks where behavior changes rather than duplicating their full coverage.

Verify:

- Fresh storage starts in Simple; the one-click toggle and locally remembered preference work across views and reload.
- Commits, remotes, internal paths, backend identifiers, and full resolved-history/technical section lists are absent from Simple's visible content but available in Detail.
- Contextual detail links switch mode and reveal their target. All supported controls remain reachable.
- Switching preserves unsaved project/workflow edits, answer/resume/adjustment drafts, graph selection, file position, and Detail disclosure state.
- SSE, heartbeat, conflict, question resolution, disconnection, and navigation during mode changes do not duplicate actions or lose drafts.
- A selected report remains inspectable, custom appearance preserves state recognition, and arbitrary branch/join graphs remain accurate.

Run `npm run check` and affected browser checks, including `simplerUi.ts`, `runInteraction.ts`, `runs.ts`, `docsteps.ts`, `appearance.ts`, `review.ts`, `nodes.ts`, `conversation.ts`, `ui.ts`, and the Project/Dashboard checks whose interactions changed. Inspect prerequisites and report skips explicitly. Update old selectors and expectations directly.

Capture initial Simple and corresponding Detail views in `ex1/image_after/` at 1440 × 1000 and check 1280 × 800. Include Dashboard, Project, definition with/without selection, User nodes, working Run, open question, live review, accepted-but-cleaning-up, completed, held/failed, disconnected, and historical views. Use long Japanese/English names and realistic records, not only short happy-path fixtures.

Compare Simple screenshots with p8. The visual acceptance criteria are:

1. The purpose, situation, and next action can be identified without reading small technical text.
2. Primary titles/icons/nodes are visibly more prominent; fewer small text blocks and routine controls compete with them.
3. Detail is one click away, and switching retains the person's work and reading context.

Check keyboard navigation, accessible toggle state, and native macOS Japanese IME during switching and background updates. Keep synthetic composition and native verification results distinct. Record any unverified manual checks honestly.

## Deliverables

- Shared Simple/Detail preference and redesigned screens in the agdevworld package.
- Updated documentation and focused regression checks.
- `ex1/image_after/` with real screenshots and capture conditions.
- `ex1/report.md` describing the visible information contract, before/after assessment, source revision, validation, remaining limitations, and local/deployed status.

Completion requires both meaningful visual reduction and preserved access to existing operations. Passing tests or providing a toggle alone does not meet the objective. For a subsequent deployment, use the existing agdevworld release process and its execution guards, then verify the installed revision separately.
