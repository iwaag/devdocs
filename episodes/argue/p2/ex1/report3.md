# Step 3 — a visible composer

Date: 2026-09-18 JST. Plan: [plan.md](plan.md) step 3. Commit: agdevworld
`69f07e9`.

## Delivered (`src/frontDeskInput.ts`, `src/scenes/FrontDeskScene.ts`)

- **The textarea is the composer.** The same DOM element that gave the bar
  an IME is now visible and styled: 16 px room font, a blue caret, a real
  selection highlight, a focus ring, a placeholder, its own dark box with the
  bar's border. It sits on the bar's bottom edge and grows *upward* as the
  draft wraps, to five rows, then scrolls inside the box. The handle keeps
  `value / set / focus / place / setDisabled / composing` and adds
  `setPlaceholder`; `place()` is still called from the layout pass, so the
  narrow layout repositions it with the bar.
- **Hints moved.** The room's prompt words, "State what you want, to open a
  new argue…" and "chat is not available right now" are the placeholder;
  "⏎ sends · ⇧⏎ newline" sits beside the counter (dropped when narrow). The
  Phaser `promptText` draft mirror is gone. The send counter and the
  `SendPhase` status line stay in Phaser.
- **Disabled state**: when the relay is not live, has no chat credential or a
  post is in flight, the textarea is `disabled` with a dashed border, dim
  text and an amber placeholder saying why; the draft stays visible.
- **Pointer routing**: nothing changed in the scene's handlers. The textarea
  is a sibling of the canvas, so its clicks, drags and wheel never reach
  Phaser's `pointerdown` / `wheel` listeners (Phaser attaches to the canvas,
  `preventDefaultWheel` off).
- The IME guard is unchanged: Enter with `isComposing` or keyCode 229 never
  sends; Shift+Enter is a newline.
- The Front Desk gets the same composer (`s3-j-desk`), as intended.

## Validation (headless Chrome over CDP, `.local/roomshot.mjs`, vite dev)

Screenshots: `agdevworld/.local/shots/argue-p2-ex1/s3-*`.

| Plan item | Observed |
|---|---|
| Composition and candidates in place | `s3-e`: `にほんご` shown underlined inside the draft at the caret, mid-text. (CDP `imeSetComposition`; the candidate window is the OS's and is not drawn by headless Chrome.) |
| Confirming Enter does not send | `.local/ime-check.js`: a keydown Enter with `isComposing`, then one with keyCode 229 — the value is unchanged after both, nothing sent. A driver-dispatched plain Enter during a composition committed the text and, in headless, inserted the `\r` the driver attaches — a driver artifact, not a send. |
| Shift+Enter breaks a line | `s3-b`: "二行目。Shift+Enter made this line." on its own line; the box grew to two rows (44 → 66 px). |
| Long draft wraps, selects, edits in the middle | `s3-b`, `s3-c` (selection of "a visible textarea: caret and"), `s3-d` ("[inserted] " typed at offset 45). Eight sentences: five rows, then a scrollbar inside the box (`s3-i`). |
| Paste | the element's own; nothing intercepts `paste`. Not driven. |
| Disabled when the relay is not live | `s3-i`: with `chat.configured=false` flipped in the demo, `disabled=true`, placeholder "chat is not available right now", the health line "this relay was built without a chat credential", the paid button "re-voice: the relay cannot write right now". A second dev server on the read-only :8095 relay did not come up (port taken), so the real path was not screenshotted; the flag is the same `usable` the real relay drives. |
| A click in the textarea does not advance | on reply 3/5 (multi-turn): a click in the dialogue box moved turn 0 → 1; a click in the composer left the cursor at turn 1. |
| Wheel over the composer | the `wheel` event reaches the textarea with `defaultPrevented=false` and the history/cursor do not move. **The CDP-synthesized wheel and scroll gesture did not change `scrollTop` in headless Chrome**, so native scrolling of the draft is not confirmed by the driver; the box does draw a scrollbar. To confirm with a real mouse in step 4's session. |
| Narrow | `s3-h` (640 px): the composer takes the bar's width, the key note is dropped. |
| Build | `tsc` + `vite build` clean. |
