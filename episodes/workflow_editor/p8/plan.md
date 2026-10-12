# Workflow editor p8 — a calmer UI grounded in the existing editor

## Objective

Implement [braindump.md](braindump.md): redesign the UI using [Take 3](../design/simpler/image_suggestion/take3/suggestion.md) as a broad visual direction. Treat the images as references for hierarchy, readability, and approachable presentation. Adapt them whenever the actual data, operations, or interaction requirements call for a different design.

The person should quickly understand what is being created, whether their attention is needed, and where to inspect or adjust the work. Detailed information and precise controls must remain easy to reach.

The primary implementation scope is the Workflow Editor and Run view shown in Take 3. Dashboard, Project Editor, and node-library changes are limited to shared styling and navigation consistency where needed. A broader redesign of those screens can follow after the two primary views have been evaluated.

## Starting point and boundaries

The implementation belongs to `pj-agdev/agdevworld/workflow_editor`, an internal agdevworld package. Use its existing TypeScript/DOM approach and installed-release workflow. The former experiment under `pj-agdev/experiments/workflow_editor` is not the implementation target.

At planning time, the agdevworld checkout was clean at `a571ae2`. The operating service's `/api/version` reported installed release `a571ae2deb5142a612b37859e9b956d7f6a96b01`, matching that source revision. Recheck both before implementation; the screenshots in `design/simpler/image_before/` are the visual baseline, not proof that the running service will remain at that revision.

This iteration changes presentation and interaction organization. It does not require new execution primitives, a new UI framework, generated summaries, or storage-schema changes. Keep supported operations and their server-side preconditions. Internal DOM structure, selectors, and tests can change directly; a duplicate legacy renderer or compatibility layer is unnecessary.

The implementation and browser checks use isolated projects, registries, and services. Production workspaces are used only for read-only visual comparison. Deployment is a separate release operation; a successful local build does not change the installed UI.

## 1. Establish a feature map and representative states

**Main files:** `src/workflowView.ts`, `src/inspector.ts`, `src/runView.ts`, `src/runSide.ts`, `src/canvas.ts`, `src/styles.css`.

Before moving controls, map every current section and operation to its destination in the new layout. Include less visible capabilities: edge editing, generated-step inspection, validation links, approval actions, user templates, run instructions, definition revisions and adjustment previews, related runs, presentation start/stop, publication retry, decisions, and historical file reads.

Create representative fixtures for:

- Definition editing with no selection, a node, an edge, and a generated documentation step selected.
- Default and manual documentation policies; custom node appearance; a branch and join; a larger graph.
- A run working normally, waiting for a plan answer, waiting for result review, answered but not yet taken up, held/stopped, failed/blocked, cancelled, and completed.
- Result accepted while cleanup is still pending, and completed without an accepted decision.
- A live presentation, a stopped one, and failed or unavailable host observations.
- Adjusted definitions, historical views, missing files, malformed records, and connection loss.

Use the existing models, compiler, reducer, and fixture helpers to construct these states. Keep fixture titles and outcomes explicit; do not encode the RTS example into the UI.

**Completion criterion:** Every existing capability has a planned entry point, and the design can be reviewed beyond the single completed-run example in Take 3.

## 2. Make the workflow definition readable and easy to adjust

**Main files:** `src/workflowView.ts`, `src/inspector.ts`, `src/styles.css`.

Replace the tall header with a compact breadcrumb, editable workflow name, save state, and Save action. Show a short purpose preview with a clear “Intent & approvals” entry. Keep intent and definition approval states visible, including stale states; the expanded area contains the authoritative intent, declared approver, and separate approval actions.

Move documentation policy, repository bindings, and detailed validation into an explicitly opened “Workflow settings” area. Keep a visible validation summary and actionable errors. Selecting a validation issue must reveal the relevant node, edge, or settings rather than lead to a hidden control.

Consolidate node creation into an “Add node” palette containing both base types and existing user templates. Preserve template appearance and provenance, catalog-error messages, and access to the User nodes library. Keep Compact/Mini, Auto-arrange, and zoom/Fit controls easy to find.

