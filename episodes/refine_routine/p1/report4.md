# refine_routine p1 — step4 report: the screens and completion follow the new structure

Date: 2026-09-10

## Relay (`agdevworld/agentroom`)

- **`routines.py` rewritten.** A routine is `#routine-<name>` with its
  `guide` topic and `routinerun-` topics. The board reads, per routine, the
  guide (the newest post, whole, with `posts` and `authors` so a stray post
  there is visible), `retired` (a ✔ on the guide), the runs newest first by
  their opening post's id, and the latest run's summary. Per run it reads
  the opening post, the origin (Front's root note in the run topic — the
  requester; absent for a run opened by hand), Front's entries, the
  `ag-routinerun` finish block (`achieved`, `reason`, `report`), the ✔
  resolution, and a state read off the posts: `unstarted`, `acked`,
  `waiting`, `awaiting`/`stalled` (somebody else posted into the run),
  `finished`. Ending a run and reaching the goal stay two sentences. The
  session tree is walked from the run topic exactly as before (served and
  root notes, never rendered); a root note in the run naming the requester
  is the origin and not a child. Payload schema `ag.routines.v2`, no
  schedule anywhere.
- **Sweep.** Routine channels are public, so the observer's sweep and queue
  already carry them; the guide and the newest three run topics of each
  routine are read deep and under ✔ (a finished run is resolved by Front's
  listener). `#front` keeps its desk rule. `AGENTROOM_SCHEDULE_JSON` is
  gone from the code and the plist template.
- **Writes.** `POST /chat {channel, topic, text}` posts only into a known
  routine's `guide` or `routinerun-` topics. `POST /routines/<name>/start`
  no longer opens a run topic: it posts the request as the Developer at
  Front's ordinary entrance, a Front Desk conversation of its own, and
  answers with `desk`, so the screen can open that conversation; Front reads
  the guide, opens the run, and the report returns to that conversation.
  Refused before posting when the guide is ✔ or missing.
- **Completion (`closing.py`).** A run is classified by its channel; its
  guide is `routine-guide`, a retiring kind that is never a target. A run's
  scope names the guide as untouched context and its requester as the
  parent that stays open; the guide, the channel and other runs are never
  reached. Ownership, work notes, Plane and channel archiving are unchanged
  (a routine channel is never a `work-` channel).
- **Cost** sessions carry their channel; the in-flight signal watches the
  run's channel; the Front Desk owns the `#front` constant.

## Frontend (`agdevworld/src`)

- `routineState.ts` types and calls follow `ag.routines.v2`; `requestRun`
  replaces the fire.
- Operation dashboard: routine cards show the latest run's state and
  evidence; the run list shows opening, origin, state, entries, finish and
  resolution; "Ask Front to run it" posts the request and shows where it
  went with a link to the Front Desk conversation; the run record panel is
  the run topic; `finish ✔ this run` completes the selected run in its own
  channel. Routines view, detail popup (latest run, guide, runs), chat
  panel, session graph and cost gauge updated. `tsc` and `vite build` pass.

## Verification

- 225 relay tests pass; `tests/test_routines.py` rewritten for the new
  shape, `test_scope.py` fixtures moved to routine channels (a run closes
  its work only, the guide is never a root, another run is another request,
  the re-run and archived-channel cases unchanged), `test_ops.py` and
  `test_cost.py` adjusted.
- Live: the relay was restarted on this code; `/routines` lists the four
  routines with their guides, `idle`, no runs; `/routines/ghtrends` carries
  the guide post. Browser: the dashboard and the routines view render
  (screenshots under agdevworld's ignored `.local/shots/rr1/`); the "Ask
  Front" section shows the guide and the conditions box.
- Not yet walked in the browser: request → run detail → finish, because no
  run exists yet. Step5's live run covers it.

## Left for later

- The installed relay plist still carries the unused schedule variable
  until its next `bootout`/`bootstrap` (step5 or 6).
- The web image on `:8090` is not rebuilt yet (step5).
