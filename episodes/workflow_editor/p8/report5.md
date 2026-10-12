# Workflow editor p8 — step 5: progressive disclosure that keeps interaction state

Changed in agdevworld `workflow_editor/`:

- `src/runSide.ts`: sections become disclosures, plus attention labels, reveal on navigation, and anchoring across closed sections.
- `src/runView.ts`: the delegated click listener is removed when the view is disposed.
- `src/inspector.ts`: group open state is recorded on activation.
- `src/canvas.ts`: the in-view note's Fit button can be clicked.
- `src/styles.css`.
- New check `checks/simplerUi.ts`, and adapted `runInteraction.ts`, `runs.ts`, `review.ts`, `docsteps.ts`.

## Disclosure in the run view's side pane

Every record in the side pane is a labelled, expandable section: Questions (resolved), Instructions, Presentations, Definition (with Adjust future work), Node catalog, Related runs, Reports and files, Execution and publication, Coordination, Result and decision, History.

- **The label carries the section's state**:
  - "Questions (2 resolved)";
  - "Presentations (1 live)", "(1 recorded open, not served)" or "(1 stopped)";
  - "Reports and files (2, 1 missing)";
  - "Execution and publication (held / host facts unavailable / n publication problems)";
  - "Coordination (Zulip) — needs attention / not configured";
  - "Result and decision (accepted)";
  - "History (40)".

  A section that holds a problem or an action is marked in its label (amber or red with a dot) while closed: a held run, a missing artifact, an unserved presentation, a publication failure, a failing delivery, unacknowledged instructions.
- **Required actions are not inside sections**. Open questions, the resume form and a missing decision are in the focus card (step 4). The focus card also gains a **Needs attention** list for problems that wait on the person but whose controls are in a section: a failed or `attention` publication checkpoint, a presentation whose stop could not be verified, failing Zulip delivery, or unreadable host facts. Each item links to its section.
- **The selected node** is an always-open card at the top of the pane, independent of the sections, with its close button.
- **Open state is the person's.**
  - Sections start closed. A toggle the person makes is kept in this browser (`localStorage`, a per-viewer convenience that tolerates a blocked store).
  - Background updates never open or close a section.
  - Explicit navigation opens the section it leads to: a file link (Read final report, Original request, a document from a node), "Adjust future work" (which also opens the form and focuses it), the focus card's section links, and "Show it" for a resolved question.

## Interaction state through disclosure and updates

The p7/ex2 machinery is unchanged: one refresh coordinator, host-only updates, stale-response guards, `expectSeq` / `expectRevision`, persistent sections patched in place, controls created once (`Owned`). Adjustment previews stay bound to their exact text, reason, sequence and revision.

What changed for the new nesting:

- **Anchoring** (`captureAnchor`) still works on the pane's direct child sections. Only *displayed* keyed items are candidates now, so a closed section is held by its label. History is held by its toggle when its newest entry is the first visible one.
- **Section bodies** are inside `div.sec-content`. The file view, the adjustment form and the side decision form are attached there, never rebuilt.
- **A closed section keeps its controls attached** (only hidden). A draft in it is the same element with the same text when it is reopened.

## Defects found and fixed in this step

1. **Run views acted on later views' clicks.** Since p7/ex2 the run view handled clicks through a listener on the shared app root (`#app`), and dispose never removed it. After visiting several runs, one click reached every earlier, disposed view, each acting on *its own* run. The new check found this as `report.md` reads against runs that were no longer shown. With Stop or Cancel, the prompt would have appeared once per earlier view, and each confirmation would have recorded the operation on that earlier run. The listener is now removed on dispose. The installed release `a571ae2` has this defect.
2. **An opened inspector group could fold.** Chrome dispatches `toggle` asynchronously, so a structural edit made right after opening a group (choosing a color) could rebuild the inspector before the open state was recorded. The state is now recorded on the summary's activation (click, Enter or Space).
3. **The canvas note's "Fit to see all" did nothing.** The canvas's pan handler captured the pointer on the note, so the button never received its click. Step 3 had probed Fit through the zoom controls only.

## Checks

| Check | Result |
| --- | --- |
| `checks/simplerUi.ts` (new) | **pass, 71 assertions**. Contents are listed in report6. Those for this step: background updates (progress, a withdrawal) neither open nor close a section; a draft in a closed section is the same control with its text when reopened; another question's typed answer stays in the focus card; a withdrawn question without a draft moves to the resolved records; a question withdrawn *while its answer is focused* keeps the control in place, read-only, with text and reason; the person's open sections survive navigation; closed sections name a held run, a missing artifact and a failed publication; no failed reads and no console errors outside the deliberate outage |
| `checks/runInteraction.ts` | pass, 39 assertions, with every section opened first as a person would; the choice persists across the check's navigations |
| `checks/runs.ts` | pass (sections opened first) |
| `checks/review.ts` | pass (sections opened first) |
| `checks/docsteps.ts` | pass; coordination is read from the section label, which states it closed or open |
| `npm run check` | pass (211 tests) |
