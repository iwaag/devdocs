# fix p1: the placeholder fan-out and its recovery loop

Date: 2026-11-02 (times JST).

## Result

All 21 managed projects' `devdocs/workflows/freework.yaml` on `main` have the `placeholder` node after
`workflow__talk`. All 21 runs are completed with their integration recorded as `integrated`, all 21 `freework`
workflows report `ready`, `valid: yes`, and `wfe observe` lists no problems. No recovery run remains open.

## What happened

Conversation `20261102-190239-2bvbp8`: the Developer asked for a `placeholder` node after `workflow__talk` in every
project's `freework`. At 19:55 ponderer (`conduct/run-070`) sent one identical `wfe request` to each of the 21 projects
and reported that the results were not yet observed. That conversation has had no message since.

1. 19 of the 21 first attempts ended without recording any finished run. Only jevlab and cagent completed on their
   first attempt.
2. Each of those 19 was held as interrupted, typically with `nothing-recorded`. Observer opened an incident and
   `warden/recover` continued the run.
3. Many resumed attempts ended the same way, so a second wave of incidents began at 20:25. rts-vs-bot then completed on
   its third attempt.

Totals: 53 freework attempts, 31 recovery runs (`warden/recover/run-074` to `run-104`), 84 launches. Of the 31
incidents, 11 closed `restored`, 9 `interrupted-again` and 11 `held`.

## Why

Observed in the attempt logs:

- The request's step 4 asked the agent to deliver the change on the published default branch. `wfe run integrate`
  refuses during an executing attempt, so an agent cannot do that from inside its own attempt.
- Many agents then ended their session with a note that someone must integrate later. The wfrun guide defines the
  session's final text as an execution log, so nobody read that note.
- Several agents pushed to `main` directly, outside the run record. protoprey and aesthete are confirmed from their
  logs and records; glossary reached `main` with no `run.integrate` record.
- A run that records its nodes through to completion is integrated by the executor without any agent action.

Inference, not established: agents treated a small, fully specified edit as a one-shot task, not as a run whose nodes
must be recorded. Nothing here shows why jevlab and cagent behaved differently, or whether the slim guide
(`unslop_guides` p1) contributed.

## What was done

Requested by the Developer after the investigation; performed by the Omni Agent, recorded as
`Omni Agent (Developer iwaag instruction)`.

1. 20:48: `wfe run stop` on the 11 unfinished runs. rts-vs-bot had already completed. Observer closed their incidents
   as `held`, and no further recovery run was created.
2. Integrating by hand turned out to be impossible: `wfe run integrate` refuses a stopped run until it is resumed.
3. `wfe run resume` with an explicit instruction: no edits, no direct push, no `wfe run integrate`, verify the file,
   record the run through to completion. wfrun-trial went first as a trial and completed in one attempt, so the other
   10 followed. All 11 completed on that attempt.

This differs from the plan given to the Developer, which was to integrate from outside and then close the runs: each
run took one more agent attempt, and runs were completed rather than cancelled.

## Final state

| Project | Run | Attempts | Project | Run | Attempts |
|---|---|---|---|---|---|
| rts-vs-bot | run-001 | 3 | glossary | run-005 | 3 |
| ppt | run-001 | 2 | lowerpoly | run-005 | 3 |
| agsfx | run-003 | 2 | aghangout | run-004 | 3 |
| jevlab | run-001 | 1 | ponderer | run-002 | 2 |
| warden | run-002 | 2 | characters | run-010 | 3 |
| forge | run-006 | 2 | gimmickimg | run-020 | 2 |
| archsage | run-001 | 4 | misc | run-011 | 3 |
| cagent | run-002 | 1 | protoprey | run-011 | 3 |
| playprey | run-005 | 3 | curator | run-010 | 3 |
| front | run-001 | 2 | aesthete | run-007 | 3 |
| wfrun-trial | run-002 | 3 | | | |

archsage's fourth attempt exists because the stop caught a third attempt that had just begun.

Verified with `wfe run integrate --status` and `wfe projects read` for each project, and `wfe observe`.

## Not done / open

- Recurrence is not fixed. The conflict between a request to deliver and integration being refused during an attempt
  is still there, and the same fan-out could repeat this.
- `README_DEV.md` still says recovery has no budget (`no budget since p11`), while `wfe run help recover` documents a
  limit of 2 recoveries since the last person's instruction (`recover-budget`). archsage's third attempt was the
  second recovery. The README was not changed.
- ponderer's conversation was not told the outcome, and `conduct/run-070` is still awaiting input.
- 21 `warden/evidence-fixture` jobs from 2026-10-27 remain queued (`waiting for worker studio`), although those runs
  read as done. They predate this incident and were left alone; they make the executor's queue count read high.
- The placeholder template's real use in a future `freework` run is unobserved.
- `git fetch` was run in the project workspaces during the investigation, which moved their remote-tracking refs.
- This report is not committed.
