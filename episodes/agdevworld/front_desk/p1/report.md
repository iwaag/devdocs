# front_desk p1 — a graphic-novel Front Desk, and one ghtrends run through it

The braindump asked for a Phaser scene, not an overlay, where the developer
talks to agfront as a character — background, portrait lower-left, dialogue,
prompt bar, a history that can be hidden — in a gyaru voice to the developer
and ordinary language between agents, on a profile and guide of its own; then
a little small talk and one `ghtrends` run. All of it exists and the run
completed: `microsoft/markitdown`, commit `a99625f` in the ghtrends repo,
reported back to the screen in character.

Steps: `report1.md` (scene and input), `report2.md` (`character_talk` and
event-driven turns), `report3.md` (relay and history), `report4.md`
(verification, deployment, the live conversation).

## What exists now

- **`/?view=frontdesk`** in agdevworld (linked from the dashboard): its own
  Phaser game, `FrontDeskScene`, with the supplied `bg.png` and
  `agfront.jpg` unchanged. Latest reply as dialogue with pagination and link
  chips; a prompt bar backed by a hidden textarea so Japanese IME and paste
  work and a composition-confirming Enter never sends; a scrollable history
  panel with acks as receipt lines and a draft that survives browsing it;
  send status, `unknown` in amber when the relay cannot be read, last known
  history kept.
- **`character_talk`** in agfront: its own profile and role (front's grant),
  its own guide defining the two voices and the event-driven turn, chosen
  from the home topic (`front-desk-…`) for direct posts and callbacks alike.
- **Three relay routes** (`/frontdesk`, `/frontdesk/<id>`,
  `POST /frontdesk/<id>/post`): Zulip is the history, the event queue carries
  updates, an unheld resolved conversation is read from the realm on request,
  submit tokens make a double click one run, a ✔'d conversation resumes in
  place.

## What the live run taught

- **Front exits after delegation and comes back on a mention** — three
  times in one conversation (a failure callback, a plan callback, a
  completion callback), each answered at home in the desk topic.
- **A conversation with an agent is a good failure detector.** The first
  delegation hit a TypeError in agautolab's `run_role` wrapper that had been
  silently failing every listener-started mission on this Mac since the day
  before. The screen showed it in Front's words within a minute.
- Two guide corrections came from reading actual posts: the whole output is
  the dialogue (a reasoning preface leaked once), and a routine's standing
  request topic is not a log (Front filed a record there).
- Phaser 4's word wrap halves emoji and Japanese; a geometry mask on a
  container child does not clip. Both are handled in the scene.

## References

- Conversation: `#front` › `front-desk-20260908-1600` (ids 5171–5207).
- Work: `#pj-ghtrends` › `workplan-trend6`, `#work-g-13` › `✔ workrun-task1-g-13`.
- Result: ghtrends repo commit `a99625f` (`main/repos/microsoft-markitdown.md`,
  `main/index.md`), pushed.
- Runs: `agfront/.local/agent/character_talk/run-0001…0007.json`, $0.875 in total.
- Code: agdevworld `516fc97`, `69e878e`, `ab23a6e`, `e6c1b0c`; agfront
  `1be5b8f`, `23012f1`, `120a872`; agautolab `953127d`; pj-agdev pointers
  through `c018c03`.
- Screenshots and the CDP driver: `agdevworld/.local/shots/frontdesk/`,
  `agdevworld/.local/deskshot.mjs` (ignored).

## Remaining issues

See `report4.md`: the `agautolab1` VM not redeployed; ✔ glyph fallback;
page label vs chips at very small widths; a pre-existing agautolab intro
test failure; run cost per exchange.
