# give_context_easier p1 — step 4 report: the context picker in the Front Room

## What changed (agdevworld `93c1a50`)

- **`src/contextPanel.ts`**: a DOM panel beside the composer, opened and
  closed by a `▤ contexts` button at the bar's left end (the composer now
  starts after it). Visibility is kept in `localStorage` (wrapped in
  try/catch; a private window just forgets it). It is the same panel in the
  Front Desk and the Arguing Room, because both are the one scene. What it
  inserts goes into that room's current draft only.
- **List**: catalog state and read age in a status line
  (`current` / `last-known` with the error / `unavailable` / `unconfigured`),
  a search box over id, name and description, each source's newest
  publication (short commit, date, subject) and its state (`last-known`,
  `empty`, `unavailable`), a ⟳ refresh, "show archived", and
  "register existing…". Clicking a row, or ↑/↓ and Enter in the search box,
  resolves `latest` at that moment and inserts `<id>@<40-hex commit>`.
- **Browse**: README and the file tree at a revision. A file click inserts
  `<id>@<commit>:<path>`, "view" previews text or an image, "insert
  <id>@<short>" inserts the revision itself. A detail view that is behind the
  newest publication says so and offers "view newest".
- **In your draft**: references already in the draft are listed. One whose
  source has a newer version gets "use newest", which replaces that
  reference and nothing else. The panel never rewrites a draft on its own.
- **`src/frontDeskInput.ts`**: `insert(text)` puts the text at the caret or
  selection remembered on blur/select/keyup/input, adds a space only where it
  would otherwise run into a word, keeps undo through `insertText`, fires the
  existing change path (counter, Send state, draft strip) and gives focus back
  with the caret after it. `replace(from, to)` is for the newer-version offer.
- **States**: loading, empty, no match, a failed read (the last list stays,
  under a red status), a failed resolve ("your draft is unchanged"). A
  refresh never rebuilds the search box, so typing or an open IME composition
  there is never cut off.
- **`src/contextState.ts`**: the relay client, plus an in-memory catalog for
  `&demo=1` (`&ctxfail=1` or `window.__ctxfail`, `&ctxstale=1`,
  `window.__ctxBump(id)` simulates somebody else publishing).
- On narrow screens the Send label shortens to `Send · run`, so the composer
  keeps room beside the toggle.

## Evidence (demo fixture, vite on :5178, headless Chrome over CDP)

Scenarios in `agdevworld/.local/ctx/` (`drive.mjs` + `s1`–`s4`, ignored);
screenshots in `agdevworld/.local/shots/ctx/`.

- `s1-insert` (all 23 assertions pass): the panel starts hidden and the
  toggle shows it with the search box focused. An IME composition plus commit
  in the search narrows to one row, and Enter inserts `world-lore@<40 hex>` at
  a caret placed mid-draft (`前半のテキスト。 world-lore@… 後半の文。`), with
  focus back in the composer. A second reference by click and a file
  reference from the browse view keep the whole draft. Esc goes from detail
  to list, then closes and refocuses the composer. The hidden state persists.
  **No human post was made** (selection sends nothing). With the relay failing,
  the error is shown, the list stays and the draft is unchanged. "use newest"
  moves one reference only after a publication elsewhere. The open state
  survives a reload. At 390 px wide the panel fits the screen.
- `s2-narrow`: at 390×800 the composer is 146 px wide (it had been squeezed
  below 70 px before the label change), and ↓ + Enter inserts.
- `s3-argue` (Arguing Room): the toggle is present. IME-committed text
  `日本語で` is kept, the reference lands at the caret, and IME typing
  continues after it.
- `npx tsc --noEmit` clean; `npm run build` passes.

## Not yet checked

- Against the live relay and catalog (needs the catalog, report1's missing
  setup); the reply-target strip beside the panel in a conversation with a
  real pending question. Both are part of step 6's live trial.
- The web image on :8090 is not rebuilt yet; it is rebuilt once with the
  relay deployment.
