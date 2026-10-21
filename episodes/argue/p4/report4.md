# argue p4 step 4 — progress shown, comparable records kept

## Delivered

| Component | Commit | What |
| --- | --- | --- |
| aghangout (deployed) | `a573f09` | Meeting room **Progress** panel: the conductor run (status, link to its run view and graph) and requester; *Now* (each waiting activity, whom it asks, who answered with message links); *Decisions* newest first (text, revision, the messages it rests on as links, next activities); *Activities*; *Still owed* (obligations with owner and later receiver). Status line names a running ask with answered/asked counts. Announcements are labelled. Records gain *Outside the speech* (annotations). Ordinary speech stays in the dialogue and History; the graph is linked, not inlined |
| WFE | `a770891` (deployed), `c5ab9c9` | Home Discussions card and `api/discussions`: each discussion's conductor and what it does now (`now: asking forge, sage`, or `last: announce announce-conclusion`). Run view cards carry the activity badge. `ponder --next` names a new node by its id (deployed check: cards read only "Do") |
| WFE archive | (step 2) | transcript `activity` / `annotation` records; manifest `conductor`, `requester`, `activities`, horizon `activity`, `annotation` |
| pj-argue | `fe64ad4` | `episodes/index.json` (`ag.argue.episodes.v1`), `episodes/README.md` (the analysis format), analyses `argue-p3.md`, `argue-p3-ex1.md` with their recorded limitations and `unknown` conditions; `tools/argue episodes [<id>\|validate]`; `show`/`validate` read activity and annotation records |

Analysis format (episodes/README.md): task; conditions in short; path;
observations per comparison question (question selection, repeated
discussion, waits and owners, interventions and effect, completion
against speech, accuracy of the final account, preservation);
limitations; sources. Unrecorded conditions are `unknown`, e.g. "whether
the review and summary would have happened without hints 1 and 2" (ex1).

## Checks

- aghangout: `npm run check` (13 tests, one new for the progress model),
  fixture browser check (webkit) all passed, with new checks: progress
  shows the conductor and its decision, a decision links its sources, and
  what it asked whom.
- **Deployed** (webkit against `http://agstudio.local:8093`,
  `step4/deployed.mjs`), 13 checks passed: Progress on `p4-s3-check`
  names `ponderer/conduct/run-001`, lists three decisions and every
  activity kind, "Nothing is owed"; the run-view link opens the conductor
  graph with six activity badges; a decision's source opens its message;
  History marks the announcement; `api/discussions/p4-s3-check` has six
  activities; the pre-p4 `argue-p3-ex1-trial` (27 messages) opens Progress
  without error and lists all 27 in History; Home states the conductor's
  last activity; no page errors.
- **Archive**: `p4-s3-check` is `current` (`de4adf5` through #213). A fresh
  clone of pj-argue from Gitea: `tools/argue validate` ok for all seven
  archives, `episodes validate` ok (2), `show p4-s3-check` reads the
  requester, conductor and activity events. Retry-safety of publication is
  the step 2 test (a second export publishes nothing).

## Observations

- In the deployed room, after jumping to #209 from a decision while a
  newer line was still typing, the dialogue box showed the start of #213
  ("Concl…") under #209's name plate until the next move. This is
  aghangout's existing playback code (not changed here); a candidate for
  aghangout-agent.
- The talk activity in the archive shows `waiting` until reconcile records
  its take-up (once a minute); the live store had `completed` at once
  after the run completed and the next reconcile.

Evidence (ignored): `pj-agdev/.local/argue-p4/step4/` (`deployed.mjs`,
`deployed-agstudio.txt`, `deployed-progress.png`, `deployed-run-graph.png`).

Deus Ex Machina note: I made the aghangout change and deployed it —
a handoff candidate for aghangout-agent.
