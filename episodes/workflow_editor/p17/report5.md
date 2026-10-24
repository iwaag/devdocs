# p17 step 5 — verification

Plan: [plan.md](plan.md), section 5. Code: agdevworld `4fd38ce`. Tests:
`test/humanReview.test.ts` (reducer, 8), `test/reviewHost.test.ts` (real
presentation servers, 5), `test/reviewFlow.test.ts` (end to end through
executor, notifier and intake, 1), `test/assessment.test.ts` (+1).

One fix came out of this step. Requested changes now queue the responsible
agent's continuation as soon as they are recorded (`ReviewHost` →
`ExecStore.afterReview`). Before, only the next reconcile or attempt end
queued it.

## The plan's checks and where each is shown

| Check | Shown by | Result |
| --- | --- | --- |
| A parent accepts child work, stops the page, prepares deferred review, frees its slot; the review stays discoverable | reviewFlow: `dispose satisfied` by the agent, the child's page stopped, `prepare`, attempt end with no hold, no job, situation `awaiting-review`, status "Work complete · Awaiting human review" with `job: null` (not stalled), reconcile queues nothing; conversation and run views carry it (step 4) | pass |
| One Start review serves the exact frozen revision of a completed child; worktree edits do not change it | reviewHost test 1: `content/main.js` served as v1 after the worktree changed to v2 | pass |
| Multiple child subjects | reviewHost test 5: two children in one round, both live, both stopped on approval | pass |
| Remote worker publication and retention | the bundle is registered on the coordinator by the forwarded command (step 2); exercised on the deployed host in step 6 | step 6 |
| Partial startup, unavailable/corrupt bundles | reviewHost test 2: `start-failed` names the hash refusal, no instance, no approval; repaired, the same round starts; refused at prepare | pass |
| Double clicks, restarts, no duplicate processes | reviewHost tests 1 and 3: a second start reuses the instance (one serving row); a start left `starting` is finished by reconcile, reusing the other round's instance | pass |
| No false approval | reducer: Start review is not approval; approval needs `reviewing`; agent decisions are refused (`review-agent`); HTTP refuses `review.*` on the generic ops route (reviewHost test 5) | pass |
| Approval binds once; cleanup failure retries without asking | reducer: second decision refused; `stop-failed` leaves the decision and the state approved, the next cleanup completes it | pass |
| Concurrent review/retention owners | reviewHost test 3 (another active round: `kept`), test 4 (kept on request, with its owner) | pass |
| Feedback closes the round, one continuation, extends the workflow, distinct review of corrected output | reviewFlow: round closed, run `in-progress`, exactly one `review:` job (still one after reconcile), brief with the feedback; appended review node, new round of `p2` with `previous: rv1` | pass |
| Already-satisfied delegation follow-up, prior integration, repeated delivery after crashes | reviewFlow: `revise --follow-up rv1` reopens the satisfied delegation and keeps the disposition; a prior acceptance of the child stays a historical fact; the correction's delivery loses its acknowledgment once and is posted once, applied once | pass (acceptance, not integration: integration keeps its own fix6-tested rules) |
| Old-browser actions cannot approve superseded rounds | reducer, reviewFlow (`rv1` refused after `rv2`), HTTP `review-superseded` for other revisions | pass |
| Agent acceptance distinguishable; synthetic answers labelled | reducer and assessment: `dispose` versus `review.decide`; `synthetic: true` shown as "a synthetic check actor, not the person" | pass |
| Cancellation and abandonment leave explicit cleanup ownership | reducer: cancelled round with `cleanup.owner` WFE (cancellation cleanup); reviewHost test 5: `ExecStore.cancel` stops the review's instance and records `done`; handed instances after changes are the agent's and stopped by the next approval (reviewFlow) | pass |
| Human waiting causes no executor churn or false stalled-work incidents | reviewFlow (no job; reconcile adds none; status not stalled); observation reports the person (or `wfe:review` while host work is owed) as responsible | pass |
| CLI, API, editor and conversation entry points agree | reviewHost test 5: CLI prepare/start/approve and HTTP view/decide on one round; `--by` required for a person's act | pass (browser views: step 6) |
| Report boundaries recognise deferred review without hiding missing handoffs | assessment test: without a round still `unknown`/`serving`; with one `deferred` and no reminders | pass |
| Existing ordinary presentations keep working | full suite (present, correction, reminders tests unchanged) | pass |

## Whole-suite and host checks

- WFE `npm run check` passes: 586/586, tsc, both builds.
- aghangout `npm run check` passes (step 4).
- All 355 run records of the registered projects replay under the new
  reducer and compiler, read-only. Every one reads; the deployed release
  also reads 355/355. No migration is needed: the new fields are additive.
  v4 graphs are followed as recorded.

## Limits of these checks

- Delivery through the real Zulip transport and a remote worker are not
  covered here. They are checked on the deployment (step 6).
- Integration of the corrected child after a follow-up is fix6's
  correction path. It is not exercised again here.
