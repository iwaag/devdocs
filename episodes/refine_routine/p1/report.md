# refine_routine p1 — report

Date: 2026-09-10. Steps: [report1](report1.md) … [report5](report5.md).

## What changed

- **Routines are guides in Zulip.** Channel folder `routine`, one public
  channel per routine (`#routine-publish`, `#routine-study-realworld`,
  `#routine-ghtrends`, `#routine-papers`), a fixed `guide` topic whose
  newest post is the whole guide. The four live standing requests were
  moved there as guide v1 with only the schedule wording changed; the five
  routines the developer had already ✔'d were not moved. The dispatcher,
  its GUI, `rtschedule`, the launchd templates and Front's schedule tool
  are gone; nothing is fired on a timer.
- **A run is a topic Front owns.** Asked at its ordinary entrance, Front
  reads the guide and opens `routinerun-<id>` with one opening post
  (request verbatim, conditions as read, guide post id, origin). The
  listener starts the run after the serving that opened it (and at
  startup), the new `routine_run` role serves it with its own guide,
  delegates from it, is resumed by callbacks through the existing root-note
  and served-mark machinery, records each serving, and ends it with an
  `ag-routinerun` block; the listener delivers the report to the origin and
  resolves the run. Ending and achieving are two fields.
- **Observation.** `agbudget` and `tools/budget.md` give a run the plan
  windows (relay `/budget` or a fixture file); the guide fixes how "until
  N % used", "consume N from the start", resets and failed or stale reads
  are read.
- **Screens and completion.** The relay reads guides and runs
  (`ag.routines.v2`), the dashboard and routines view show guide, runs,
  origin, entries, finish and resolution, "Ask Front" posts the request into
  a Front Desk conversation, and completing a run touches that run and its
  work only.

## Verified

- agfront 91 tests, agentroom 226, frontend build; step2's suite found one
  real bug (callback into a run hit the empty-topic guard), step5's live
  run found one (`/inflight` read the retired fire field); both fixed.
- One real `ghtrends` run, requested through the new door: opened from a
  Front Desk conversation, started by the listener, delegated to autolab,
  resumed twice by callbacks into the run (never the desk), ended on the
  end condition with `achieved: true`, reported into the desk, ✔ on the
  run; a listener restart re-served nothing. About $0.47 of Front runs.
  Evidence: `#front › front-desk-20260909-152341`, `#routine-ghtrends ›
  ✔ routinerun-2026-09-09T15:24`, commit `dc6258d` in the ghtrends project.
- Usage-condition boundaries by fixture; the observation connection live.

## Left

- Not proven live: a usage condition reached mid-run, a hold, a hand-opened
  run, the other three routines.
- Nothing resumes a held run by itself; developer interrupts, forced
  cancel and timed or recurring runs are the next phase.
- Reusing a delegate topic across runs would misroute the second run's
  callbacks (a topic is anchored once); the run guide asks for a fresh
  topic per delegation.
- Local details (hosts, stream ids, credentials) are in the ignored
  environment memo.