Show the inspector when the person selects a node, edge, or generated step, or explicitly opens settings. Provide a clear close/reopen path. For nodes, prioritize name and description; keep type, appearance, repository access, connections, and deletion accessible. Generated documentation steps remain inspectable and are not independently editable or movable.

Closing a panel must retain its draft. External changes, save conflicts, invalid YAML, and unsaved-navigation guards remain visible and functional. Opening settings or changing selection must not silently save or discard anything.

**Completion criterion:** The default screen foregrounds the readable workflow, while every existing edit and approval operation remains reachable and correctly scoped to the reusable definition.

## 3. Rebalance the canvas without prescribing the concept image's graph

**Main files:** `src/canvas.ts`, `src/styles.css`; `shared/layout.ts` only where layout metrics or arrangement require it.

Improve card typography, dimensions, descriptions, repository chips, and report-icon placement. Use restrained existing appearance colors; preserve type recognition and state text/glyphs independently of color. Keep the final report visually distinct as a documentation endpoint, not a new authored node type.

Tune initial framing for readable names and useful context. The concept's two-row chain is an example, not a required new layout algorithm. Preserve stored positions and arbitrary DAGs. Auto-arrange stays explicit and changes layout only; showing a run must never persist a new arrangement into its definition.

Keep explicit Fit available for seeing the whole graph. When a large graph would require unreadably small initial cards, use a readable initial view with pan/zoom and a clear way to inspect the rest. Document the chosen framing rule and test it on both small and large graphs.

Opening or closing an inspector changes the available canvas size. Recompute geometry without repeatedly resetting the person's zoom or pan. Check hit targets, connector endpoints, report attachment, node reveal, and generated-step edges after card-size and container-size changes. Dispose any newly introduced observers or listeners with the view.

**Completion criterion:** Names and states are readable in normal use; branches, joins, reporting dependencies, and manual arrangements remain accurate and usable.

## 4. Organize the Run view around the person's current task

**Main files:** `src/runView.ts`, `src/runSide.ts`, `src/runCommon.ts`; small view-specific modules if extraction clarifies ownership.

Give the Run view a compact header that distinguishes recorded execution progress, human decision, executor activity, and connection health. Keep fixed/adjusted definition identity and an “Open current definition” link explicit. Move timestamps, backend information, attempt IDs, and detailed provenance into a dependable details entry.

Use the actual run state to prioritize the main area:

| Situation | Main content and action |
| --- | --- |
| Open plan or ordinary question | Question, relevant evidence/report, and its existing answer form |
| Open result-confirmation question | Existing review link with availability, question, and answer form |
| Answer recorded, awaiting take-up | Answer receipt and who holds the next move; do not present it as an unanswered question |
| Work in progress | Active work/documentation step and observed executor activity, with access to existing Stop |
| Held, failed, or blocked | Reason and the supported recovery controls; Resume requires its existing instruction |
| Accepted, cleanup pending | Accepted decision plus unfinished cleanup; do not label the run completed |
| Completed | Available outcome records, final-report/request links, and the actual decision state |
| Cancelled | Cancellation reason and any remaining presentation/cleanup concerns |
| Historical revision | Recorded information and files, without live host facts or mutation controls |

A question answer is not automatically a result decision or an executor resume. Keep those operations and their existing preconditions distinct. Do not introduce “Pause,” “Run this node only,” or automatic presentation restart.

Use deterministic existing data for headings and summaries: an appropriate presentation title when available, otherwise the workflow name and run reference; recorded outcomes must identify their originating step. Do not select cleanup text as the substantive delivered outcome simply because it is the latest event. If there is no suitable summary, show status and links to the original request and reports. No new model call or fabricated prose is needed.

Show file actions only when their availability is known. A stopped presentation remains explicitly stopped; retained files and reports are separate from a live URL. Keep the existing review screen and its result-specific explanation as the one-click review destination.

Maintain the graph as an inspectable overview using the definition revision in force. Work and documentation counts stay separate. Make selected-node records read-only; the existing “Adjust future work” operation remains a separate, clearly scoped facility with preview and Apply.

