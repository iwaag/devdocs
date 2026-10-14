# project_migration p3 ex1 — step 1: one approval path

Written 2026-10-14. agdevworld `e1e6be3` (not yet released; the release is step 3).

## What was wrong, traced on p3's records

- **"Accepted" while approval was still needed.** The header's decision chip read only the latest decision. In p3, runs 001–003 were accepted from the confirmation answer, then `report.md` was written after that. Integration stopped with `needs-agreement`, and the header still said *Accepted*. The renewed acceptance was in *All records → Result and decision*, with a required evidence field.
- **"No response" and the second answer.** `provide/run-002`'s history shows the Developer's acceptance at 11:05:59 and a second one at 11:06:26. The integration ran at 11:06:35, in the executor's next 60-second pass. Between the click and that pass, the only feedback was a banner at the top of the page. The status beside the control did not change, and *Accept result* stayed enabled. The second click recorded another acceptance of the same result. Answers were the same: success or failure appeared only in that banner.
- **A stale status.** `integrate --status` listed the journal oldest first, so its first line was decision 0's `needs-agreement`. In `provide/run-002`, the executor's queue was working from a list of rows it had read earlier. Row 4 (decision 1) integrated decision 2, which was the latest. The stale entry for row 5 was then refused as "already integrated", which **overwrote that row's state**. The journal row now contradicts `run.json`.

## Changes (agdevworld `workflow_editor/`)

| Area | Change |
| --- | --- |
| `shared/approval.ts` (new) | One reading of the result's standing with the person: *review needed*, *answer recorded · waiting for <agent>*, *decision needed*, *changed since acceptance — accept again*, *integrating*, *integrated*, *not integrated*, *rejected*. It is always the **latest decision's** standing. `run.json`'s integration record outranks a journal row. The header, the focus card, the CLI and the tests all use it (`src/integrationState.ts` re-exports it). |
| Run view | The header chip comes from that state. A changed result gets its own focus card: what happened, the files changed since the acceptance (as links) and *Accept the changed result* next to them. Evidence is optional for a renewed acceptance, and the record names the changed files. *Accept* is offered only when it would record something new. *Retry integration* is not offered for a changed result. Answering and accepting stay separate: while a review question is open or its answer has not been taken up, the answer is the only path. |
| Feedback | Every form says what happened next to its button. *Recording…* shows while the request is in flight, with the button disabled, so a second click sends nothing. Then it shows what was recorded, or why nothing was, and keeps the text. A recorded answer on its question reads *Recorded … Waiting for <agent> to take it up*, in a different style from a failure. A missing name says so in the form. |
| Duplicate protection | Server-side, under the run's lock. An acceptance of the result the latest acceptance already binds is refused with `already-accepted` (409), whether it is direct (browser or CLI) or comes from a confirmation answer. "Same" is compared the way integration compares: the run's `run.json` and owned gitlinks are set aside. Nothing is recorded and nothing is queued. A changed result, a rejection or a pre-p9 acceptance can still be accepted. A retransmission with the same receipt is still a duplicate. A repeat from the same question keeps `decision-exists`. Historical decisions are unchanged. |
| Executor queue | Each queued row is read again before it is acted on. A row for an earlier decision becomes `superseded`. So does a row whose result an earlier, integrated acceptance already brought in. |
| `wfe run integrate --status` | The first line is `Now: …`, with the decision (and the changed files, if any). The journal follows, newest first. It marks rows of earlier decisions and rows that `run.json` contradicts. |
| Docs | `docs/integration.md` (one acceptance per result, status, header chip wording) and `docs/runs.md` (acceptance, feedback). |

## Checks

- `test/approvalstate.test.ts` covers the plan's regressions 1, 3 and 4, plus the waiting-for-agent state:
  - renewed acceptance is shown with its control and files, then integrated;
  - a same-result acceptance is refused after a confirmation, before and after integration, with no second decision or row, while a changed result is accepted again;
  - two queued acceptances end as `superseded` + `integrated`, and `--status` prints `Now: Integrated…` first.
- `checks/approval.ts` is a browser check (regression 2 plus the UI side of 1 and 3). It uses a private project, registry, execution store and service, in Simple. A simulated 500 shows the reason next to the button and keeps the text. A slow request shows *Recording…* with the button disabled, and a forced second click reaches the service 0 times. Success shows *Recorded … waiting for autolab*, and the header says the same. The renewed-acceptance focus card lists `report.md`. A double click records one decision and queues one integration. After that no Accept is offered, and after the executor's pass the header reads *Accepted · integrated*. Result: 17/17 checks passed.
- `npm run check`: type check, **311/311 tests**, build. One existing test (`conversation.test.ts`) first failed: a repeated confirmation decision was refused by the new check before the reducer's more specific `decision-exists`. The fix keeps `decision-exists` for the same question.
- `checks/displayMode.ts` expected "No decision"; it now expects "Decision needed". It is not run here: it needs the fixture service.
- Read-only on the real forge records: `--status` for `provide/run-002` now starts `Now: Integrated into main (decision 2…)` and flags the row that contradicts `run.json`; run-001 and develop-capability/run-001 read correctly.

## Not changed

- Journal rows and decisions already recorded (the contradicting run-002 row is annotated, not rewritten).
- Integration still runs in the executor's 60-second pass; the feedback says so ("within about a minute").
- "Record another answer" on an answered question stays: it is a correction before take-up, not an acceptance.
