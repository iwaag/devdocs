# Workflow editor p17 — report

Plan: [plan.md](plan.md). Step reports:

Follow-up: [report7](report7.md), the Developer's waterfall session and
the chat's review list, including delegated child reviews (2026-10-25).
Extension: [ex1](ex1/report.md), approval finishes review cleanup through
the page's owner and continues the work after the review without recovery
(2026-10-25).

- [report1](report1.md): the Human review node and its durable round
- [report2](report2.md): reopenable, owned presentations
- [report3](report3.md): deferred review in agent work and corrections
- [report4](report4.md): the one-button review experience
- [report5](report5.md): verification
- [report6](report6.md): deployment, trials, the waiting example

## What the Developer can do now

Open `http://agstudio.local:8093/hangout/?conv=p17-example` and press
**レビューを開始**. That is the waiting example: misc made a golden
rectangle with the Fibonacci spiral, ponderer checked it and stopped its
page, and the work is complete and waits only for your review. The review
page (`http://agstudio.local:8093/wfe/#/ws/ponderer/review/conduct/run-039/rv1`)
starts the page, shows what to check, and records Approve or Request changes
under your name. Approval stops the page; requested changes continue
ponderer in the same room and end in a new review.

## Implemented contracts

| Plan item | Now |
| --- | --- |
| Human review as a first-class capability | node type `review`, in the editor palette, the graph, the catalog (`human-review`), `ponder --next review:<node>`, `wfe review …`, HTTP `…/reviews` (agdevworld `docs/reviews.md`) |
| Agent acceptance and human review apart | `dispose satisfied` is the agent's; `review.decide` is the person's: refused from an attempt unless it relays their words (`because`, recorded as relayed); synthetic actors flagged and shown as such; approval permits nothing and never queues integration |
| Work complete, awaiting review | the review node waits on its round; status "Work complete · Awaiting human review" (situation `awaiting-review`); no process, no job, no hold, never stalled; compiler v5 keeps the final report off review nodes |
| One Start review | records the request, starts every frozen subject through its owning run (completed children included; a cancelled one never), verifies identity, the screen and the entry, reuses what serves, records what it found; partial failure is `start-failed` with reasons, retryable |
| Frozen material | subjects by owning run (creation), presentation and manifest revision, the delegated result and repository trees; worktree edits never reach them; bundles rechecked at prepare, start and approval; newer presentations shown beside, never substituted; approval names the revisions (`review-superseded`) |
| Approve and cleanup | decision recorded once; cleanup through each owning run (`presentation.observe stopped`), verified; failures retried, never re-asked; instances another active round uses or kept on request are kept; earlier rounds' handed instances stopped with the answering approval |
| Request changes | feedback recorded, round closed, node completed, run stays `in-progress`/`due`; one continuation (`review:<ws>/<wf>/<run>/<rv>`), queued at once and by reconcile; addressed by appended work, a reopened node, record-done, `delegate revise --follow-up`, a new round, or `--no-change`; `missed-review` hold otherwise; the rejected round stays |
| Same child after satisfaction | `wfe delegate revise --follow-up <rv>` reopens a `satisfied` delegation (kept in `reopened`) for the same child; cancelled or executing children refused; the relay index reopens |
| Ownership and restarts | `present.sqlite` v2 `usages` (started or reused, active or released) and `review_work` (one host operation per round at a time); reconcile finishes a start left `starting` and retries owed cleanups; run cancellation cleans up its rounds |
| Assessments and reminders | a pending round is `deferred`, not unknown; a page a round serves or will serve is not a forgotten cleanup; without a handoff nothing is hidden |
| Entry points | WFE review page, run-view focus card and node cards, aghangout room line and button, Zulip messages with the page, conversation API `reviews`; links resolved from the host each time |
| Guidance | wfrun guide (shared facts), ponderer `conduct.md` (its own fact, plus the trial B gap), command help |

## Validation

- WFE `npm run check` passes: 586/586 tests (25 new across reducer, real
  presentation servers, the end-to-end flow through executor, notifier and
  intake, assessment), tsc and both builds. aghangout `npm run check`
  passes, 31/31.
- All 355 run records on this host read under the new release; no
  migration was needed.
- Deployed versions were verified on the coordinator and on agpc. The
  Nautobot placement was updated through nctl, and agpc is converged.

## Deployed trials and autonomous outcomes

- **Trial A** (check actor `p17-check`, conversation `p17-trial-a`,
  ponderer/conduct/run-038 → misc/freework/run-008). Between the actor's
  three clicks (start, request changes, approve, all via the real room and
  review page), the agents did everything themselves:
  - ponderer announced the deferred review, judged the figure, prepared rv1
    and ended its attempt normally;
  - after the change request it reopened the same child with
    `--follow-up`, got a corrected page and prepared rv2;
  - the approval stopped both rounds' instances, and the run completed.
  - The page answered from agpc (another LAN host).
- **Trial B** (the waiting example). ponderer first relayed the child's
  confirmation question live to the room, which is the fix6 pattern: the
  child and its page waited.
  - The conduct guide was fixed for this (`4b0bdb9`).
  - Omni redirected run-039 as the room's requester (#372). That step was
    controlled, not autonomous.
  - Then ponderer answered the child, the child stopped its page, and
    ponderer prepared rv1 for the Developer.

## Limitations

- One real trial per case. Trial B shows the conductor did not yet treat a
  child's confirmation question as part of its own review. The guide now
  says so, but the fix is not yet tested by a new trial.
- aghangout shows the review as a status line and a button, and the
  decision happens on WFE's page. The room's header badge (作業中 n件)
  still counts a run awaiting review as work.
- A reused instance keeps the answer line it was started with (only
  instances a review starts point to the review page).
- Integration of a corrected child after a follow-up is fix6's path and
  was not re-run here.
- The room's Start review posts as the area's person, like its messages.
  The trial re-attributed it to the check actor in the browser.

## Recovery references

The ignored `pj-agdev/.local/p17/notes.md` records:
- the state before the release;
- the backup (all stores; the present store's v1 → v2 is refused by
  `ab342dd`);
- the activate commands;
- how to find records using `review.*`;
- the agpc pin rollback, and the aghangout, guide and catalog rollbacks;
- the trial identifiers.

`pj-agdev/.local/devenv.md` has a p17 section, and
`pj-clusterintent/nctl/.local/p17/` has the batch and playbook logs.
`devdocs/README_DEV.md` has the p17 entry. Historical runs were not
rewritten.
