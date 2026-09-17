# Step 4 — the Arguing Room, and the Front Desk on the separated flow

Date: 2026-09-18 JST. Plan: [plan.md](plan.md) step 4. Verified in a browser
against the scripted demo adapter (no relay writes, no paid run); the checks
that need the live relay and a real rendering are step 5's.

## What was built

- **One scene, two rooms.** `FrontDeskScene` is parameterized by a
  `RoomAdapter` (`src/roomState.ts`): which conversations the room lists, how
  one is opened, where a post goes, how another interpretation is asked for,
  which settings room is the background, and whether it has a completion
  door. `deskAdapter` (`/?view=frontdesk`, `?conv=<id>`) and `argueAdapter`
  (`/?view=argue`, `?argue=<anchor>`) are the whole difference; portraits,
  playback, history, composer, IME input, settings loading and text layout
  are shared, not copied. `src/frontDeskState.ts` is deleted.
- **Navigation.** Each room links to the other and to the operation room;
  the dashboard header has an *Arguing Room* link beside *Front Desk*.
- **Conversation creation / selection / resumption.** The history panel's
  chips list the room's conversations (argues by stem, newest first). In the
  Arguing Room with nothing open — or after *new argue* — the first post in
  the bar opens an argue (`POST /argues`) and the scene moves to its anchor.
  A ✔'d argue is resumed by posting; the status line says the discussion is
  resumed and what it ended in stays as it is.
- **Original and dialogue views.** Every agent post is a reply to page
  through — Front's, and in an argue every specialist's. `view: dialogue ⇄
  original` decides its turns (`src/frontDeskPlayback.ts`): a saved
  interpretation's turns, or the post as written. A post with no usable
  rendering — pending, overdue, failed, a speaker with no character — falls
  back to the post as written with the reason in the status line, so reading
  and replying never wait for a rendering. A rendering that lands under the
  reader keeps their place. **The composer always posts into the source
  conversation**, whichever view is on.
- **Source references.** The 📎 chips in the dialogue box and in the history
  are clickable: they switch to the posts as written, move to the cited post
  and mark it `◀ cited` in the history, scrolled into view.
- **Interpretations.** `interpretation: …` shows which one is on show and how
  many are saved, steps through them and back to following the active
  settings; each is drawn with the manifest of its own revision
  (`FrontDeskSettings.resolve`), so older interpretations keep their pinned
  faces and background. `reinterpret ⟳` asks for one at the settings current
  now and says earlier ones stay. The button row marks a rendering in
  progress (`…`), an unavailable renderer (`⚠`) and failures (`✖`).
- **Multiple speakers.** A rendered turn names its character. A post as
  written is drawn with its speaker's own character when the settings have
  one — an agent by roster name, a logical speaker by its label in `senders`
  — and with the common icon under its own label otherwise, so `sage:arxiv`
  stays `sage:arxiv`. Front (the account, never a logical speaker on it)
  stands in the lower left; everybody else speaks from the upper-left box.
  The Front-only assumptions that were audited out: the `developer` /
  `agent` post kinds, the fixed `#front` caption, `Front received it`, the
  `conv` parameter, the unconditional finish button (hidden in a room with
  no completion door), and the `front` room background.
- **Settings manifest** (`agdevworld-settings` `1af317b`): `[characters.archsage]`
  (`agents = ["archsage"]`, nickname ターくん, `face.png`) and `[rooms.argue]`.
  Checked against the published roster read from the live relay: `front` →
  front, `autolab` → autolab, `agforge` → forge, `archsage` → archsage;
  `cagent` and `agobserver` publish no character and are shown as written.
  A sage is deliberately not mapped to the council's character. Synced on
  this host; the relay serves revision `1af317b66219` with both rooms.
- **The combined path is gone** on all three sides: agfront (step 2), the
  relay's `split_dialogue` (step 3), and here the `DeskDialogue` types, the
  reply-with-scene playback and the old demo. Old records are not migrated:
  an old reply shows its text as it is.

## Verification (headless Chrome over CDP, `.local/deskshot.mjs`, vite dev)

Screenshots (ignored): `agdevworld/.local/shots/argue-p2/`.

| Plan item | Observed |
|---|---|
| Human post | `a1`→`a2`: a blank Arguing Room, the argue background, "State what you want, to open a new argue…"; typing and Enter opens `#argue › argue-demo` and posts. |
| Pending fallback | `a3`: Front's plain reply on show at once with `answered 7s ago · being rendered…`, `interpretation: active demo (0 saved) …`. |
| Rendered dialogue | `b1`/`b2`: the same reply as character turns, a 📎 citation and a 🔗 chip, `dialogue at demo`. |
| Multi-speaker history | `b2`–`b4`, `b7`: Front, Autolab (two turns from one post, each cited) and `sage:arxiv` under the common icon "as written"; `▶ 2 new replies waiting` while reading. |
| Source lookup | `c2`: clicking the 📎 chip switched to `view: original`, opened the history on the cited post marked `as written ◀ cited`. |
| Failed rendering | `c3` and the desk's fourth reply: the original stays on show; in the dialogue view the status carries `rendering failed: …`. |
| Room switching | the nav links in `a1` (Front Desk ↗) and `c2` (Arguing Room ↗); each room loads its own background from the settings revision. |
| Reinterpretation | `b6`/`b7`: `reinterpret ⟳` → "asked Front to re-voice this at settings 1af317b66219 — earlier interpretations stay", then `2 saved` and the interpretation button stepping between them. |

`tsc` and `vite build` are clean. Reload/resume against real conversations,
reinterpretation after a real settings change, and the extreme-lore check on
a real substantive run are done in step 5, after deployment.

## Commits

| Repository | Commit |
|---|---|
| agdevworld | `82bac93` |
| agdevworld-settings | `1af317b` |
