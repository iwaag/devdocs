# Workflow editor p8 — step 6: validation and screenshot comparison

Changed in agdevworld `workflow_editor/`:

- `src/canvas.ts`: a card that had the keyboard focus gets it back when the cards are rebuilt.
- `src/workflowView.ts`: a new node's name field is focused at once, not one frame later.
- `checks/states.ts`: a long, mixed-language workflow and run.
- `checks/statesShots.ts`: more capture cases.
- The checks updated for the new layout: `ui.ts`, `e2e.ts`, `appearance.ts`, `nodes.ts`.

## Automated checks

All were run on the final source of this step. Every private service ran on its own port, data directory and presentation ports.

| Check | Prerequisites | Result |
| --- | --- | --- |
| `npm run check` | — | pass: type check, 211 tests, production build |
| `checks/simplerUi.ts` (new in step 5) | build | **pass, 71 assertions** |
| `checks/runInteraction.ts` | build | pass, 39 assertions |
| `checks/runs.ts` (`--repeat 20`, the default) | build | pass |
| `checks/docsteps.ts` | build | pass |
| `checks/review.ts` | build; a presentation instance on this host (ports 18390–18399) | pass |
| `checks/appearance.ts` | build | pass |
| `checks/nodes.ts` | build | pass |
| `checks/conversation.ts` | build | pass |
| `checks/ui.ts` | a helper library, run through `checks/e2e.ts`. That needs the dev service on 8095 (`server/main.ts --registry ../.local/wfe-fixture/registry.json`) and Vite on 5175, both started for the run and stopped afterwards; it reseeds only the ignored dev fixture `agdevworld/.local/wfe-fixture` | **e2e pass, 45 assertions** |

What `simplerUi.ts` asserts, against the representative states of `checks/states.ts`:

- **Definition screen**:
  - the inspector is closed with nothing selected;
  - name, purpose and both approval states are visible;
  - the intent and the approval actions are behind "Intent & approvals", with the reason they are disabled. The approver survives closing the panel;
  - the readable-scale framing and its in-view count, and Fit;
  - the Add node menu: base types, templates, User nodes link, focus in, Escape out;
  - name and description come first in the node inspector;
  - an opened group survives a structural rebuild;
  - closing and reopening the inspector keeps the draft and returns the focus;
  - the zoom and pan are unchanged by opening or closing the inspector;
  - a validation issue selects its node;
  - settings and their focus return;
  - edges and generated steps are inspectable, and generated steps are read-only;
  - every connection starts and ends at its cards' anchors;
  - every report icon is inside its card;
  - small graphs open at 100 % and large ones readable from the start.
- **Run view**, in each state of the matrix: the focus card's kind and next action (review link, answer forms, receipt with "Record another answer", resume form, failure with recovery, accepted-while-cleaning-up, the step-labelled outcomes with cleanup apart, the decision offered when missing, cancellation with the open presentation's Stop, two open questions). Also:
  - the title band keeps execution, decision and executor activity apart;
  - the historical view has no forms or host facts;
  - a malformed record draws nothing;
  - a failed publication is named in the focus card and in the closed section's label;
  - "Read final report" and "Adjust future work" open their sections.
- **Disclosure and updates**: covered in report5. Sections are neither opened nor closed by updates; drafts in closed sections are the same controls; a resolved question moves only without a draft or focus; a question withdrawn under focus stays read-only with text and reason; open sections persist per browser.
- **Connection loss and reconnection; layout**: the page fits 1280 × 800 without horizontal scrolling, with the review link and the answer form on the first screen. No failed reads and no console errors (outside the deliberate outage).

### Selector and expectation changes in existing checks

- **`ui.ts`**: the helpers open Workflow settings before adding a binding, use the Add node menu (`menuitem`), open the Connections group before connecting, and use `exact` for Fit (the canvas note also offers "Fit to see all"). New helpers `openSettings`, `openIntent`, `openGroup`.
- **`e2e.ts`**: opens Intent & approvals before the intent and the approvals. Two expectations predated p8 and were already wrong: since p4 a delegate node carries a "display only" warning, so the scenario's "1 error, 0 warnings" and "valid again" (no warnings) could no longer hold. The check now asserts the error count and "no errors".
- **`appearance.ts` and `nodes.ts`**: open the Add node menu, and the Appearance group, before using them.

## Defects found by validation (fixed in this step)

- **Focus taken after adding a node.** After adding a node, the name field was focused one animation frame later. e2e's typing landed in the wrong field: the description went into the name. A person typing quickly would hit the same race. The focus is now set synchronously, and only the reveal of the card waits for layout.
- **Focus lost on keyboard selection.** Selecting a card with Enter rebuilt the cards and the focus fell to the page. It now returns to the same card.

## Manual verification

On the private fixture, at 1440 × 1000 unless stated.

- **Keyboard**:
  - **Run view tab order**: breadcrumb links, Refresh, Open current definition, Run details, the review link, the question's presentation link, the answer field, Record answer, the focus card's links, the canvas, every card and step in turn, the in-view note's Fit, then the side pane.
  - **Run view sections**: a section toggle opens with Enter.
  - **Editor**: Enter on a card or report icon opens its inspector with the focus kept on the card; Escape closes it and the focus stays there; Enter on "Intent & approvals" moves the focus into the intent field.
  - **Menu and panels**: the Add node menu moves the focus in, the arrow keys move within it, and Escape returns the focus to the trigger. Closing settings or the inspector returns the focus to what opened it (asserted in `simplerUi.ts`).
