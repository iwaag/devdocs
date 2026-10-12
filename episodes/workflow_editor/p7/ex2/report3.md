# Workflow editor p7/ex2 — step 3: persistent sections replace whole-pane rendering

Commit: agdevworld `4e9071f`.

## What changed

- **`src/runSide.ts` (new): `RunSide`.** The right pane's shell is built once. These sections each keep one root (`section.side-sec[data-key=sec:<name>]`) and an explicit update method:
  - actor;
  - selected node;
  - questions;
  - instructions;
  - presentations;
  - definition and its adjustment;
  - catalog;
  - related runs;
  - reports and files;
  - execution;
  - coordination;
  - result;
  - history.

  Sections that do not apply are hidden, never removed. Status display and controls are separate inside each section:
  - **Display** is built fresh and *patched* into the persistent block.
  - **Controls** are created once in the constructor, or once per question id, and are never rebuilt. They are the name, answer textareas, resume instruction, adjusted YAML, adjustment reason, evidence input, and their buttons.
- **`src/patch.ts` (new).** A small patcher, not a virtual DOM:
  - unchanged nodes get no write;
  - changed text is updated in its own text node;
  - attributes are synced, except the person-owned `open`;
  - children carrying `data-key` keep their element when inserted, removed or moved.
  
  Dynamic items are keyed by question id, instruction id, presentation id, revision id, attempt id, checkpoint id, history seq, file name and answer index. Keys are scoped to their section and its root (`q:`, `i:`, `p:`, `r:`, `at:`, `cp:`, `h:`, `f:`).
- **Delegated actions.** Built display carries no listeners. Node links, file links, close buttons, presentation start/stop, checkpoint retry, Stop run, notice dismissal and "Answer under Questions" name their action in `data-action`. `runView.ts` handles them in one click listener, using the state at click time. A reused element therefore never keeps a stale closure.
- **`src/runView.ts`** keeps the orchestration and the server snapshot: the step 2 coordinator, operations and file reads. It passes data and callbacks to `RunSide`.
  - Header, crumbs, summary, banners and legend are patched.
  - The canvas redraws only when the active revision, bundle, node states, selection or repositories change, which a JSON key compares.
  - A connection change updates only the live pill and banners.
  - The 30 s clock patches the summary and pane, so only relative-time text nodes change.
  - `side.replaceChildren()`, `drafts` value restoration and focus/caret reconstruction (`data-focus-key`) are removed.
- **`src/runCommon.ts` (new).** The shared helpers (`STATE_LABEL`, `ago`, `runHash`, `nodeLabel`, `activeWorkflow`, `chip`, `fact`, `line`, `nodeLink`, `fileLink`) move here. `projectView.ts` imports from here; there is no re-export.
- **Duplicate answer form removed here rather than in step 4.** With persistent controls, a second form for the same question in Selected node would have been a second owner of the same draft. The node detail now shows each question as a summary, with "Answer under Questions" or "Show under Questions" linking to the one canonical form. Clicking it scrolls the card to `nearest` and focuses the form with `preventScroll`. Background updates never do this.

## Verification

`checks/runInteraction.ts`, after two fixes to the check itself:

1. **The history mark is now a JS property.** It was a `data-` attribute, which the patcher correctly removes because built content does not carry it.
2. **The long file uses unbreakable lines.** The global `pre { white-space: pre-wrap }` wrapped the spaced filler, so there was no horizontal overflow to keep. The step 1 failure of `scrollLeft` (80 → 0) was real, since the old renderer replaced the `pre`, but the new fixture makes the assertion meaningful.

| Assertion | Step 1 (`6058c0f`) | Now |
| --- | --- | --- |
| Answer textarea connected and focused across SSE | fails | passes |
| Composing textarea connected after an update | fails | passes |
| Composition completes (`compositionend:日本語`) and typing continues: `Small 日本語 map` | fails | passes |
| Heartbeat-only reads: 0 mutations, 0 value writes, same control | fails | passes |
| File view `scrollTop` 400 and `scrollLeft` 80 kept | fails | passes |
| Visible history entry: same element, same offset (117.97 px before and after) | fails | passes |
| Lifecycle (step 2) | passes | passes |

History position is currently held by Chrome's native scroll anchoring (`overflow-anchor: auto`): the pane's `scrollTop` moved from 3182 to 3394 by itself, because the anchored `li` now survives. Step 5 decides whether to keep that mechanism.

Other checks:

- `npm run check`: 211 tests passed, plus type check and production build.
- Existing browser checks, all passing against this build: `runs.ts` (33 checks), `docsteps.ts`, `appearance.ts`, `review.ts` (including the adjustment preview/apply), `nodes.ts`, `conversation.ts`, `exec.ts` (including stop, the resume instruction and acceptance).

## Left for step 4

Steps 3 and 4 share code, so the following was deliberately kept simple here:

- Forms whose operation stops applying keep their draft. Their button is disabled, and they are removed only when empty and not focused. They do not yet become read-only after composition, with an explanation.
- A successful submit clears a control only if its text is still the submitted text. This compares values, not edit versions.
- There is no in-flight guard against double submission.
- An adjustment preview is invalidated by edits to the YAML only, not by edits to the reason or by revision/sequence changes.
