# Plan: Preserve input and reading state in the run editor

## Objective and scope

Fix the run view so background updates do not interrupt Japanese IME composition, replace active inputs, reset nested scrolling, or move the reader away from the content being read. Keep execution status, run progress, and connection health live while the user interacts with the view.

This plan implements the findings in [investigation.md](./investigation.md). The confirmed implementation target is `pj-agdev/agdevworld/workflow_editor`, starting from agdevworld commit `6058c0f`. The deployed release has not yet been matched to that checkout.

Backward compatibility is not required. Internal interfaces, DOM structure, selectors, and tests may change directly; do not add compatibility shims, retain the old renderer, or introduce a migration period. Breaking changes are allowed when they simplify this fix. Existing run data need not be discarded: changing its storage schema is unnecessary for this problem.

Use the existing TypeScript and DOM approach. Do not introduce a UI framework or redesign unrelated editor screens. Deployment is a separate operation from implementing and validating this plan.

## Step 1 — Add isolated reproduction checks

**Files:** `checks/runInteraction.ts` (new), fixture helpers under `checks/` as needed.

Create a private project, registry, service, and browser fixture using the same server modules as `checks/runs.ts`. Avoid production workspaces and registries. Ensure cleanup also runs when browser startup fails.

Build a run containing an open question, a long readable file, enough history to scroll, and the states needed to expose resume and adjustment forms. Add assertions for:

- The same textarea remaining connected and focused across a real SSE run-progress update.
- CDP `Input.imeSetComposition` remaining usable across that update, followed by composition completion and continued typing.
- File content retaining nonzero vertical and horizontal scroll positions.
- An executor response whose only change is `executor.at` leaving the answer form untouched. Intercept the fixture response for deterministic heartbeat changes.
- The currently visible history entry keeping its viewport position when newer entries are inserted above it.

Record which assertions fail against the current implementation. Check DOM identity and composition behavior, not only the final input string. Keep native macOS IME validation separate from synthetic composition checks.

**Completion criterion:** The checks reproduce the confirmed input replacement and nested-scroll failures without modifying production data.

## Step 2 — Give each mounted run view one refresh lifecycle

**Files:** `src/runView.ts`; a run-specific refresh module if extraction improves clarity. Change `src/live.ts` only if its existing interface cannot support this design.

Replace the independent full-load and asynchronous interval paths with one per-view refresh coordinator:

1. Accept full-run and host-facts refresh requests; merge pending requests so a full refresh subsumes a host-only refresh.
2. Allow only one refresh batch to run at a time. Requests arriving during that batch become the next batch rather than being lost.
3. Fetch independent resources concurrently within a batch, but publish one coherent result with explicit per-resource success or error state.
4. On a full refresh, fetch presentation facts using the newly read run's presentation state. Preserve the existing last-usable-record behavior for malformed run data.
5. Mark the coordinator disposed on navigation and ignore every subsequent completion. Close SSE, cancel timers, and abort requests where supported; result guards remain mandatory even when abort is used.
6. Keep the current three-second host refresh cadence without overlapping requests or accumulating an unbounded queue. Reconnection and successful mutations request a full refresh.
7. Make mutation callers await their requested refresh's completion, rather than returning early merely because another load is running.

Use a separate file-request identity for the open file. A late response for file A must never replace file B, a newer request for A, or a disposed view.

**Completion criterion:** Delayed responses cannot roll state backward or update a disposed view; SSE and polling still converge to the latest server state.

## Step 3 — Replace whole-pane rendering with persistent sections

**Files:** `src/runView.ts`, `src/runSide.ts` (new), small run-specific section modules if needed.

Create the right-pane shell once. Give each section a stable root and an explicit update method. Keep orchestration and the current server snapshot in `runView.ts`; pass current data and action callbacks to sections.

Include actor, selected-node details, questions, instructions, presentations, definition revisions and adjustment, catalog, related runs, reports/files, execution, coordination, result, and history. Separate each section's status display from its interactive controls.

- Remove the normal-update path through `side.replaceChildren()` and the associated focus/caret reconstruction logic.
- Keep unchanged section roots and descendants connected. Do not detach and reinsert an active input as a substitute for preserving it.
- Use stable keys for dynamic items: question ID, instruction ID, presentation ID, revision ID, attempt ID, checkpoint ID, and history seq, scoped to the mounted run and display location.
- Insert or remove only the items whose membership changed. Update text, attributes, and button availability in place.
- Restrict canvas work to changes that affect its graph, status, or selection. Connection indicators and host-only facts must not invoke a full view render.
- Preserve open/closed `details` state across background updates.

A small local keyed-list helper is acceptable; a general virtual-DOM implementation is unnecessary.

**Completion criterion:** Background refreshes preserve input, file viewport, and history item identity while new server data remains visible.

## Step 4 — Define input ownership and safe action handling

**Files:** Persistent form sections introduced in Step 3; `src/runView.ts` action handlers.

Apply the same ownership rules to actor name, question answers, resume instructions, adjusted YAML, adjustment reason, and decision evidence:

- Initialize controls once. Treat user input as the draft source; do not assign `value` during unrelated background updates.
- Clear a draft only on an explicit user action or successful submission of that exact draft version. If the user edits while a request is in flight, preserve the newer text.
- Keep selection, textarea scrolling, focus, and composition through DOM continuity rather than restoration.
- Track composition only to handle exceptional invalidation or removal. Never freeze the entire view until composition ends.
- If a question is withdrawn or the run becomes cancelled/completed during composition, update its status and disable submission immediately. Keep the composing control attached and editable until composition finishes; then make it read-only with its draft retained and an explanation. Avoid disabling, hiding, or removing the active composing control.
- Historical views remain read-only. Explicit navigation may dispose the view through the existing unsaved-draft handling.

