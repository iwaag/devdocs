# Project Room p1 — Step 5 report: deployed, verified, reported

Plan: [plan.md](plan.md), step 5.

## Service state before touching anything

`nctl status` (pj-clusterintent): ok, every submodule clean.
`nctl drift --host agstudio`: converged, `agdevworld-web-agstudio` and the
rest of the placements applied. No desired-state change was needed: the
relay and the web container are the existing placements, and nothing new
was added to the realm (no bot, no channel).

## Deployed

- **Relay** (`com.agdev.agentroom`, launchd on agstudio): `kickstart -k`
  twice on 2026-09-19 — after step 4's code and again after `/healthz`
  gained `mirror.calls`. Both starts resumed the event queue (`live (resumed
  queue)`, `resyncs: 0`): no sweep. No plist change; the new routes need no
  new credential.
- **Web** (`:8090`): `docker compose up --build -d web` in `agdevworld`;
  `/?view=project` answers 200 with the new bundle.
- **Settings**: revision `47d5d713fab1` (step 4) served by the relay.
- Dependency pins: none changed (pyagag stays at the lock's pin; the relay
  uses nothing new from it).

## Checks

- Relay tests: `uv run pytest tests/` in `agentroom` — **323 passed**
  (`test_projectroom.py` 12, `test_projecttalk.py` 10, the rest unchanged).
- `npm run build` (tsc + vite): passes.
- **Live reads on `:8094`**: `/projects` → 33 project channels, 9 live;
  `/work/6770` → the YuE2 mission with 4/4 tasks completed, destination
  `#pj-mediagen › workplan-yue2-music-study — the planning conversation,
  answered by autolab-agstudio1`; `/projects/pj-worldtrend/topics/
  researchplan-worldtrend` → a document, not postable, with the Front Desk
  and `?view=argue&argue=7217` as the path.
- **Repeated board reads add no Zulip history sweeps**: `/healthz`
  `mirror.calls` read 3 before and 3 after 60 reads (20 × board, project
  detail, work conversation).
- **Browser on `:8090`, live data** (`.local/shots/project-p1/live/`): the
  board; `pj-mediagen` with ✔ history shown (2 missions, 15 unrecorded
  plans, m6286's archived work channel marked incomplete); the YuE2 plan
  with its four tasks, plan text and links (source in Zulip, the upstream
  docs, the ComfyUI node); task 1's run; `pj-worldtrend`'s research plan
  as a document. No page errors. The demo-fixture flow (step 3) covered
  find → read → comment → reply → revisit, unread dots, drafts, IME, narrow
  layout and direct URL reload; the failure cases (uncertain send, ✔'d
  target, deleted anchor, document refusal, stale mirror) are fixtures in
  the relay tests and the demo.
- Screenshots and drivers are under `agdevworld/.local/` (ignored); the
  local notes are in `pj-agdev/.local/devenv.md`.

## Live posting — still pending

No post went through the new door to the realm. Every such post buys an
autolab run in the conversation it lands in, and the plan asks for a
deliberately selected test conversation and an explicitly authorized post;
neither was given in this run, and the Omni Agent does not pick a live
mission to comment on by itself. What is validated: the mocked poster in
`test_projecttalk.py` (destination after rename, reused names, duplicate
token, uncertain delivery, completed and deleted targets, the document
refusal) and the demo round trip in the browser. To close it: pick a
`workplan-` or `workrun-` conversation that may receive a comment, open it
in the room, post, and watch the reply arrive on the 4 s poll.

## Found in the realm while looking

- `pj-mediagen › ✔ workplan-yue2-music-study` (m6770) carries `planned` as
  its recorded state and ✔ on the topic: the acceptance on 2026-09-13 was
  written in prose and the topic resolved by hand, so autolab's record never
  got `done`. The room shows it as a `resolved-unfinished` gap rather than
  a success — the plan's "a resolved topic does not by itself prove
  successful work", met live on the first real screen. The completion door
  (`/complete`) would have written the `done` note; a hand resolve does not.
- Six of nine live project channels have no `goal` / `researchplan` topic:
  they predate `agproject`. They read as `no-document`, which is the realm,
  and the room's path to Front is the same for them.

## Docs

`agdevworld/README_DEV.md` (Project Room section), `agentroom/README.md`
(`/projects` and the talk routes, steps 1–2), `devdocs/README_DEV.md`
(a paragraph beside Memos and the Arguing Room), `pj-agdev/.local/devenv.md`
(the machine-specific notes). Commits: agdevworld `e3c211b`, `e268f17`,
`ed96560`, `6afaf2f`, `2d25861`, `83e33a9`; pj-agdev pointers up to
`b2c56f5`; agdevworld-settings `47d5d71`; devdocs per step.
