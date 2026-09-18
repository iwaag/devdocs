# Step 2 — the three view buttons say what they do

Date: 2026-09-18 JST. Plan: [plan.md](plan.md) step 2. Commit: agdevworld
`5c4b0f5`.

## Delivered (`src/scenes/FrontDeskScene.ts`, `src/roomDemo.ts`)

- **Reading and writing apart.** The two read-only controls sit together on
  the right: `‹ interpretation 1/2: active = current settings ›` (it cycles,
  says where it is, and says what `active` means; a pinned one reads
  `rev <sha>`; with nothing saved, `interpretation: active = current settings
  · none saved yet`) and `view: dialogue ⇄` / `view: original ⇄` as before.
  The paid one stands 36 px apart on the left, amber on dark, labelled
  `re-voice with current settings ($) · <revision>`.
- **Before the click**: a `confirm()` naming the settings revision, how many
  agent posts will be re-voiced ("the cost follows their length"), how many
  interpretations are saved and that all stay, and that this buys one
  presentation run and posts nothing into the discussion. A browser
  `confirm()` is what the plan allowed for an experimental UI.
- **While it cannot be asked**: the button is dimmed and non-interactive —
  renderer `rendering`, a request in flight, a post being sent, no active
  settings revision, no agent speech yet, or a relay that cannot write —
  and, except for the busy renderer (which the line below already says),
  the reason is written after it.
- **After**: a line under the row from the payload the room already polls
  every 4 s — `renderer: idle` / `rendering N posts…` / `⚠ renderer
  unavailable — <reason>`, `✖ N renderings failed`, `N re-voicings asked` —
  plus the existing 12-second note.
- The URL entry `&mode=original` and the 📎 source jump are untouched.
- The demo adapter now reports `rendering` for a re-voicing it was asked
  for and has not delivered, so the busy state can be looked at.

## Validation

| What | Evidence (`agdevworld/.local/shots/argue-p2-ex1/`) |
|---|---|
| Row, dialogue and original | `s2-b`, `s2-c`: the same row in both views; the cycler dims in the original view. |
| Before any agent speech | `s2-a`: the paid button dimmed with `re-voice: no agent has spoken yet`. |
| Confirm | the driver logged the dialog text (revision `1af317b66219`, 1 agent post, 1 saved); a dismissed confirm asks nothing (`DIALOG=dismiss` run: no request, row unchanged). |
| Busy | `s2-d`: right after accepting, the button is dimmed and `input.enabled=false` while the line reads `renderer: rendering 1 post… · 1 re-voicing asked`. |
| After | `s2-e`, `s2-f`: `1/3` then `2/3: rev demo` when cycled; the note "asked Front to re-voice this at settings … — earlier interpretations stay". |
| Narrow | `s2-g` (640 px): the row wraps under the buttons and stays readable. |
| Live argue | `s2-h`, `s2-i`: `argue-20260918-124701` against the deployed relay (:8094, p2 code): `renderer: idle · ✖ 1 rendering failed · 2 re-voicings asked` — state the old row showed only as a `✖` glyph. |
| API budget | 90 reads (`/argues`, `/argues/7217`, `/frontdesk` × 30) against the :8095 relay added **0 lines** to Zulip's server log (14095 before and after; the log was current, heartbeats visible). The state line costs no call. |
| Build | `tsc` + `vite build` clean. |

## Observations

- The live argue already carries two re-voicing requests and one failed
  rendering, none of which the earlier row made legible. Whether the failure
  is the one p2 called *unavailable after 15 minutes* is for step 4's look at
  the renderer log.
- Front's own words in that argue (reply 7257, in the dialogue view): it had
  resolved the argue at 7241, the human's next post un-resolved it, and Front
  then said everything was settled without resolving again — the shape step 1
  now lets the human finish by hand.
