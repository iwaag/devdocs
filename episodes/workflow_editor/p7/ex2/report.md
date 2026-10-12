# Workflow editor p7/ex2 — final report: input and reading state in the run view

## Outcome

The braindump reported that, in the run view's right pane, text conversion was reset and scroll positions jumped back while the editor updated itself. The investigation confirmed it, and this episode fixed it in the source:

- Background updates no longer replace the controls being typed into.
- Updates no longer reset the file view's or the pane's scroll position.
- They no longer interrupt an IME composition.

Execution status, progress, presentations and connection state stay live while the person types and reads.

| | Status |
| --- | --- |
| Implementation | complete: agdevworld `861afe8` … `e16f2c4`, pushed. pj-agdev gitlink updated |
| Automated validation | complete: `checks/runInteraction.ts` 38/38, `npm run check` (211 tests), and the 7 existing run-view browser checks, all passing |
| Native macOS Japanese IME | **not validated** (see below) |
| Deployment | **not done**: the installed release is still `6058c0f`, the investigated revision |

## Cause and fix

The investigation found three things wrong:

- Every run event, and every 3 s host read that differed only in the executor's heartbeat time, rebuilt the whole right pane with `side.replaceChildren()`.
- Drafts were copied into new controls. That kept the characters but lost the original element, so the composition, selection and internal scroll went with it.
- Polling and SSE reloads were not coordinated with each other.

The fix, step by step (reports 1–7):

1. **Reproduction first.** `checks/runInteraction.ts` reproduced every failure on `6058c0f`:
   - the control was detached mid-composition;
   - heartbeat-only rebuilds happened;
   - the file view's offsets went from 400/80 to 0/0;
   - a history entry moved from 118 px to 330 px.
2. **One refresh coordinator per view** (`src/runRefresh.ts`).
   - Merged full and host requests, one batch at a time, per-resource outcomes.
   - Operations await their own refresh.
   - Disposal aborts reads and ignores completions.
   - File reads carry their own identity.
3. **A persistent right pane** (`src/runSide.ts`).
   - Display is patched in place (`src/patch.ts`, with keyed items).
   - Controls are created once and never rebuilt.
   - Actions are delegated and read state at click time.
   - The duplicate answer form in Selected node became a link to the one form.
4. **Input ownership.**
   - No background value writes.
   - A submission clears only the exact draft it sent.
   - Typing during a request is kept; double submission is blocked.
   - A composing control turns read-only only after `compositionend`, with its text and the reason kept.
   - Adjustment previews are bound to their text, reason, revision and sequence.
5. **Reading position.**
   - The pane holds its own anchor (`overflow-anchor: none`), with distinct rules for the top of the history and the top of the pane.
   - The open file is re-read on change and keeps its offsets, clamped.
   - The person's own navigation is not anchored.
6. **Scoped updates.**
   - A heartbeat-only change renders nothing.
   - Other host changes touch only the four sections that show them.
   - Connection changes touch only the pill and banners.
7. **The full validation matrix and documentation.** The matrix found one more defect, a form present in the historical view, which was fixed. The README now describes the update lifecycle and the check.

## Native macOS Japanese IME: not validated

All composition results are synthetic: CDP `Input.imeSetComposition` and `Input.insertText` in headless Chromium. They show that the element, its focus and the composition events survive every update, but not how the native candidate window behaves.

A manual fixture was provided on 2026-10-12: a private service on port 8203, with progress every 3 s and a heartbeat every 2 s. The Developer was asked to compose in the answer and definition textareas and then submit. When the final report was requested, the fixture's run held 2191 records and **no answer from the browser and no adjustment preview**. So the manual procedure was not carried out, and no native result is claimed. The fixture was stopped and its temporary data removed.

To validate later, do either of these, then follow the procedure in report7.md for both textareas:

- set up a similar fixture: a private service built like `checks/runInteraction.ts`'s, with progress records and executor heartbeats written periodically. The script used here was a scratch file and is not in the repository;
- use the editor after deployment.

## Not done here

- **Deployment.** From pj-agdev: `agdevworld/scripts/wfe-release.sh deploy .local/agdev`. A local build does not change the operating service.
- **Limitations** (report7):
  - Anchoring falls back to the previous offset when the anchored item disappears.
  - Artifacts outside the run folder are re-read on every full refresh.
  - The 30 s clock still builds fresh section trees, although it writes only the time texts.
- **Environment note.** The browser checks need `chromium_headless_shell-1234` for the pinned `playwright-core`. Some other tool had replaced the shared cache with 1248; the matching revision was installed again (report1).
