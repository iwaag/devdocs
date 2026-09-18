# Argue p2 ex1 — a room a human can actually finish, read and type in

## Goal and approach

Source: the first human look at the Arguing Room after [p2](../report.md).
Three things were unclear to the person who opened it:

1. There is no way to close an argue from the room, and Front did not close
   it either.
2. `reinterpret ⟳` is not distinguishable from a view toggle; it is a paid
   write.
3. The prompt bar is a hidden textarea: no visible caret, selection or
   multi-line editing.

This phase fixes those three before the pending human-led session is run.
It is a breaking change in a private experimental environment: no backward
compatibility, no migration of old records, no new security work. Choose
names, layout and file boundaries freely; the constraints below are the few
that protect other components. Keep secrets and machine-specific facts in
ignored files.

## Step 1 — Let a human close an argue from the room

Today an argue resolves only when Front's reply ends in an `ag-argue` block
(`outcome: project|study|plan`, `complete: true`) **and** the listener
verifies the named channel, document and autolab answer exist
(`agfront/agent/guides/argue/guide.md` § Finishing, `agfront/src/agfront/argue.py`
`verify_outcome`). An argue that does not land on a project, study or plan
stays open forever, and the room hides `finish ✔` because the argue adapter
has no `completion` (`agdevworld/src/scenes/FrontDeskScene.ts` around
line 262, `src/roomState.ts` `argueAdapter`).

- Give the argue adapter a `completion` like the desk's: `closePlan` /
  `close` over the relay's `/complete/plan` and `/complete` with
  `{channel: 'argue', topic: 'argue-<stem>'}`. The room key is the anchor
  id, so the adapter needs the argue's live topic for the query; the detail
  payload already carries `topic` and `live_topic`. Then the existing
  `FrontDeskClosePanel` (preview → click → outcomes) works unchanged.