Remove the duplicate editable answer form from selected-node details. Show the question summary there with a link to its single canonical form in Questions. An explicit link click may reveal, scroll to, and focus that form; background updates may not. Update relevant tests directly rather than maintaining old selectors.

Persistent handlers must read the latest accepted run state at action time. Do not capture an old `rec`, question status, sequence, or definition revision in a long-lived closure. Keep `expectSeq` and `expectRevision` server checks, prevent duplicate submissions, and preserve drafts on errors and conflicts.

Bind an adjustment preview to its submitted YAML, reason, and base revision/sequence. Invalidate Apply when those inputs or its server preconditions change, and ignore preview responses that arrive after the draft changed.

**Completion criterion:** All forms survive background updates, submit current state safely, and never lose newer text because an older request completed.

## Step 5 — Preserve reading position during content changes

**Files:** Reports/files and history sections; `src/styles.css` if scroll anchoring requires an explicit policy.

Keep the open file's scroll container for the lifetime of that file selection. For unchanged content, perform no DOM write. When the same file changes, update the content and preserve vertical/horizontal offsets, clamped to the new range. Reset offsets when the user explicitly selects a different file. Re-read the open file on relevant full refreshes, with the stale-response guards from Step 2; report read errors without silently blanking previously loaded content.

Before applying a batch that changes the right pane's geometry, capture a visible stable content anchor and its offset relative to the pane. Prefer a visible history entry when the user is reading history. After the batch, adjust the pane scroll position by the anchor's displacement. If the anchor was removed, fall back to the nearest surviving item or the previous clamped offset.

Use one anchoring mechanism per scroll container; avoid double correction by native browser anchoring and application code. Verify changes above the viewport, not just updates within the visible section. When already at the top of newest-first history, remain at the top and show new entries. When reading older entries, preserve the reading position. Do not add automatic bottom-following behavior.

Explicit actions such as selecting another node, opening another file, or following a question link may intentionally change the viewport. Distinguish these actions from background refreshes.

**Completion criterion:** File offsets and the visible history anchor survive progress, file changes, and section height changes; explicit navigation still behaves predictably.

## Step 6 — Update only the display affected by each change

**Files:** `src/runView.ts` and section update methods.

- Replace whole-response `JSON.stringify` checks that trigger pane rebuilding with explicit section inputs and comparisons of displayed fields.
- Store heartbeat facts, but do not touch forms or history for an `executor.at`-only change. Availability, process state, queue state, and execution errors must still update.
- Patch notification and presentation displays independently, including presentation links within questions.
- Update connection indicators and connection-related banners on connection changes. Reconnect still performs a full data refresh.
- Keep relative-time updates scoped to their text nodes.
- Patch error displays independently. A host-facts fetch failure must not clear drafts, remove the last usable run, or suppress subsequent retries.

**Completion criterion:** Heartbeats and connection changes leave interactive DOM untouched while meaningful status changes remain timely.

## Step 7 — Validate the complete behavior and document the result

**Files:** `checks/runInteraction.ts`, affected existing browser checks, `README.md`, and a report beside this plan.

Extend the Step 1 checks to cover this matrix:

| Scenario | Required outcome |
| --- | --- |
| Heartbeat-only change, SSE progress, file change, disconnect/reconnect during input | Same connected control, focus, draft, selection, and composition |
| Actor, answer, resume, YAML, reason, and evidence forms | No unrelated value assignment; internal scrolling preserved |
| Selected-node question summary | Explicit link reaches the one canonical answer form |
| Question withdrawal or run cancellation during composition | Status updates; submission blocked; draft and composition preserved |
| Successful/failed/conflicting submission, plus typing during the request | Only the submitted unchanged draft may be cleared |
| Adjustment preview followed by edits or revision changes | Stale preview cannot enable Apply |
| History prepend and unrelated section growth | Visible entry stays at the same relative position, allowing only rounding tolerance |
| Same-file content update, different-file selection, out-of-order file responses | Preserve, reset, or ignore respectively |
| Slow polling, simultaneous SSE, and navigation away | No overlapping refresh batches, stale application, or post-disposal mutation |
| Historical run and malformed record | Read-only/error behavior remains correct without losing unrelated local state |

Use controlled browser network delays for races and real fixture run operations for SSE. Run `npm run check`, then the focused interaction check and affected existing checks, including `checks/runs.ts`, `checks/docsteps.ts`, `checks/appearance.ts`, and `checks/review.ts`. Resolve failures caused by the new DOM structure rather than keeping duplicate legacy paths. Record unavailable prerequisites or skipped checks explicitly.

Manually validate macOS Japanese IME: start conversion, leave candidates open through several heartbeat/progress updates, select a candidate, continue typing, and submit. Verify both an answer and the YAML textarea. Report synthetic browser results and native IME results separately; do not claim native validation from CDP alone.

Document the final update lifecycle and the browser-check command. Record changed files, test results, remaining limitations, and the source revision in the completion report. Before any later deployment, identify the installed release and use agdevworld's release workflow; a successful local build does not mean the operating service has changed.

**Completion criterion:** Automated regressions pass, native IME behavior is explicitly verified or identified as outstanding, and the report clearly distinguishes implementation, validation, and deployment status.
