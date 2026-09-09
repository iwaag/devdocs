# refine_routine p1 — step5 report: deployed, and one real run proven end to end

Date: 2026-09-10

## Deployed

- Tests: agfront 91, agentroom 226 (one added after the live run), frontend
  `tsc` + `vite build` clean. Front's listener restarted on the step2/3
  code, the relay on the step4 code (and again after the fix below), the
  web image rebuilt with the step4 frontend.
- Front's introduction re-posted in `#agents` (message 5508): it says
  Front runs routines from guides and that `routinerun-` is its own
  prefix. The ops board now reads run topics as Front's (roster prefixes
  `front-`, `routinerun-`).
- The old schedule variable stays in the installed relay plist until its
  next bootstrap; the code ignores it.

## The live run

Requested through the new door, `POST /routines/ghtrends/start`, which
posted as the Developer into a fresh Front Desk conversation
(`#front › front-desk-20260909-152341`, message 5509): "run ghtrends now …
one repository only, end as soon as autolab reports the commit; before
starting, read the budget and start nothing if the Claude 5-hour window is
already at 90 % or more".

| when (UTC) | what | evidence |
|---|---|---|
| 15:23:41 | Front Desk serving reads the guide (#5498), opens `#routine-ghtrends › routinerun-2026-09-09T15:24` with one opening post (request verbatim, conditions as read, guide id, origin), replies to the developer | #5511 (root note), #5512, #5513 |
| 15:24:32 | listener starts the run right after the desk reply (`starting run … opened from front-desk-…`) | listener log |
| 15:25:15 | first run serving: reads `tools/budget.md` (5-hour window 50 %, below 90 %), delegates to autolab in `#pj-ghtrends › workplan-trend9` (#5516), records the entry | #5518 |
| 15:25:41 | autolab's plan names Front → the mention resumes the **run**, not the desk; Front posts the task start into `work-g-19 › workrun-task1-g-19`, records it, marks the callback served | #5527, served mark up to 5522 |
| 15:27:11 | autolab reports commit `dc6258d` (Tencent/teamai-cli) and resolves its task; the callback resumes the run | #5530 |
| 15:27:51 | run serving judges the end condition met, writes the entry and the `ag-routinerun` block (`achieved: true`); the listener delivers the report into the desk conversation and resolves the run topic | #5534, #5533 (desk), ✔ on the run, served mark 5530 |

Three `routine_run` servings, claude_code/sonnet-5, about $0.47 in total
(records `run-0001…0003` under agfront's ignored records). No post landed
in the desk conversation between the request and the report except Front's
own opening reply, so the desk cost one run to open and none to wait.

- Restart recovery, live: the listener was restarted after the run; the
  startup sweep found nothing to serve (the served marks bound the
  callbacks, the run is ✔).
- Screen: the dashboard shows the routine as `finished · 1 run · 0 open`
  with the finish reason; with "Show resolved" on, the run card (opened by
  Front, from the desk conversation, 4 entries, 2 linked conversations,
  goal reached, ✔), the conversation flow (run → workplan → workrun), and
  the run record with the finish block. `finish ✔ this run` previews the
  completion: the run already ✔, the guide "mentioned, not touched", the
  desk conversation "opened for … stays open", the work channel archivable.
  Screenshots under agdevworld's ignored `.local/shots/rr1/`.

## Usage condition

- The boundary cases (below, already reached, reset passed, failed read)
  are proven by the step3 fixtures, not live.
- The connection is proven live: the run's first entry quotes the budget
  read (50 % at 15:24 UTC) and the last entry re-reads it (52 %); the
  claude_code card had recovered from the expired-token failure seen in
  step3 by then. The condition itself never bit — 90 % was chosen so a
  short real run could complete.

## Defect met live

- The host-observation line read `KeyError: 'fire'`: `/inflight/<name>`
  still built its session block from the retired fire field. Fixed
  (channel, topic, opening post), a test added, the relay restarted.

## Deus Ex Machina note

- The run request was posted by the Omni Agent through the relay's door,
  standing in for the developer; everything after it was in-system. Front
  had to post the task start into autolab's `workrun-` topic itself, which
  is autolab's contract, and it did so unprompted.

## Not proven

- A usage condition reached mid-run, a hold, or a run opened by hand.
- The other three routines have no run yet.
