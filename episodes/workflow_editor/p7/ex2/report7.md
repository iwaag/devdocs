# Workflow editor p7/ex2 — step 7: validation and documentation

Source: agdevworld `e16f2c4`, from the investigated base `6058c0f`, in seven commits:

| Step | Commit | Content |
| --- | --- | --- |
| 1 | `861afe8` | isolated reproduction checks |
| 2 | `15b6e8f` | one refresh lifecycle (`src/runRefresh.ts`) |
| 3 | `4e9071f` | persistent sections (`src/runSide.ts`, `src/patch.ts`, `src/runCommon.ts`) |
| 4 | `875681b` | input ownership and safe actions |
| 5 | `29bfcd8` | reading position |
| 6 | `0d4f335` | scoped updates |
| 7 | `e16f2c4` | matrix, forms attached only when they apply, README |

## Change made in this step

- **Forms attach only when their operation applies.** The matrix's historical-view row found that the resume form, and in principle the decision form, were attached at construction inside hidden sections. A historical view therefore carried a `textarea` in its DOM. Controls are still created once, but the updates now attach them: resume while the run is held, decision in a current view, adjustment while the run can be adjusted.
- **README.** It now documents how the run view stays live (coordinator, persistent pane, input ownership, reading position, scoped updates) and the `checks/runInteraction.ts` command.

## Automated validation (Chromium, `checks/runInteraction.ts`: 38 assertions, all passing)

| Scenario | Required outcome | Result |
| --- | --- | --- |
| Heartbeat-only change, SSE progress, file change, disconnect/reconnect during input | same control, focus, draft, selection and composition | passes. Covered by: SSE during typing and composition; composition across 4 heartbeat reads and a file change; composition across a service restart; focus and selection [3, 9] kept across SSE, file change and the host cadence |
| Actor, answer, resume, YAML, reason and evidence forms | no unrelated value assignment; internal scrolling kept | passes: 0 value writes on all six controls; answer and YAML `scrollTop` kept |
| Selected-node question summary | the link reaches the one canonical form | passes: no form in the node detail, exactly one "Answer to q1" control, the link focuses it |
| Question withdrawal or run cancellation during composition | status updates, submission blocked, draft and composition kept | passes for both: editable while composing, then read-only with text and reason |
| Successful, failed and conflicting submission, plus typing during the request | only the unchanged submitted draft is cleared | passes: in-flight typing kept, double click records once, unchanged draft cleared, 500 and 409 keep the text with the reason |
| Adjustment preview followed by edits or revision/sequence changes | a stale preview cannot enable Apply | passes: reason edit, sequence move and a late preview response all leave Apply disabled |
| History prepend and unrelated section growth | the visible entry stays at the same relative position | passes: 117.97 → 117.97 px for a prepend; 118.0 → 118.0 px when Reports and Instructions grow above; at the top of the history the heading holds and new entries show; at the pane top it stays at the top |
| Same-file update, different-file selection, out-of-order file responses | preserve, reset, ignore | passes: 500/120 kept with new text (clamped to 496 when the file shrinks); 0/0 for another file; a late response for the first file is ignored; a failed re-read keeps the text |
| Slow polling, simultaneous SSE, navigation away | no overlapping batches, stale application, or mutation after disposal | passes: at most 1 read of each kind in flight with 1.2 s delays; sequence monotonic; no reads after leaving mid-read |
| Historical run and malformed record | read-only and error behaviour correct, unrelated local state kept | passes: no forms at a commit; a malformed `run.json` disables submission but keeps the control and draft, and repair re-enables it |

Also in the check:

- a heartbeat-only window shows no element added, removed or re-attributed anywhere in the pane;
- an executor turning unavailable shows within 1.5 s;
- a host read failure appears beside the last facts and clears on recovery.

## Regression (all against `e16f2c4`'s build)

| Command | Result |
| --- | --- |
| `npm run check` (tsc, 211 tests, vite build) | pass |
| `node checks/runInteraction.ts` | 38 passed |
| `node checks/runs.ts` | 33 passed |
| `node checks/docsteps.ts` | 35 passed |
| `node checks/appearance.ts` | 15 passed |
| `node checks/review.ts` | 22 passed |
| `node checks/nodes.ts` | 20 passed |
| `node checks/conversation.ts` | 12 passed |
| `node checks/exec.ts` | 20 passed |

Prerequisite (recorded in step 1): the Playwright cache needed `chromium_headless_shell-1234` for the pinned `playwright-core` 1.62.1. It was installed with the project's own CLI.

The p1 seed-based checks (`step2–4.ts`, `e2e.ts`) and `rehearsal*.ts` were not run. They exercise the definition editor and the live rehearsal workspace, not the run view's right pane. `rehearsal-runs.ts` uses the answer and evidence selectors, which are unchanged (`Answer to <q>`, `Decision evidence`, `.question[data-question] button.primary`).

## Native macOS Japanese IME — outstanding

Not validated yet. All composition results above are synthetic (CDP `Input.imeSetComposition` / `Input.insertText` in headless Chromium), and they do not show how the native candidate window behaves. The manual check needs a person at the keyboard:

- **Fixture.** A private service with an open question, a progress record every 3 s and an executor heartbeat every 2 s.
- **Procedure.** In both the answer textarea and the Adjusted definition textarea:
  1. start a conversion;
  2. keep the candidates open through several updates;
  3. pick a candidate;
  4. keep typing;
  5. submit or preview.

**Outcome (2026-10-12).** A fixture was provided on port 8203. When the final report was requested, its run held no answer from the browser and no adjustment preview. The manual procedure was not carried out, and native IME remains unvalidated. The fixture was stopped and its data removed.

## Status

- **Implementation:** complete (steps 1–7), committed and pushed to agdevworld `main`.
- **Validation:**
  - Automated: complete, as above.
  - Native IME: outstanding.
- **Deployment:** not done, and not part of this plan. `wfe-release.sh status`: the installed release is `6058c0f` (previous `cde57cd`), the very revision investigated. So the investigation's open point is settled: the reported behaviour matched the deployed code. A local build does not change the operating service. Deploying uses `agdevworld/scripts/wfe-release.sh deploy .local/agdev` from pj-agdev.

## Remaining limitations

- **Anchoring.** It is measured at the pane's top edge. An update that removes the anchored item falls back to the previous offset, which can shift slightly. In this fixture a recorded run never removes an item: questions, history and revisions only grow.
- **Artifacts outside the run folder** have no listing stamp, so an open one is re-read on every full refresh.
- **The 30 s clock** still patches the whole pane. Only relative-time text nodes change, but the fresh trees are built each time.
- **The canvas** keeps its own rendering. It is redrawn when node states change, which is unrelated to the right pane.