**Completion criterion:** The person can identify their next action or read the outcome without scanning technical metadata, and the screen remains truthful across the full state matrix.

## 5. Add progressive disclosure while preserving interaction state

**Main files:** `src/runSide.ts`, `src/runRefresh.ts`, `src/patch.ts`, and affected view containers.

Group resolved questions, presentations, definition revisions, catalog snapshot, related runs, reports/files, execution/publication, coordination, and history behind labeled expandable sections. Provide the selected-node inspector independently of those records. Keep errors and required human actions visible even when their detailed section is closed.

Reuse the p7/ex2 persistent controls and keyed sections. Do not rebuild or reparent active forms on background updates. Each question retains one canonical answer form; explicit navigation from a selected node can reveal that form. Multiple open questions remain reachable with a count or list rather than being discarded by a single “primary task.”

Retain panel open/closed state, input drafts, focus, selection, IME composition, nested file scrolling, and history anchors. Adapt the existing anchoring code to any new section nesting or scroll containers; its current selectors assume direct child sections of the right pane.

When a state transition resolves or invalidates an active form, update status and submission availability immediately, but do not hide a composing control or discard unsent text. Keep that draft accessible until the person explicitly leaves it. Background updates must not automatically reopen panels or move the reader; explicit navigation may reveal content.

Retain the refresh coordinator, host-only updates, stale-response guards, and `expectSeq`/`expectRevision` checks. Adjustment previews remain tied to the exact YAML, reason, sequence, and revision inspected. Historical reads never mix in current host observations.

**Completion criterion:** A calmer layout does not regress the input and reading guarantees established in p7/ex2.

## 6. Validate behavior and compare actual screenshots

Add a focused browser check, for example `checks/simplerUi.ts`, using a private service and fixture. Assert meaningful behavior: visible next actions, correct state distinctions, accessible hidden controls, preserved drafts through disclosure and updates, and accurate graph connections. Avoid tests that merely mirror CSS class names or lock the concept image's pixels.

Run the package's `npm run check`, then relevant existing browser checks: `checks/ui.ts`, `checks/runs.ts`, `checks/docsteps.ts`, `checks/appearance.ts`, `checks/nodes.ts`, `checks/review.ts`, `checks/conversation.ts`, and `checks/runInteraction.ts`. Review each check's prerequisites before invoking it; update affected selectors directly. Expand validation if changes reach additional behavior, and document unavailable prerequisites or skipped checks.

Capture real redesigned screens under `p8/image_after/`, with a screenshot index and capture conditions. Compare at the baseline 1440 × 1000 viewport and also check a smaller desktop viewport such as 1280 × 800. Include no-selection and selected definition views, a branch/join, a large graph, an open plan question, a live review, accepted-but-cleaning-up, stopped/failed, completed, historical, and expanded details.

Manually check keyboard navigation, focus when panels open/close, long English/Japanese text, and native macOS Japanese IME during background updates. Separate synthetic composition results from native IME verification.

Review screenshots against three questions:

1. Is the current situation and the person's role clear on opening the screen?
2. Can the person reach the available outcome or relevant evidence without searching through files or old questions?
3. Can they reach precise details and controls without losing their place or draft?

Document justified deviations from Take 3, including unavailable data and layouts that work better for actual graph structure. Refine the implementation based on this comparison rather than pursuing exact image reproduction.

## 7. Deliverables and release readiness

- UI implementation in agdevworld's `workflow_editor/`, with affected documentation and checks updated.
- Real screenshots and an index under `p8/image_after/`.
- `p8/report.md` describing final behavior, source revision, check results, manual verification, deviations from the concepts, and remaining limitations.

Completion requires preserved supported operations, passing relevant automated checks, and actual visual evidence for the redesigned states. Record native IME or other manual verification still outstanding rather than claiming it from automation.

For a subsequent installed release, use agdevworld's `scripts/wfe-release.sh` from the reviewed source commit. Reinspect installed version and execution activity, retain the script's open-attempt guard and rollback behavior, and verify service and CLI revision after activation. Report local implementation and deployment status separately.
