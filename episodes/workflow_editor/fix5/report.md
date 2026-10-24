# Workflow editor fix5 — report

Plan: [plan.md](plan.md). Step reports:

- [report1](report1.md): evaluator and coverage
- [report2](report2.md): responses on every path
- [report3](report3.md): `record-done`
- [report4](report4.md): guidance
- [report5](report5.md): verification and replay
- [report6](report6.md): deployment and trials

## What changed

| Braindump wish | Now |
| --- | --- |
| "After implementing, changing or experimenting, did you present it in a presentation node?" | Completing the run or its last work node, a result report, or a review question that presents nothing gets **`presentation-missing`** when completed `work` is not covered by a later or linked presentation and no present node is planned |
| "After presentation feedback, did you forget cleanup?" | An answer about a presentation carries a reminder in the brief, the continuation and at take-up: more work, or cleanup is due. Completing the review, a cleanup node, a report or the run with a page still serving and no cleanup planned gets **`presentation-serving`**. Registering from a talk or work node gets **`presentation-inside-work`**, and registering with no cleanup planned gets **`cleanup-unplanned`**. Nothing stops a page by itself |
| "Before completing or reporting with ask: did you run something without a node? Here is how to add it afterwards" | Asks, reports and the run's completion get the self-check **`unrecorded-activity`** when no work node is visible but there are signs of work. It says WFE cannot know, and names **`wfe run record-done`**. That command records already-performed work as a completed node, honestly marked, with checked evidence |

How it works:

- One evaluator, `shared/reminders.ts`, runs inside each operation under the
  run's lock, so a refused operation records, posts and queues nothing.
- The same refusal reaches the CLI, HTTP (`yes`), remote workers and the
  run view.
- `--yes` continues past the reminders for that one command only.
  Correctness errors stay errors.
- Roles come from template provenance, icons and presentation records,
  never from node names.

## Validation

- WFE `npm run check`: 558/558, tsc, both builds. agautolab: 47 passed,
  1 skipped.
- Every bullet of plan step 5 has a test: 16 in `test/reminders.test.ts`
  and 1 in `test/workers.test.ts` (report5).
- A read-only replay of real records reproduced the braindump's cases
  without false alarms on well-formed runs:
  - gimmickimg freework run-001…008 → `unrecorded-activity`, and
    `presentation-serving` where pages were left serving;
  - run-010 → `presentation-missing`;
  - run-009 and run-011 → nothing;
  - rts-vs-bot improve-game, front discuss, forge converse, agops recover
    and cagent change → nothing;
  - front converse → 51 of 52 runs nothing; the rest are real talk-time
    file writes and one page left serving.

  The replay narrowed two rules: `work` means the template only, and a
  talk that registered a page stays a talk.
- Deployed checks found four defects. All four were fixed and redeployed
  (report6).

## Trial run references

- **gimmickimg freework/run-012** (real; requested by `fix5-check`):
  - work was put on a `work` node before it was done;
  - the completion got `presentation-missing`, and **the agent answered
    with `--yes` at once**;
  - it then used `--yes` pre-emptively on its later completions;
  - no presentation was made. This is the accepted limitation, and nothing
    was added to counter it.
- **Retrospective fixture** (scratch area, deployed release, `fix5-check`):
  the self-check named the hidden work; `record-done` recorded it with
  artifact, commit and entry evidence; a retransmission recorded nothing;
  the presentation reminder still applied; the run view marks the node
  ([screenshot](run-view-retro.png)).

## Remaining limitations

- **The trial did not change the agent's behaviour.** run-012 used `--yes`
  immediately and presented nothing. The reminders make the omission
  visible at the moment it happens and in the record (`--yes` is in the
  attempt log), but they do not make the agent present. Further changes
  are the Developer's call, for example the reminder's wording or what the
  freework talk says. The plan rules out hiding the flag, enforcing
  reflection or adding approval.
- **The self-check's signals are heuristics:**
  - file mtimes are compared with the record's running intervals;
  - artifacts and outputs are recorded facts;
  - a talk counts as long after 10 minutes;
  - on a remote worker, files reach the coordinator only when the attempt
    publishes;
  - work outside the run's copy is invisible;
  - front/converse turns that write `knowledge/*.md` get the self-check,
    and answer it with `--yes`.
- **Only nodes from the `work` template are asked to be presented.**
  Untemplated nodes an agent adds are not.
- **A release before fix5 cannot replay records that contain
  `run.record-done`.**

## Recovery references

- The ignored `pj-agdev/.local/fix5/notes.md` holds:
  - the previous release and the activate command;
  - how to find and restore records that contain `run.record-done`;
  - the agpc batch values and playbook logs
    (`pj-clusterintent/nctl/.local/fix5/`);
  - the store backup (`.local/agdev/.local/backups/fix5-predeploy-20261024-151921/`);
  - trial log and fixture paths.
- `pj-agdev/.local/devenv.md` has a fix5 section.
- `devdocs/README_DEV.md` has the fix5 entry.

Historical runs were not rewritten. No reminder, record-done or adjustment
was applied to them.
