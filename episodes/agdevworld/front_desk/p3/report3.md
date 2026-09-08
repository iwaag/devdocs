# Step 3 — the Front Desk completion flow

agdevworld `2d84df2`; pj-agdev pointer moved. Verified in the browser
against demo data; no realm or Plane write was made in this step.

## What exists now

- **A `finish ✔` button** in the Front Desk's button row, between
  `settings ⟳` and `new conversation`. It opens a panel and nothing else:
  only the panel's own button ever writes.
- **`src/frontDeskClosePanel.ts`** — the panel, a Phaser container like the
  rest of the scene. It draws the relay's words and decides nothing: one row
  per action with a glyph and colour for its state (`▶ ready`, `✔ done`,
  `⨯ blocked`, `· kept`), the reason underneath, and the kind spelled out
  (`Work`, `topic`, `channel`, `this conversation`). Below the rows:
  **what is not this conversation's** (the exclusions, with the topic they
  are anchored to instead), **what is not known** (unread topics, windowed
  histories, Plane refusals, a truncated walk), and the relay's own sentence
  *"this closes work; it does not stop a running agent"*.
- **The final click approves the concrete plan.** The button reads
  `close N targets`; the request carries the fingerprint of the plan on
  screen and nothing else. A `409` refusal is drawn as a **preview again**,
  with the reason and the refreshed targets — never as a result, because
  nothing moved.
- **Progress and per-target results.** During the call the summary says
  `closing…`; afterwards each row wears its outcome (`✔ applied`,
  `· already`, `⨯ failed`, `— skipped`) and the relay's note for it. The
  summary distinguishes the two endings in words: *"closed — 3 changed, 2
  already were"* against *"partially closed — 3 changed, 1 failed, 3 left;
  the conversation stays open"*. Failed and blocked rows stay on screen with
  `refresh` and `try the rest again` beside them.
- **Refreshing after a change**: the panel calls back into the scene, which
  re-reads the conversation and the board, so the caption picks up `✔
  resolved` and the history chips show the ✔ — the readable history is
  untouched, since resolving renames a topic and deletes nothing.
- **Reopening does not reopen the work.** `POST /frontdesk/<id>/post`
  already un-resolves a ✔'d conversation so the post lands in it; the reply
  now carries `resumed`, and the status bar says
  *"↩ reopened this conversation — the work it closed stays closed"*. The
  resting line for a resolved conversation changed to *"posting here reopens
  the conversation, not the work it closed"*. A resolved child topic and an
  archived channel are not touched by that post, and the screen no longer
  lets a reader assume otherwise.
- **The demo source plays the whole flow** (`?view=frontdesk&demo=1`): a
  plan with one of each state, one target that fails the first time and
  succeeds on the retry, a blocked Work that keeps the conversation open,
  a kept channel and an excluded topic. No relay needed.
- **Draft, IME, switching and layout.** The panel never touches the hidden
  textarea, so a draft survives opening, closing and Escape (measured:
  `テスト` before, during and after). Escape closes the panel before the
  history panel; the wheel scrolls whichever is under the pointer; a click
  inside the panel no longer advances the reply behind it. Switching
  conversation **drops the plan** — a plan belongs to one conversation — and
  every request is discarded if the id changed while it was in flight. The
  panel takes the right side on desktop and the whole frame under 720px.

## Verification

`npm run build` and `tsc --noEmit` clean. Browser checks over CDP
(`.local/deskshot.mjs`, shots in `agdevworld/.local/shots/p3/`):

| shot | what it shows |
|---|---|
| `p3-b-preview` | preview: 5 to change · 1 already · 1 blocked · 1 kept, the exclusion and the note |
| `p3-c-partial` | after the click: 3 changed, 1 failed, the conversation **kept open** with its reason |
| `p3-d-retry` | `try the rest again`: the failed topic applied, nothing repeated, still open for the blocked Work |
| `p3-e-narrow` | 520×900: the panel takes the frame, every row wraps |
| `p3-f-draft` | the draft `テスト` before, during and after the panel |

## Left for step 4

Nothing has been executed against the realm or Plane yet. The running relay
still serves the pre-p3 code and has no `AGENTROOM_PLANE_ENV`; the deployed
web image on `:8090` is p2's.
