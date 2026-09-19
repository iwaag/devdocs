# Project Room p1 — Step 3 report: the Project Room

Plan: [plan.md](plan.md), step 3. Code: `pj-agdev/agdevworld` —
`src/projectRoom.ts` (the screen), `src/projectState.ts` (relay reads and
writes, read positions, drafts), `src/projectDemo.ts` (the `&demo=1`
fixture), `src/scenes/ProjectRoomScene.ts` (background, tint, portrait);
`main.ts` routes `?view=project`, the Front Desk / Arguing Room nav and the
operation dashboard link to it.

## Delivered

`http://localhost:8090/?view=project` (or `:5173` under vite). A dedicated
scene: the settings' room background scaled to cover, a shared tint over it
(0x0d0f14 at 25 %, the starting value; step 4 tunes it), and the portrait of
whoever answers in the selected conversation — Autolab's character for a
plan or a run, Front's for a document, the common icon when the settings
have no face. The board is four DOM panels over the scene:

- **left** — every project and study: kind chip (`project` / `study` /
  `kind unknown`), the most urgent reply state, mission and task counts
  (`tasks 1/1?` when a work channel is not mirrored), latest activity, the
  first gap; archived channels under their own heading.
- **centre** — the selected project: kind, origin argue (a link into the
  Arguing Room by anchor), description, gaps (`setup only…`, `a research
  plan alone is not executable work…`, `no goal or research plan…`,
  incomplete task counts), then PURPOSE (documents), SETUP, PLANS & RUNS
  (missions with `work <state>` and `reply <state>` chips, `replaces` /
  `replaced by`, task counts, their tasks indented with state and reply),
  PLANS WITHOUT A RECORD, TASKS WHOSE MISSION IS MISSING. Finished missions
  fold away while something is open; `show ✔ history` shows them.
- **right** — the selected document or plan as written, facts (recorded
  state with its caveat, reply with evidence, counts, origins), links found
  in the source filed as 📦 repository / 📄 report / 💬 zulip; a task shows
  its description and its latest report post; a document shows the path to
  Front (Front Desk, the origin argue) and says no mission belongs to it.
- **bottom** — the conversation as written (human / agent / ack, portraits
  from the settings), the destination label with the responsible instance,
  the status judged against that instance, and the composer: the rooms' IME
  textarea posting into the source conversation as the Developer. A ✔'d
  target is refused once and offers "resume and post"; a failed or uncertain
  send keeps the draft and says to read the history before retrying; a
  document has no composer.

URLs: `?view=project&project=<key>&plan=<anchor>` / `&run=<anchor>` /
`&topic=<name>` are written on selection and restored on load. Drafts are
per conversation in `localStorage` and survive a selection change, a refresh
and a reload; a read position per conversation gives the pink dot on rows
and projects with a post newer than the reader saw. Refresh is 4 s for the
selected conversation and 15 s for the board and project; panels re-render
only on a changed signature and keep their scroll positions.

## Checks (browser, `deskshot.mjs` over CDP, demo source, vite `:5173`)

Screenshots in `agdevworld/.local/shots/project-p1/` (ignored):

- `02–04`: project → plan → run navigation; URL `…&project=93&run=6773`.
- `05–06`: a run comment on a ✔'d task: refused with the resume button,
  resumed and posted, ack and reply arrived on the 4 s poll, unread dot moved.
- `07`: a document: composer disabled ("a document is not posted into"),
  Front Desk / argue links, source text on the right.
- `08`: an uncertain send (`fail once`): draft kept, "check the history
  before sending again"; switching to the goal and back restores the draft.
- `09`: IME composition (`にほんご`) — the composition's Enter did not send.
- `10–11`: 600 px viewport: single column, panels stacked, page scrolls,
  composer follows its slot.
- `12–13`: direct reload of `…&project=152&plan=6400` restores the selection;
  `show ✔ history` reveals the replaced mission, its replacement, the
  resolved-unfinished one and the old-style plan.
- No page errors in any run (`window.__errors` armed each time).
- `npm run build` passes (tsc + vite).

Not done live: the room was driven on the demo fixture; the relay on `:8094`
still runs the pre-p1 code until step 5 deploys it, and the `project`
background falls back to the Front Desk's until step 4 registers it.
