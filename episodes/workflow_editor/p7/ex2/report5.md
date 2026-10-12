# Workflow editor p7/ex2 — step 5: reading position during content changes

Commit: agdevworld `29bfcd8`.

## What changed

- **One anchoring mechanism for the pane.** `.run-side` now has `overflow-anchor: none`; the file `pre` has it too. `RunSide.update()` holds the reading position itself:
  - **Capture, before applying.** In the first visible section, the anchor is the first keyed item whose top is at or below the pane's top edge. If there is none, the section itself is used. The anchor's offset from the pane top is recorded.
  - **History.** Inside History, the anchor is the first visible entry, so a reader of older entries keeps their place. When the newest entry is the first visible one and the History heading is in view, the heading is held instead. Newer entries then appear beneath it rather than above the viewport.
  - **Top of the pane.** At `scrollTop` 0 nothing is held: the pane stays at the top and new entries show.
  - **Restore, after applying.** The scroll offset is adjusted by the anchor's displacement. If the anchor was removed or hidden, the previous offset is restored, clamped by the browser.
  - Step 3 had relied on Chrome's native anchoring. It is disabled now, so there is no double correction, and the behaviour no longer depends on the browser implementing `overflow-anchor`.
- **Explicit navigation is distinguished.** Selecting a node, opening, closing or switching a file pass `explicit` to `renderSide`, and those updates do not anchor. Background refreshes, the host cadence and the clock always anchor. A link to a question scrolls to it explicitly (step 4).
- **The open file.**
  - **Re-reading.** The file is read again on a full refresh when the run folder listing shows a different size or modification time for it. Files outside the folder, which are recorded artifacts with no listing, are re-read on every full refresh. A historical view is never re-read.
  - **Same file.** The `pre` element stays for as long as the file is selected. Unchanged text causes no write. New text is written with both offsets kept, clamped to the new content.
  - **Another file.** Opening another file resets both offsets.
  - **Clicking the open file again** keeps its text and offsets while it reloads.
  - **Read errors.** A failed re-read keeps the shown text and adds "Could not read it again: …". Stale and late responses are dropped by the step 2 request identity.

## Verification

New and extended sections of `checks/runInteraction.ts`, all passing:

| Scenario | Observed |
| --- | --- |
| Reading history near the bottom; two new files (Reports and files grows) and an instruction (the Instructions section appears) are added above | entry offset 118.0 → 118.0 px |
| History heading at the top of the view, then a progress entry | heading 20.0 → 20.0 px; the first entry is the new seq 61 |
| Pane at the top, then a progress entry | `scrollTop` stays 0 |
| Original prepend case (step 1) | 117.97 → 117.97 px, same element |
| Same file rewritten while scrolled to 500/120 | new text, offsets 500/120 |
| Same file shortened | top clamped to 496 (its maximum), left 120 |
| Another file opened | offsets 0/0 |
| First file's read delayed 1.5 s, second file opened meanwhile | the second file is shown; the late response is ignored |
| Re-read fails (injected 500) | text kept; "Could not read it again: injected read failure" |

Sections 1–10 and the lifecycle checks still pass.

Regression run against this build:

- `npm run check`: 211 tests passed, plus type check and build.
- `runs.ts`, `docsteps.ts`, `appearance.ts`, `review.ts`, `nodes.ts`, `conversation.ts`, `exec.ts`: all passed.
