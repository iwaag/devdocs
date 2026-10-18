# p13 ex1 Step 3 — run guidance and context in the run view

## What changed (agdevworld `workflow_editor/src/`)

- **`runContext.ts` — the run's Context card**, in the main column between
  the focus card and the graph, in both modes. It is collapsible, and its
  open state is kept per viewer. Its summary reads, for example, "request ·
  1 instruction (1 not acknowledged) · attempt …: context recorded · 2
  changes for the next attempt". It holds:
  - **Original request**: the run's input file, read once, shown in place
    (long text is cut off, with a link to the whole file in the side pane).
    It is never editable.
  - **Instructions for this run**: the recorded instructions with their
    acknowledgement, and *Add an instruction for this run*. The form
    records the existing `instruction.receive` op through the view's normal
    `record()`, with the person's name as `from`: the same record a
    Conversation post or `wfe run instruct` makes. The form is not offered
    on a past reading, a cancelled run or a completed one, and the card
    says why.
  - **Used for this attempt**: for the latest attempt, one sentence:
    - prepared and delivered;
    - prepared but not delivered;
    - delivery not recorded;
    - prepared, but the snapshot is unavailable;
    - started before context was recorded;
    - no attempt yet.
    Then the sources it was given and its exact text.
  - **Changes for the next attempt**: "Settings changed since attempt …"
    with added/changed/removed sources, or "No changes". Also: "This
    attempt keeps what it was given…", the text the next attempt would get,
    and the session consequence in Detail.
  - **Settings for future attempts**: *Edit workflow instructions*, *Edit
    project instructions* and *Edit instructions for all projects* open the
    same `OverrideEditor` in place. Each says it applies to future attempts
    of every affected run, this one included, that the attempt in progress
    keeps what it was given, and that saving sends nothing to this run (the
    instruction form is the way to steer it now). A save reads the
    comparison again; no attempt, job or run record is touched. Links lead
    to the project's and workflow's own Context sections for shared purpose
    and guides.
  - **Detail**: *Every attempt*, newest first, with snapshot, fingerprint,
    delivery, each source's hash, file and revision, and exact texts.
    Snapshots that are unavailable or older than p13 show the recorded
    identities only.
- The comparison is read again only when the run's attempts change, after a
  save, or on *Read again*; ordinary refreshes only re-render.
- **Run side pane**: the Detail-only *Context* section (p13) is removed,
  since the card replaces it. `goto-section context` reveals the card.
  `RunSide.setActor` keeps one name across the card and *Recording as*. The
  old `runContextPanel` is deleted.
- The name fields of the context editors are labelled *Revision author* and
  *Instruction author*, so they are not confused with the views' existing
  *Your name* fields.

## Checks

`checks/contextEditors.ts` now also runs these journeys, in Simple.

Journey 4:
- The request is readable in place.
- The attempt's context and its delivery are stated.
- Settings saved since the attempt show as changes.
- Editing the workflow instruction from the run states its reach. After
  saving, the workflow instruction appears as a next-attempt change, while
  "Used for this attempt" is unchanged.
- The run record's sequence and attempt count are unchanged.
- The graph selection is kept.

Journey 5:
- *Send to this run* records exactly one instruction (i1) with its author.
- No override holds the text, and it is not listed as a next-attempt
  setting.
- The form clears.

Other cases:
- The section toggles from the keyboard, and every context control has a
  label.
- Detail lists every attempt with its snapshot.
- A deleted snapshot reads "unavailable".
- A run before its first attempt says so.
- A cancelled run hides the form and says why.

Results: all pass in Chromium and WebKit, desktop and 390 px. A summary that
would not wrap at 390 px was fixed (`.ctx-toggle` wraps its text).

Regressions:

- `checks/context.ts` (p13), with its run part moved to the card: all pass
  in Chromium and WebKit.
- `checks/runInteraction.ts`: one failure, "an unchanged submitted draft is
  recorded and cleared". It fails identically on the Step 2 baseline,
  twice.
- `checks/simplerUi.ts` and `checks/displayMode.ts` show only their
  baseline failures (see report2).
