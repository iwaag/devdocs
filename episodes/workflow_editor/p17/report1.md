# p17 step 1 — the Human review node and its durable round

Plan: [plan.md](plan.md), section 1. Code: agdevworld `c70852e`
(`workflow_editor/shared/humanReview.ts`, `shared/run.ts`,
`shared/compile.ts`, `shared/status.ts`, `shared/notify.ts`); contract:
`workflow_editor/docs/reviews.md`.

## What was traced first

- **Reducer** (`shared/run.ts`): waits are `{question}`, `{external}` or
  `{delegation}`; completion, `situation()`, `dueReason()`, holds at
  `attempt.end` and fix6's correction continuation (an unaddressed
  correction keeps a completed execution `in-progress`) are the patterns
  reused.
- **Compiler** (`shared/compile.ts`, then v4): a report step after every
  authored node and a final report after every terminal node. A review
  after the work would have held `_final-report`, so the agent could not
  finish its work before the person's review.
- **Scheduling, status, observation** (`server/exec.ts`,
  `shared/status.ts`, `server/observe.ts`, `server/continuation.ts`): every
  place that branches on the wait kind was listed. A new kind would have
  fallen through to the delegation or external text.
- **Presentations** (`server/present.ts`): `Presenter.start` refuses a
  completed run ("its presentations are not served again"). That is step
  2's main obstacle.
- **Delegation** (`server/delegate.ts`, `shared/delegation.ts`): `revise`
  refuses a disposed delegation, and the relay index row of a judged
  delegation is sticky `closed`. Both matter for step 3.
- **Conversations** (`server/conversationService.ts` `view()`, aghangout
  `src/room.ts`): where step 4's entry point goes.

## Representation decided

- **A dedicated node type, `review`.** Existing types are performed by an
  agent: a talk asks and waits for an agent to take up the answer, and a
  do node is the agent's work. Once prepared, a human review's waiting,
  start, decision and cleanup are WFE's records, so they cannot be
  truthful on those types. It is not a prompt convention.
- **Compiler `ag.wfe-compile.v5`.** A review node gets no report step and
  does not precede the final report. The final report follows the nodes
  whose successors are all reviews. Graphs without review nodes compile as
  under v4, so existing runs are unaffected; v4 records are followed as
  recorded.
- **A round** (`run.json` `reviews.<rvN>`) records:
  - owning run and node;
  - prepared-by actor (with attempt);
  - summary;
  - responsible agent and reviewer;
  - relation to the previous round (`previous`: the round whose requested
    changes it answers);
  - frozen subjects. Each subject has the workspace, run identity with
    creation, presentation id, manifest revision, and the delegated result
    and repository trees it represents. The file layer fills these in at
    step 2/3.
- **Operations.**
  - `review.prepare` and `review.take-up` belong to the agent and are
    fenced by its attempt.
  - `review.start` and `review.decide` belong to the person.
  - `review.serve` and `review.cleanup` are host facts.
- **Waits.** The review node waits on `{review: rvN}`. The situation is
  `awaiting-review`, and the status reads **Work complete · Awaiting human
  review**. It is never the executor's move.

## The states as implemented

| State | Next |
| --- | --- |
| awaiting | the person starts it; the attempt that prepared it ended normally (no hold) |
| starting | the host records `review.serve` for that start; a second start is refused (`review-starting`), and a result for an older start is refused (`review-stale`) |
| reviewing / start-failed | approval needs `reviewing`; `start-failed` names each subject that did not start and why, and is never dropped from an approval |
| approved | decided once; cleanup pending/failed; a failed stop is retried and never re-asks |
| complete | cleanup verified; the node completes, then the run (one `completed` event) |
| changes-requested | the node completes with the feedback; the run stays `in-progress`/`due` until work is appended (adjust adding nodes, a reopened node, record-done, `delegation.revise`, a new round) or `--no-change`; an attempt leaving it unaddressed is held `missed-review` |
| cancelled | run cancellation closes undecided and approved rounds; started instances stay owned by WFE's cleanup |

Agent acceptance, the person's answer, the review outcome and cleanup are
separate records:

- `review.decide` from an attempt is refused (`review-agent`) unless it
  relays the person's words with `because`. A relayed decision records the
  person as `by` and the agent as `relayed.by`.
- A synthetic actor is recorded with `synthetic: true`.
- Approval does not touch integration or any permission.

## Checks

- `test/humanReview.test.ts`, 7 tests:
  - compile v5;
  - the prepared state and its status;
  - preparation rules;
  - start/serve/approve/cleanup with partial failure, double click, stale
    start, a superseded revision and a failed stop;
  - agent versus relayed decisions;
  - requested changes: closure, continuation, the `missed-review` hold,
    addressing by appended work, the next round's `previous`, old round
    immutable;
  - cancellation cleanup ownership.
- The full WFE suite ran with 578 tests. Two failed:
  - `contract.test.ts` used `review` as its example of an unknown type, so
    the example was changed;
  - `sqlite.test.ts` has a lock-timing assertion that failed under load
    and passed on two reruns.
- Compiler version assertions in two tests now name v5.

## Open for later steps

- File layer: subject resolution and freezing, host start/stop (step 2).
- Agent integration: guides, the catalog template, the brief, assessment,
  follow-up of a satisfied delegation (step 3).
- UI and conversation entry points (step 4).
