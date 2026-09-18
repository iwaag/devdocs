# Argue p2 ex1 — report

Date: 2026-09-18 JST. Plan: [plan.md](plan.md). Steps: [1](report1.md) ·
[2](report2.md) · [3](report3.md) · [4](report4.md).

## Delivered

1. **A human can finish an argue from the room.** The completion engine
   knows an `ARGUE` (`#argue › argue-*`): its closure is the argue topic and
   nothing downstream — the setup plan, mission and channel it led to, the
   desk it came from and its memo all stay — listed in the preview as "not
   this request's — left alone". A human close writes no outcome: a `✔`
   without an outcome note means "ended by the human". The room reaches the
   shared door by anchor (`/argues/<anchor>/close-plan`, `/close`), so
   `finish ✔` and the existing panel work in `/?view=argue`; posting into a
   `✔`'d argue resumes it as before.
2. **The view row says what each button does.** `‹ interpretation n/N:
   active = current settings ›` and `view: dialogue ⇄` read; `re-voice with
   current settings ($) · <rev>` writes, confirms first (revision, how many
   posts, that earlier interpretations stay, one run), and is disabled while
   the renderer is busy or the relay cannot write. The renderer's state,
   failures and asked re-voicings are a line under the row, from the payload
   the room already polls.
3. **The composer is a visible textarea**: caret, selection, wrapping,
   growth to five rows, placeholder hints, a dashed read-only state with the
   reason. IME composition, the confirming Enter, Shift+Enter and paste are
   the element's own; its clicks and wheel never reach the canvas.
4. **Deployed** on agstudio (web image, relay); no listener change.

## Validation

| What | Evidence |
|---|---|
| Relay suite | agentroom 301 passed; `tsc` + `vite build` clean. |
| Browser, demo | `agdevworld/.local/shots/argue-p2-ex1/`: close → `✔` → resume; the row in both views, the confirm, the busy state, the cycler; the composer's selection, mid-edit, composition, narrow layout, disabled look. |
| Live, deployed | smoke argue 7259: close from the door (`applied`, nothing written), **Front's listener silent on the ✔**, resume by post served at once, closed again. $0.20. |
| API budget | 90 room reads → 0 lines in Zulip's server log; the state line and the close plan cost 0 calls each. |
| `nctl drift` | converged=46, 0 diffs. |
| **Human session on this code** | **Pending** — see [report4](report4.md). A human-led argue on p2's code did happen this morning (`argue-20260918-124701`, $2.83) and exhibited all three braindump items. |

## Remaining limitations

- Native wheel scrolling of a long draft inside the composer could not be
  confirmed by the CDP driver (events arrive unprevented; the box draws a
  scrollbar); a real mouse in the human session settles it.
- The disabled composer was screenshotted by flipping the demo's chat flag,
  not against a read-only relay.
- The confirm is a browser `confirm()`; a Phaser overlay would match the
  room better.
- Closing an argue leaves its `#memo` topic open (presentation only; a ✔
  there changes nothing). Decided, not forgotten.
- Front has no "let's stop here" recognition; a human's thanks still buys a
  run, and Front still cannot end an argue that reached no outcome except by
  a human's `finish ✔`.
- `argue-20260918-124701` is still open; it can be the argue the human
  closes from the room.

## Commits

| Repository | Commits |
|---|---|
| agdevworld | `d7fa2ad` (step 1), `5c4b0f5` (step 2), `69f07e9` (step 3) |
| pj-agdev | `0e45161` (submodule pin) |
| devdocs | reports 1–4 and this report |
