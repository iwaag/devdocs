# p14 ex1 Step 1 — compact rows and one reading surface

## Before

The installed release (p14, `ed688f6`) showed every block as a card with a
kind chip, a scope chip, *Managed by*, *Last edited by* with age and run,
observation age and coverage, the latest check, line and byte counts, the
whole body in a grey preformatted box, detail links, and Edit/Remove on
every card. Home's section also opened with an empty *New instruction*
form when reached through `#/home/context` before the blocks had loaded
(`reveal(true)` called `focusOrAdd()` on an empty list). Captures: the
area's ignored `workflow-editor-p14ex1/before/` (Chromium, WebKit, 390 px;
Home, the forge page and the forge Context section).

## What changed (`src/contextParts.ts`)

- **Reader.** One side panel per page (`aside.ctx-reader`), below the
  sticky header so crumbs and the display switch stay usable, full width
  on a phone. It shows content its owner keeps; closing or opening another
  item never touches a draft. Esc or *Close* returns focus to the row it
  was opened from. A route change closes it (`main.ts`).
- **BlockCard** is now a row plus a panel, sharing one state and one
  draft:
  - row: the title at the Workflows-list level (15.5 px, 650), one meta
    line — manager, scope only when the grouping does not say it, at most
    two short exceptions (*Unsaved changes*, *Last refresh failed / was
    partial*, *No evidence recorded*, *A check failed*, *Partly verified*,
    *Not verified*, *Edited outside WFE*, *Cannot be read*) — and one
    action;
  - a short instruction (≤ 3 rendered rows at the list's width, wide
    characters counted twice) reads in full in its row and its action is
    *Edit*; everything else is *Open*;
  - panel: title, kind · *Managed by* · scope; banners (problem, external
    edit, failed refresh with its detail, concurrent change, results); the
    body as prose (paragraphs, lists, headings, `code`, built as DOM text);
    *View resource details* (or one link per detail), *Ask <manager>*
    (the consultation workflow's project, in a new tab), *Edit*;
    *Evidence & history* (last editor with time and run, each evidence
    item's status, basis, observation time, coverage and link, check,
    refresh, the revision history loaded on first open);
  - editing in the panel: title, kind (new blocks only), text, its size
    in rendered rows and delivered bytes with the all-projects advice,
    Save / Discard / Cancel, author, **Remove**, the "managed by someone
    else" and "wider scope" notes; conflict recovery unchanged;
  - Detail adds file, revisions, section hash, delivered size, id,
    detail references and the consultation workflow.
- **BlockList**: rows; **one Add** with a two-item menu (instruction /
  environment information) that the host places in its heading; a new
  block opens in the panel in edit mode, an untouched one disappears when
  the panel closes, a typed one stays as a row marked *Unsaved changes*.
  `focusFirst()` replaces `focusOrAdd()`: an entrance never creates an
  empty block.
- Hosts adjusted only for the Add placement (project/workflow/Home
  sections, run card settings, overview quick edit, overview chooser).
- CSS: the p14 card styles replaced by row and panel styles.

## Checks

`checks/contextBlocks.ts` rewritten for rows and the panel:
list content without sizes, editors, times, runs or Remove; the short
instruction inline with Edit; the panel's prose, details link, Ask, Edit,
evidence and history; focus returning; a human edit of cagent's block and
cagent's later update; a draft kept through closing the panel and a mode
switch, then a conflicting save; the failed refresh in row and panel;
Detail's hash and size; inherited rows naming the scope; Add with a kind
choice; the run card; opening and closing a block changing no revision and
no next-attempt fingerprint.

| Check | Chromium | WebKit | Chromium 390 px | WebKit 390 px |
| --- | --- | --- | --- | --- |
| `contextBlocks` | pass | pass | pass | pass |

Unit tests 401/401. Found while running WebKit: the Add menu closed on
`focusout` before the click landed (Safari does not focus a clicked
button); it now closes on a pointer outside it or on keyboard focus
leaving it.

`context` (the overview) still passes; `contextEditors` still asserts the
p14 cards and moves with Step 2's section changes.

## Revision

agdevworld `cfcc2e9`.
