# fix5 step 1 — shared warning evaluation and operation coverage

agdevworld `8ff4b84`.

## Entry points mapped

| Boundary | Entry point | Where the check runs |
| --- | --- | --- |
| Completion | `wfe run complete` → `runOp` `node.complete` | `runOpLocked`, on the record the write would produce |
| Combined adjustment + completion | `wfe run adjust --completing` → `adjustRun` | inside `adjustRun`'s lock, on the revised record, before the revision file is written |
| Question | `wfe run ask` → `runOp` `question.ask` | `runOpLocked` |
| Presentation registration | `wfe present register` → `registerPresentation` | inside its lock, before any file is copied |
| Outward report / completing post | `wfe conversation reply --kind result\|synthesis`, `--completing` | `adviseRun` (run lock, read only) before the post |
| Answer take-up | `wfe run take-up` | advisory text after recording, never a refusal |
| Resumed attempt | brief and continuation (`pendingAnswerLines`), `wfe run state` | advisory text with the pending answer |

The transition that completes authored work is the completion that moves
the run's execution to `completed`. Documentation steps such as
`_final-report` do not hold it open (`summarize`), so the check keys on
that transition, not on `_final-report`. `freework/run-010` completed at
seq 15 (`composite`), before `_final-report` ran.

## The evaluator

`shared/reminders.ts` `evaluate(boundary, facts)` returns structured
reminders (`code`, `text`, `repair`, `refs`). `server/reminders.ts`
`adviseOrRefuse` adds the file-layer signals and refuses with one
`RequestError(409, …, {code: 'reminder', reminders})`. Callers opt in by
passing `advice`. CLI and HTTP operations pass it. WFE's own records do
not: attempt ends, intake, and a discussion completing a node for a run.

Node roles come from structured records only:

- **present**: template `present`, the `present` icon, or a node that
  registered or was linked to a presentation;
- **cleanup**: template `cleanup` or the `cleanup` icon;
- **work**: template `work`, or an untemplated `do` node that the run added
  itself (not in its original definition);
- **confirm**, **talk**, **activity** and **step** come from the graph.

| Code | Boundary | Condition |
| --- | --- | --- |
| `presentation-missing` | run-completing completion; completing the last work node; result report; a result-confirmation ask with nothing presented | completed work after the latest presentation's registration, not linked to it, and no present node planned |
| `presentation-open` | run-completing completion; report; completing a cleanup node; completing a node that took an answer about a presentation | an open presentation and no cleanup node planned |
| `presentation-inside-work` | registration | registered from a talk, work, confirmation or activity node |
| `cleanup-unplanned` | registration | no cleanup node planned |
| `unrecorded-activity` | ask, report, run-completing completion | catalog templates exist, no work node is visible, and there is a sign that work happened: changed paths in the run's copy outside its records, artifacts or outputs, or a talk that ran for more than 10 minutes |
| `discussion-activity` | completion | fix3's activity check, moved into the shared evaluator |
| `discussion-open` | `wfe run complete` | fix3's conductor check (it needs the conversation store), passed in as `extra` and merged into the same response |

A take-up of an answer about a presentation, and the pending answer in a
resumed brief, carry `answerReminder`. It reminds the agent to decide
whether more work is needed or cleanup is due. It never stops a page. It
appears once with that answer.

## Model additions used by later steps

- `run.record-done`: one history entry that adds a revision with one new
  node, completed.
- `NodeRecord.retrospective`.
- `Presentation.links`.

These are in the reducer now so that the evaluator can treat a linked
presentation as covering the node it is linked to (step 3).

## Checks

`test/reminders.test.ts` has 8 tests:

- refusal before mutation, then repair and retry;
- `--yes` does not suppress correctness errors and does not carry over to
  later commands;
- system operations are not evaluated;
- a warned adjustment writes no revision file and no record;
- a registration from a talk copies no files;
- review prompts cleanup, and a verified stop clears the reminder;
- a planned cleanup satisfies the review reminder, while completing the
  cleanup with the page still serving does not;
- a consultation question gets no reminder;
- in the CLI: exit 1, then `--yes`, then a retransmission records nothing.

The fix3 test now expects the consolidated text. Before these changes the
full suite was 540/541; the one failure was that fix3 wording. tsc is clean.
