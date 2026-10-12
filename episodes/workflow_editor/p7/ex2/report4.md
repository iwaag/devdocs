# Workflow editor p7/ex2 — step 4: input ownership and safe action handling

Commit: agdevworld `875681b`.

## What changed (`src/runSide.ts`)

- **`Owned` control wrapper.** Every input in the pane goes through one rule set, except the actor name. That covers the answer textareas, resume instruction, adjusted YAML, adjustment reason and decision evidence. The rules:
  - **No background value writes.** The text in the control is the draft source. Values are written only at initialization (the adjusted YAML from the definition in force) and on clearing after a successful submission.
  - **Exact-draft clearing.** Edits bump a version. `submit()` remembers the text and version it sent. After a recorded result, it clears the control only if both are unchanged. Text typed while the request was in flight is kept.
  - **No double submission.** While a submission is in flight, the control's buttons are disabled and a second `submit()` is a no-op. The Result buttons share one busy flag across Accept and Reject.
  - **Composition-aware closing.** When the operation stops applying, the buttons are disabled at once and the reason is shown beside the control. Readonly is applied only after `compositionend`. The control is never disabled, hidden or removed while it is active (focused or composing). An empty, inactive control whose operation no longer applies is removed. One that holds text stays, read-only, with the text kept and the reason shown:
    - question withdrawn, taken up, or run cancelled;
    - run no longer held (resume);
    - run completed or cancelled (adjustment).
- **Actions read the latest state.**
  - Answer: checks the question's current state before sending.
  - Resume: checks that the run is still held.
  - Accept: checks that the run is completed.
  - `expectSeq` comes from the shown record at send time (`runView.record`). No long-lived closure captures `rec`, a question, a sequence or a revision. Drafts are kept on errors and on 409 conflicts.
- **Adjustment preview binding.** A preview is stored as `{text, reason, revision, seq, view}`. Apply is enabled only while all of the following hold:
  - the YAML and reason equal what was inspected;
  - the revision in force and the run's sequence are unchanged;
  - the result was applicable;
  - nothing is in flight.

  A stale preview stays visible with "Edited since this preview" or "The run moved on since this preview (r1, seq 3 → r1, seq 4)". Apply sends the *previewed* text, reason, `expectRevision` and `expectSeq`, and the server checks both again. A preview response that arrives after the draft changed is discarded and enables nothing. After a successful apply, the form refills from the new revision only if its text was not edited in the meantime.
- **Historical views** create no forms. Explicit navigation keeps the existing unsaved-draft prompt (`isDirty()` now reads the controls themselves).
- **Duplicate answer form.** Already removed in step 3: Selected node links to the single form under Questions.

## Verification

New sections 7–10 of `checks/runInteraction.ts`:

| Scenario | Result |
| --- | --- |
| Selected node `ask`: no textarea in the node detail, exactly one "Answer to q1" control, its link focuses that control under Questions | passes |
| Submission delayed 1.5 s: a second click while in flight, then typing " or large" | passes: one answer `Medium` recorded; the control keeps `Medium or large`; the button was disabled in flight |
| Submitting the unchanged draft | passes: recorded, control cleared |
| Injected 500 on submit | passes: draft kept, "Not recorded: injected failure" |
| Question withdrawn mid-composition (`あお`) | passes: card turns `q-withdrawn` and the button is disabled at once; the control stays connected, focused and editable. After committing `青`: read-only, text kept, reason shown |
| Adjustment preview on a boundary run | passes: Apply enabled for the fresh preview; disabled after a reason edit, enabled again when restored; disabled with "moved on" after the sequence moved |
| Preview answered after the YAML changed (1.5 s delay) | passes: no preview shown, Apply disabled |

Sections 1–6 still pass. The first attempt at the preview test used `ship/run-001`, where `build` is running. The server correctly refused the adjustment (`mid-node`: adjustments apply only at a node boundary), so the test now uses a second run where nothing runs. Run cancellation during composition is part of the step 7 matrix.

Against this build:

- `npm run check`: 211 tests passed, plus type check and production build.
- `runs.ts`, `docsteps.ts`, `appearance.ts`, `review.ts`, `nodes.ts`, `conversation.ts` and `exec.ts`: all passed. This includes review's preview/apply flow under the new binding and exec's stop, resume instruction and acceptance.
