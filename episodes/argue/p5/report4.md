# argue p5 — Step 4: the reading experience

## Result

developer/pj-aghangout `b3e0e2c` (from `53386ff`; not deployed yet):

- `src/meeting.ts`: short lines in the transcript and the player;
- `src/api.ts`: `dialogue` and `retryDialogue`;
- `src/local.ts`: the `dialogue` preference;
- `src/styles.css`;
- `fixture/server.ts`: dialogue routes and `fx-dialogue`;
- `checks/browser.ts`;
- README and guide facts.

## What the reader gets

- **Opening.** The conversation opens as 会話（短い表示）. A one-line note
  says the lines are the characters' short rephrasing, an interpretation
  and not the original; that selecting a bubble shows the original; and
  that replies and citations always go to the original.
- **Agent messages.** Each one keeps its header (number, name, agent,
  kind, reply target, time) and its footer (返信する, run link, workflow
  cards). Between them:
  - the portrait from the character export, or initials for a character
    without one (ponderer) and for an unmapped participant (argue,
    aghangout, …);
  - the bubble, in the speaker's accent.
- **Bubbles are buttons.** Click, Enter or Space opens the original in
  place; the button turns into 短い表示に戻す, with `aria-expanded` and
  `aria-controls`. The message stays where it was on screen and focus
  stays on the button. An original opened this way stays open while other
  lines arrive.
- **Citations.** `[#n]` in the overview opens the cited message's
  original, since a citation names original words.
- **Person messages** are always shown as written.
- **A line not ready.** A line being made shows 短い表示を準備中… いまは原文を
  表示しています above the original. A failed line shows
  短い表示を作れませんでした with 短い表示を再試行 (WFE's retry), and the
  original stays. If WFE reports the dialogue unavailable, the note says
  why in red and every original is shown.
- **表示 switch.** 短い表示（日本語） / 短い表示（English） / 原文, kept in this
  browser. 原文 reads no lines at all.
- **Reading cost.**
  - The page reads lines when a new message arrives, when the language
    changes, or every 4 s while some line is being made; never on every
    3 s poll.
  - A read only queues; WFE's worker calls the model. Component renders
    never call anything.
- **The player** (再生表示) consumes the same records: it plays the short
  line, and a 原文を表示 chip shows the original. It makes no second
  interpretation.
- **Kept as before.** Drafts, the outbox, reply targets (`#n への返信`
  names the original id), unread marks, workflow cards and panels.

## Verification

- **Unit checks.** `npm run check`: typecheck, build and 28 unit tests.
- **Browser checks.** `npm run browser-check` passes in Chromium and in
  WebKit (212 checks each), with WFE's embedded graph renderer. The new
  checks:
  - **short lines**: the opening view and its note; pending leaves the
    original readable; the line arrives without a reload; the portrait
    from the export; initials for a character without a portrait; an
    unmapped participant plain under its own name; person words as
    written; a failed line says so and keeps the original;
  - **in place**: the original opens in place; the message stays put
    (within 2 px); `aria-expanded` and focus; Enter and Space toggle;
    retry renders a failed line; an opened original stays open while
    others update; reply names the original id; the draft is kept;
  - **citations, polling, languages**: a citation opens the original; no
    line reads in 7 s once all are in; English lines; 原文 shows
    everything as written; the preference survives a reload; 原文 reads
    no lines;
  - **new speech, player, unavailability**: a new agent message arrives as
    written, then as a line; the player plays the line and shows the
    original on request; without a renderer, the originals and the
    reason;
  - **narrow (390 px phone)**: no sideways scrolling; bubbles at least
    40 px tall; a tap opens a long original without sideways scrolling.
- **One check failed at first, in the check itself.** "the message stays
  where it was" measured the position before Playwright scrolled the
  bubble into view for the click. The check now scrolls first and then
  measures.
- **One layout defect, fixed.** The note's margin overrode the reading
  column's auto margins, so on a wide screen it started at the left edge.

Screenshots (ignored): `pj-aghangout/.local/shots/{chromium,webkit}-{desktop,narrow}-dialogue*.png`,
`pj-agdev/.local/argue-p5/step4-desktop.png`.
