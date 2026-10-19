# project_migration p8 ex1 — Step 4 report: one status, short reasons

Date: 2026-10-19. agdevworld `0251983`; deployed in Step 5.

## The shared reading

`workflow_editor/shared/status.ts` `runStatus(record, facts?)` is the one
interpretation of a run's state for people. It reads the record's work,
control and execution facts — plus, where the caller has them, the
execution host's job for the run, a child's current situation and node
names — and returns:

- `state`, `label`, `reason`, `text` (`label · reason`), `tone`;
- `waits[]`: each wait's kind (`person`, `delegation`, `external`), target,
  short text, what continues it, the record's own sentence, and the child
  where there is one;
- `hold` with its structured cause, and `detail`: the full sentence.

Nothing is stored: there is no second status ledger. Contract:
`docs/runs.md` *Status in short*.

`situation()` (the machine-facing word) now separates
`awaiting-person` / `awaiting-delegation` / `awaiting-external` and adds
`due` — the executor's move the record owes (an answer or a child's fact
not taken up, an instruction not read, work left ready). An answer
recorded but not taken up used to read `awaiting-person`.

| Situation | Text now | Before |
| --- | --- | --- |
| question to the run's person | `Waiting · Your answer` | Waits for you |
| delegation | `Waiting · jevlab study result` (+ `(child interrupted)` / `(child paused by its person)` when known) | "Waits for you" in lists; "Executor: idle, waiting for a person" |
| external result | `Waiting · <its reason>` (`(overdue)` past a deadline) | Waits for you |
| owed, job queued | `Queued · Waiting for an execution slot` | Queued (activity line only) |
| owed, host facts unknown | `Continuing · Answer to q1 arrived` | Waiting for a person |
| owed, nothing queued (host facts) | `Stalled · …; nothing queued to continue it` | — |
| a person's stop | `Paused · <their reason>` | Held (stopped) + paragraph |
| interruption | `Interrupted · Talk node left running at exit` | Held (interrupted) + paragraph |
| lost process | `Unknown · Process lost; outcome unconfirmed` | Held (unknown) |

A hold remains the status when a question is also pending, and the
question's lead then says the answer is recorded but the run continues only
when resumed. `Watching` is not used: nothing watches on a run's behalf.

## Consumers updated

| Consumer | Change |
| --- | --- |
| Project run list (`src/projectView.ts`, `RunSummary.status`, `GET runs` with the host's jobs) | badge from the status (Waits for you only for the person's own answer; Waiting, Queued, Paused, Interrupted, Unknown, Stalled); a status line under each unfinished run in Simple and Detail; holds now visible in lists |
| Run view (`src/runView.ts`, `src/runActivity.ts`, `src/runStatus.ts`) | the activity line says the status (`Interrupted · …`, `Waiting · …`, `Queued · …`); a status chip in Detail's header; delegate node cards know `due`/paused children |
| Run focus (`src/runSide.ts`) | the head is the status text; the lead is one short sentence; the hold's sentence and the generic "work is kept" explanation moved into **Details**; a question beside a hold no longer implies an answer resumes it; child words cover the new situations |
| Delegations API | `childNow.hold`, `childNow.status`, and the parent's `status` with its children as read now |
| Conversations API and CLI | every run linked *and* each message's own entrance run (`turn`) carry `status`; `wfe conversation show` prints it |
| Front Desk (`src/roomState.ts`, `FrontDeskScene.ts`) | each run link reads `work: converse/run-… — Waiting · …`; the strip above the dialogue lists the conversation's unfinished work with its status (held runs in warning colour), clickable to the run view |
| CLI | `wfe run list` adds the status line; `wfe run show` a `Status:` line |
| Executor | the answer-continuation check no longer depends on the situation word |
| Slack bridge | polls a `due` run like an executing one |
| Observer | decides from `waiting.responsible`, holds and causes, not from `situation` (audited: no rule changes needed); docs list the new situations |

## Checks

- `test/status.test.ts` (6 tests): each representative text above from
  records built by the reducer, including the hold-plus-question case, the
  relayed child hold, several waits and old holds read from their sentence.
- Full WFE suite: **440 tests pass**. Three older tests asserted the old
  conflation (`awaiting-person` for an external wait and for answers not
  taken up, `idle` for a recovered run with its job queued) and now assert
  `awaiting-external` and `due`.
- UI checks (`checks/`) contain none of the replaced strings.

## Screenshots (fixture, working-tree build)

Captured by a one-off script that builds five runs in a throwaway area
(the `converse` shape: Talk, Literature survey → Deliver) and serves them
with the working tree's WFE; images in the ignored
`pj-agdev/.local/project-migration-p8-ex1/step4/`:

| File | Shows |
| --- | --- |
| `project-runs-simple.png` | the run list: Queued · Waiting for an execution slot; Paused · Circumstances changed…; Waits for you / Waiting · Your answer; Interrupted · Talk node left running at exit; Waiting · kid make result |
| `run-interrupted-talk-simple.png` | activity line and focus head `Interrupted · Talk node left running at exit`, one-sentence lead, Details collapsed, the resume form |
| `run-delegation-wait-*.png`, `run-your-answer-*.png`, `run-paused-*.png`, `run-queued-*.png` | the same reading in each run view, Simple and Detail |

The same text appeared in the activity line, the Detail chip and the focus
head for every case (script output in the folder's log). The Front Desk is
captured on the deployed service in Step 5, with real conversations.
