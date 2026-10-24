# fix5 step 5 — verification (before deployment)

agdevworld `3d569db`. The deployed trial is in [report6.md](report6.md).

## Required checks

- WFE `npm run check`: tsc, **557/557 tests**, and both vite builds pass.
- agautolab `pytest`: 47 passed, 1 skipped (step 4).

## Behavioural checks (`test/reminders.test.ts`, 15 tests; `test/workers.test.ts` +1)

| Plan item | Covered by |
| --- | --- |
| A missing presentation is warned before any mutation or outward post; a repair lets the retry through; `--yes` continues and does not suppress correctness errors | the last-work refusal leaves seq and state unchanged; a planned present node at `adjust --completing` passes; `--yes` still gets `state` and `outdated` errors |
| CLI, direct API, remote worker and combined adjust-and-complete behave the same | CLI exit 1 then `--yes`; HTTP `ops` and `adjust` return 409 and accept `yes`; a forwarded `present register` gets the same refusal and `--yes`; the combined path writes no revision file |
| Concurrent changes and repeated requests neither duplicate effects nor apply a stale revision | `expectSeq` is stale even with `--yes`; of two concurrent completions one records; a same-receipt retransmission records nothing (CLI, record-done) |
| A presentation answer reaches the agent normally; a request for changes does not stop the page; a finished review prompts cleanup; a verified stop clears the warning; planned cleanup and multiple revisions are handled | the answer "Make it blue." appears in `pendingAnswerLines` with its reminder, once; nothing is stopped; completing the review is refused until `presentation.observe stopped`; a planned cleanup node satisfies it; completing the cleanup while the page still serves does not; "after the latest presentation" compares registration seqs |
| A retrospective node is added and completed atomically, marked honestly, linked to real evidence, and never executed; a retry, including on a completed run, duplicates nothing | step 3 tests: one entry, `[null, 'completed']`, no start or end, one `completed` event, evidence checked, the link keeps the registration, `checkRun` replays |
| Custom node ids and catalog-derived nodes work; consultation-only flows collect no irrelevant warnings; self-checks say unrecorded work is unknown | custom ids `build`, `page`, `tidy` (cleanup by icon), `show`, `wrap`; a consultation question at the start of a talk gets nothing; the self-check says "WFE cannot tell whether work was done" |

## Replay against real records (read only; no record changed)

The script replays each run's history up to the entry that completed the
run. It evaluates that completion as if it had come through the CLI, with
the run copy's current files as work signals.

| Runs | Result | Reading |
| --- | --- | --- |
| gimmickimg freework run-001…008 (all work inside the talk) | `unrecorded-activity` in all 8; `presentation-serving` in 003, 005, 007, 008 | the failures fix3/fix4 described, now caught at the boundary |
| gimmickimg freework run-009 (talk → sheet → present → confirm → cleanup) | none | |
| gimmickimg freework run-010 (work nodes, no presentation) | `presentation-missing` (composite, loops, scene, verify) | the braindump's case |
| gimmickimg freework run-011 (present → review → cleanup) | none | files copied in the present node are explained |
| rts-vs-bot improve-game run-002…007 | none | run-001 predates frozen graphs |
| front converse run-001…052 | 51 none; `presentation-serving` in run-051 (p1 served, no cleanup); `unrecorded-activity` in run-004, 033, 034 (`knowledge/*.md` written during the talk) | the last three are real file writes during a conversation turn. The self-check asks rather than asserts, so `--yes` is the expected answer there |
| front discuss, forge converse, agops recover, cagent change | none | |
| ponderer conduct (30) | `discussion-activity` | this replay treats every completion as a CLI completion. In practice these complete through `wfe discussion …`, which WFE records without advice, so they are not evaluated |

Two rules were narrowed because of this replay:

- **work** is the `work` template only. Treating untemplated nodes added
  by a run as work flagged front/converse's `tell` and `report` nodes, and
  a cleanup node without an icon.
- **A talk that registered a presentation stays a talk.** Otherwise run-005
  would have explained its own talk-time writes.

A changed file counts only when it was written while no node of its own
was running. This is judged from the file's mtime against the record's
running intervals, with 60 s of slack. That rule explains run-011's
present-node copies.

## Known limits of the signals

- **mtimes are approximate.** A copy re-materialized from Git has checkout
  times. On a remote worker, the coordinator's record copy holds the work
  only after the attempt publishes it, so a self-check during a remote
  attempt sees only recorded artifacts, outputs and talk time.
- **Work outside the run's copy is invisible.** This includes other
  machines and shares not recorded as outputs.
