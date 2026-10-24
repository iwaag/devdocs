# fix6 step 5 — behaviour and failure recovery checks

Plan: [plan.md](plan.md) step 5. Code: agdevworld `4888ea3`, `09596f6`.

## Results

- WFE `npm run check` (tsc, all tests, both builds) on `4888ea3`: 571/571
  (558 before fix6; 13 new). `09596f6` adds assertions to two of them; the
  release build ran the check again (report6).
- agautolab `15258d7` (guide only): pytest 47 passed, 1 skipped.
- Every registered project's 351 run records replay under the new reducer
  (read-only, scratch script): the new fields are optional, so no migration.

## Plan bullets → tests

| Plan item | Test |
| --- | --- |
| Work-template output, untemplated substantive work, screenshots/video, a defensible report-only result; no warning from coordination nodes alone | `test/assessment.test.ts`: "work-template output…", "untemplated substantive work and screenshots/video…" (a study's notes-only result and a no-file `relay` node raise nothing), "the child's own completion warns for untemplated substantive work" |
| A parent without work nodes is reminded at disposition and outward reporting; a satisfied delegation stays assessable; a repair removes the warning | `test/correction.test.ts`: the first two tests (disposition, run-completing completion, satisfied still assessed); the same-child test (after the correction `delegatedReminders` has no `presentation-missing`, and `adviseRun` with a `report` boundary passes) |
| Confirm node, an agent's own answer, a person's relayed answer, a decision with no human answer | `assessment.test.ts` "the person's review…" (run-005's shape is `unknown`; a `because` from Pat and Pat's own answer are `ok`; `accepts: parent` has no check) |
| A presentation linked to the result, an older unrelated one, requested changes, requested continued availability, verified cleanup | "work-template output…" (older page does not cover later work; a later page covers it), "a page still serving…" (open review, the answer shown, `wfe present keep`, verified stop). A presentation *linked* to work recorded afterwards uses fix5's `links`, covered by fix5's record-done test; the assessment reads the same field |
| Warned operations have no partial effects; repair and `--yes` across CLI/API/remote and combined completion paths; stale results and ownership errors not bypassed | `correction.test.ts` "CLI and HTTP…" (exit 1 / 409, nothing recorded, `--yes` / `yes`, retransmission once), the stale-manifest cases with yes, `result-returned` with yes. Remote workers forward `wfe delegate` and `wfe conversation` like every command (fix5's forwarded-reminder test covers the mechanism); deployed check in report6 |
| Correction survives retries and crashes at each handoff, resumes once, keeps completed work, produces a new result, rejects stale disposition, waiting states observable | the same-child test (revise retransmission, lost acknowledgment, repeated intake application, one continuation, completed work kept, new result and manifest, old result `result-superseded`); the "unaddressed" test (lost continuation after a coordinator restart re-queued by reconcile, `missed-correction` hold, `--no-change`) |
| Prior integration/acceptance, concurrent disposition/correction, closed delegation, cancellation, held execution; replay keeps both identities | the same-child test (an acceptance before the correction; after it a new acceptance with a different revision is recorded, not `already-accepted`; replay); "competing and closed"; "a cancelled child…" (held parent launches nothing; cancelling the parent leaves the child) |
| Warning tails keep facts and a repair | "the refusal tail keeps the facts and a repair…" (three reminders), and the tails asserted in the CLI/HTTP and disposition tests |

Fixtures reproduce the recorded failures (a child completing with a
viewable file and no presentation; a parent with no work node; a decision
from the requester agent's answer). No historical run was changed.

## Read-only replay on real records

- Assessments (report1): presentation-missing for exactly the known
  omissions (misc run-001…004, gimmickimg run-001, run-010), none on
  consultations, studies or presented results.
- Substantive untemplated nodes across all runs: 6 runs, each with code or
  media files (rts-vs-bot implement/repair nodes, playprey tool scripts,
  misc run-002's `read-paper`/`work-2d`, lowerpoly `reproduce`); the one
  untemplated `present` node in front/converse/run-002 stays a present node.
