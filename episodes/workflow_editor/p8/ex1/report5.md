# Workflow editor p8/ex1 — step 5: switching without losing the person's state

**Source**: agdevworld `ee52112`. **Deployment**: none; the installed release is still `8cda5c9`.

## Why the work survives

The switch changes visibility only (step 1).

- **No duplicated controls.** No form, field, section or control exists twice. None is rebuilt by a switch.
  - The workflow draft, the project draft and the request form keep their one owner and their one element.
  - So do the approver and actor fields, the answer, resume, evidence and adjustment controls, and the selection.
  - Each disclosure keeps its own open state: Intent & approvals, the inspector groups and the run's side sections.
- **No view lifecycle on a switch.** The refresh coordinator, scoped host updates, stale-response guards and server preconditions are untouched.
- **Redraws on a switch.** Only two things redraw:
  - **the canvas**, because Simple cards have other metrics (step 3);
  - **the project's hero**, so it can say that Project settings hold unsaved edits. It is replaced alone.
- **Disposal.** Each view removes its display listener on dispose.

What the switch adds on top (`displayMode.ts`):

- **Composition.** A switch asked for during a composition is held, and applied when the composition ends.
- **Scroll memory.** Each mode's window scroll, and the scroll of every `data-keep-scroll` element, is kept per screen (for now that element is the run's side pane). It is restored on return; the browser clamps the value if the content changed.
- **Focus.** A focused control that the new mode hides gives the keyboard to the toggle.
- **Sticky top bars (new).** The top bars stay in view (`position: sticky`). The toggle is therefore one click away anywhere on a long Detail page, and a switch never needs a scroll to reach the toggle first. The editor's inspector and the run's side pane now stick below the bar instead of under it.

**Adjustment previews** are bound to their input, as in p7. Switching away and back leaves Apply exactly as it was: enabled for an unchanged preview, disabled once the run moves on. A display preference never re-validates a preview.

## Validation

`checks/displayMode.ts` passes **94 of 94** on the state fixture. The step 5 part covers the following.

**Workflow editor**

- These survive Simple → Detail → Simple:
  - a draft name and a node description;
  - the open Intent & approvals panel with an intent draft;
  - a declared approver;
  - the selection.
- Simple says "Unsaved changes".
- Leaving asks "Discard unsaved changes?" in Simple too.
- No display listener remains on the dashboard afterwards.

**Project**

- The request draft is the same field in both modes.
- Simple's hero says Project settings hold unsaved changes.
- The settings draft is still there to save.
- Detail's window scroll (600 px) is restored after a visit to Simple.

**Run**

- **Drafts.** A resume instruction and decision evidence survive.
- **Adjust future work, opened from Simple**, focuses the adjustment form. An adjustment with its reason is previewed, and then:
  - A synthetic `compositionstart` on the reason defers a click on Simple until `compositionend`.
  - The hidden reason field hands the keyboard to the toggle.
  - Back in Detail, the text, the reason and the open Definition section are intact, and Apply is still enabled.
  - A background record then makes the preview stale and disables Apply.
- **Reading position.** Three open Detail sections, a side pane scrolled to 500 px and an open 300-line file scrolled to (400, 60) all come back as they were. Simple shows the same file at the same offset.
- **Background changes.**
  - Two answer drafts survive a background update between switches.
  - A question withdrawn between switches keeps its draft, read-only with its note.
  - Recording the other answer records exactly one answer; run.json was read to confirm it.
- **Disconnection** was covered in step 4: a service outage and reconnection in Simple.
- **Listeners.** After five views (three runs, a workflow, a run), exactly one display listener exists.

`checks/simplerUi.ts` and `checks/runInteraction.ts` pass again with the sticky top bars.

## Composition: synthetic, not native

The deferral was exercised with synthetic `compositionstart` and `compositionend` events. A real macOS Japanese IME has not been tried in this step. With a native IME, a click elsewhere normally commits the composition before the click arrives, so the deferral is a second line of defence. Step 6 records the manual check's status.