- Teach the completion engine what an argue is. `agentroom/src/agentroom/closing.py`
  `classify()` returns the generic `TOPIC` for `#argue › argue-*`; add an
  `ARGUE` kind (constants come from `agag.argue`: `ARGUE_CHANNEL`,
  `ARGUE_TOPIC_PREFIX`) and decide its **boundary** in `Scope` /
  `_ownership`. Recommended boundary: closing an argue resolves the argue
  topic and nothing downstream — a project or study it opened stays open
  (this matches the p2 words "posting here resumes the discussion, not
  what it ended in"). The origin desk conversation, if any, is somebody
  else's request and stays out of the closure.
- A human close is not an outcome. Do not fabricate an `[selfnote][outcome]`
  or an `ag-argue` block; a ✔ without an outcome note simply means "ended
  by the human". Front's own outcome path stays as it is. Optionally record
  the human's reason as a plain post before the ✔; do not make it required.
- Posting into a ✔'d argue already resumes it (relay + `resumed` words);
  keep that. Check that Front's listener does not re-serve the resolved
  argue on the ✔ itself (the memo rule and `pyagag` 0a33830 cover the
  Notification Bot lines; confirm for `#argue`).
- Optional, only if cheap: let Front recognise a human's explicit "let's
  stop here" and reply without proposing an outcome. Do not add a new
  discussion protocol; p2's report asked that facilitation changes wait
  for real sessions.

Verify: `agentroom/tests/test_closing.py` / `test_close_plan.py` style
fixtures for an argue root (plan lists the argue topic READY, downstream
project channel not touched, resolved argue reports `already ✔`); in the
browser, `finish ✔` appears in `/?view=argue`, the preview names only the
argue, the click resolves it, the row shows ✔, and a further post resumes it.

## Step 2 — Make the three view buttons say what they do

The row is `reinterpret ⟳ · interpretation: … (N saved) · view: dialogue ⇄`
(`FrontDeskScene.ts` `renderViewButtons`, `toggleView`,
`cycleInterpretation`, `reinterpret`). Only the first writes: it POSTs
`/argues/<anchor>/render` and buys one `present` run (Sonnet 5, about
$0.05 for a five-post argue in p2). Its only feedback is a 12-second note.

- Separate reading from writing visually: the two read-only controls
  together, the paid one apart and styled as an action (colour, a label
  such as `re-voice with current settings ($)`, or a small confirm step).
  A confirm is acceptable here; a browser `confirm()` is fine for an
  experimental UI if a Phaser overlay is too much work.
- Say what will happen before the click and what happened after: the
  settings revision it will use (`this.settings.activeRevision`), that
  earlier interpretations stay, and afterwards the renderer state the relay
  already reports (`presentation.renderer.state`: `rendering`, `unavailable`,
  and `failed[]`). Disable the button while `renderer.state === 'rendering'`
  or `this.asking`; a second click during a render is the most likely
  accidental double spend.
- Make the toggle read as a toggle: `view: dialogue ⇄` is fine, but the
  interpretation cycler should show that it cycles (e.g. `‹ interpretation
  2/3 ›`, or two arrows) and that `active` means "follow current settings".
- Keep the URL entry points (`&mode=original`) and `showSource` (a rendered
  line jumps to its source post) working.

Verify against the demo adapter (`src/roomDemo.ts`) and the real relay:
screenshots of the row in both views, one deliberate reinterpret with its
confirm and its progress, and the p2 API-budget check (`90 reads → 0 Zulip
calls`) still holding — the new state display must come from the existing
detail payload, not from extra relay calls.

## Step 3 — Replace the hidden textarea with a visible composer

`src/frontDeskInput.ts` overlays a 2 %-opacity textarea with transparent
text and caret under a Phaser-drawn bar; `renderPrompt` in the scene draws
the tail of the draft on one line. Selection, caret, wrapping and
multi-line review are therefore invisible. The reason for the DOM element
(IME composition, candidate windows, paste, `isComposing` / keyCode 229
guard, Shift+Enter newline) is sound and must be kept.

- Make the composer a real, visible DOM textarea placed over the bar area:
  its own font, caret, selection highlight, auto-growing height up to a few
  rows, and a visible disabled state. Keep `createFrontDeskInput`'s handle
  (`value / set / focus / place / setDisabled / composing`) or replace it
  together with its callers; the scene's `place()` call in the layout pass
  gives you the rect.
- Move the hint texts (`adapter.words.prompt`, "State what you want, to
  open a new argue…", "chat is not available right now", the Enter /
  Shift+Enter note) to the textarea's `placeholder` / a caption, and drop
  the Phaser `promptText` draft mirror. The send counter and the
  `SendPhase` status line can stay in Phaser or move to DOM; pick one.
- Pointer routing: the scene treats "no Phaser object under the pointer"
  as "advance the dialogue" (`pointerdown` handler near line 290). A DOM
  element over the canvas receives its own events, so clicking or
  dragging in the textarea must not page the dialogue; check the wheel
  handler too, so scrolling a long draft does not scroll the history.
- The narrow layout (`this.narrow`) already resizes the bar; reposition the
  textarea in the same pass. `src/chatPanel.ts` has a plain
  `<textarea id="chat-input">` with the same Zulip-posting semantics and
  can be a style reference.
- Old Front Desk callers share the scene, so the desk gets the same
  composer; that is intended.

Verify in the browser with Japanese IME: composition and candidate window
show in place, the confirming Enter does not send, Shift+Enter breaks a
line, a long draft wraps and can be selected and edited in the middle,
paste works, the disabled state shows when the relay is not live, and a
click in the textarea does not advance the dialogue. Screenshots into
`agdevworld/.local/shots/argue-p2-ex1/`.

## Step 4 — Deploy, run the human session, report

- Check the deployed state through Nautobot / `pj-clusterintent/nctl` and
  the ignored environment notes; rebuild the web image and the relay
  (agentroom) on agstudio; no listener change is expected unless Step 1's
  optional Front behaviour was done.
- Run the changed suites (`agentroom`, `agdevworld` `tsc` + `vite build`,
  `agfront` if touched) and read the summary lines, not the tail.
- Run the human-led session that p2 left pending: open `/?view=argue`, start
  an argue, read an agent's reply in both views, reinterpret once, reply
  from the new composer, close the argue from the room, resume it with a
  post. Record cost from the run JSONs (`.local/agent/<role>/run-NNNN.json`)
  and note the observations p2 asked for (Front's pace, courtesy mentions,
  reply prefaces).
- Write `report.md` with delivered behaviour, validation, the human
  session's observations, and remaining limitations. Commit and push
  `agdevworld`, `pj-agdev` submodule pins, and `devdocs`.

## Useful implementation findings

- `agentroom` completion engine: `close.py` `Closer` applies a plan whose
  fingerprint the human saw; `closing.py` decides scope from link notes
  (`[served]`, `[rootchat]`) and never from prose. An argue's memo is linked
  by `[selfnote][memosource]`, deliberately not `rootchat`, so the memo is
  not in the closure graph — resolving an argue leaves its `#memo` topic
  open; decide whether to ✔ it in the same action (cheap, the memo is
  presentation only) or leave it.
- `agag.argue.parse_argue` identifies an argue's anchor note; the relay's
  `argueroom.py` resolves anchor → current topic (`Located`). Reuse that
  for the completion query rather than trusting the stem in the URL.
- Every relay-backed view re-reads on the `agdevworld:completed` DOM event
  (`completionState.announceCompleted`); fire it after a room close so the
  argue list shows the ✔.
- `present` rendering cost scales with agent speech; a reinterpret of a
  long argue is proportionally more. The relay derives `pending` and says
  `unavailable` after 15 minutes; it cannot tell slow from stopped.
- Two Omni Agent sessions can run on the Developer account at once and
  neither Front nor the room can tell; check `git log` before editing a
  file that says "changed on disk".
- The Bash tool is zsh here: quote `====`-like arguments, `grep` is
  ugrep. `pytest | tail && git commit` commits on red.