- **Focus when panels open and close**: the cases above, and the run view's explicit navigation. "Answer it" focuses the question's form, "Adjust future work" focuses the YAML, and file links scroll to the file view without taking focus from it.
- **Long English and Japanese text** (`longtext` workflow, `run-jp`):
  - a mixed-language title of about 90 characters fits on one line in the run view and is cut with an ellipsis in the editor's name field;
  - the purpose preview is cut with an ellipsis;
  - Japanese and English card names wrap inside the card (cards grow, edges stay attached);
  - descriptions keep their two-line limit on the card and appear in full in the inspector and node details;
  - a two-paragraph Japanese question wraps in the focus card.

  Captured in `def-longtext.png`, `run-longtext.png` and `run-longtext-1280.png`.
- **IME**: synthetic composition (CDP `Input.imeSetComposition` and `insertText`) is exercised by `runInteraction.ts` across SSE updates, heartbeats, a file change, a disconnect, a withdrawal and a cancellation, all in the focus card now. **Native macOS Japanese IME during background updates was not verified.** It needs a person typing with the system IME in a real browser. It is outstanding, as it was in p7/ex2.

## Screenshots

There are 39 real captures in [image_after/](image_after/) with the index and the capture conditions: 29 at 1440 × 1000 and 10 at 1280 × 800. They include every state named in the plan: definition with and without selection, branch and join, large graph, open plan question, live review, accepted while cleaning up, held and failed, completed, historical, and expanded details. They also cover the remaining fixture states and long text. The baseline of the same states with the old UI is in [image_before_states/](image_before_states/).

## Review against the three questions

### 1. Is the current situation and the person's role clear on opening the screen?

**Run view: yes, in every state of the matrix.**

- **Before**: in `image_before_states/run-review.png` the only cue was the review button in a dark header among timestamps. In `run-failed`, the reason took a node selection to find. `run-accepted` differed from an in-progress run only by a small chip.
- **After**: the focus card states the situation and the person's part in its heading and first line:
  - "Ready for your review";
  - "A question waits for your answer" / "2 questions wait";
  - "Your answer is recorded … the next move is theirs";
  - "Held (stopped)";
  - "Verify failed";
  - "Accepted — cleanup is still running";
  - "Completed — no decision recorded";
  - "Work in progress — nothing is needed from you now".

  The title band keeps the recorded state, the decision and executor activity apart; the header shows the connection.

**Editor: yes for its purpose.** Name, purpose, approval states and validation are on one band, and the graph is the main object, opening readable.

### 2. Can the person reach the outcome or the evidence without searching files or old questions?

**Yes.**

- **Review**: the live review is the first button.
- **Completed run**: its recorded outcomes are listed by step with cleanup apart, and "Read final report" (`report.md`) and "Original request" are one click and open in the Files section.
- **Questions**: a plan question links its `plan.md`, and a review question its presentation.
- **Old questions** are a closed "Questions (n resolved)" section, not the reading surface.

The limit is the data. There is no generated "What changed" summary; see the deviations.

### 3. Can the person reach precise details and controls without losing their place or draft?

**Yes, by the checks above.**

- **Where things are**:
  - Run details holds timestamps, backend, attempts and provenance;
  - each record is a labelled section that keeps its open state;
  - the selected node is a separate card;
  - the editor's settings and intent are explicit panels.
- **What is kept**: drafts, focus, composition, nested file scrolling and the history anchor survive disclosure and background updates (`runInteraction.ts`, `simplerUi.ts`). Opening the inspector does not move the graph.

## Deviations from Take 3, and why

| Take 3 | Implemented | Reason |
| --- | --- | --- |
| A "What changed" heading with a prose summary, test and review results | The presentation's title as the run's name, then the **recorded outcomes** labelled by step, with cleanup listed after the confirmation | No summary exists in the records. Generating one was out of scope, and the plan forbids fabricated prose. The outcomes carry the same facts (e.g. "Verify: 31 of 31 passed") with their origin |
| Every completed run shows a "What changed" panel and a stopped-server panel side by side | One focus card whose content depends on the state (11 kinds) | The concept showed one completed run. Active runs need the question, the resume or the failure there instead |
| Run graph in two rows, fully visible | Stored positions, a readable 72 % view centred on the current position, an in-view count and Fit | The plan keeps stored positions and arbitrary DAGs. A two-row wrap would be a new layout algorithm and would misrepresent branches |
| Definition graph in two rows at 100 % | Stored positions; a graph that does not fit opens readable from its start | Same reason. Small graphs open whole at up to 100 % |
| Report icons as document icons inside cards | Done; the final report is a dashed "Document" endpoint, not a card like an authored node | Keeps the documentation step distinct, as Take 3's own notes require |
| Side panel shows the selected node when nothing else is chosen | Selected node card above labelled sections; no placeholder card when nothing is selected | No data to show without a selection |
| "Questions · 2 resolved", "Presentations · 1 stopped", … as rows | Same idea, with problems marked in the label and a "Needs attention" list in the focus card | The concept had no failure states to show |
| Header shows "Live connection" | "● live / not live / history" pill in the header | Unchanged semantics, shorter |
| Inspector sections as rows with chevrons (Appearance, Repositories, Connections) | Expandable groups with one-line summaries; Repositories open by default | As the concept; the open state survives rebuilds |

Dashboard, Project Editor and User nodes are unchanged apart from shared styles. The plan limits this iteration to the two primary views.

## Limitations

- **Native macOS IME**: not verified (above).
- **Card descriptions at 72 %** are about 9.5 px. Names are readable; descriptions need 100 % or the inspector. This is the cost of keeping stored layouts readable without re-arranging.
- **The editor's canvas keeps its view when the inspector opens.** The selected card is revealed, but the rest of the graph is not refit. This is by design, so the person's zoom and pan are never reset.
- **The working-run capture's host facts are synthetic**. The run view's display of a live process is asserted in the check with the same interception.
