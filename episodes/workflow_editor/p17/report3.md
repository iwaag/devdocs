# p17 step 3 — deferred review in agent work and corrections

Plan: [plan.md](plan.md), section 3. Code: agdevworld `a3297ae` (with the
brief and continuation from `3fa432f`); agautolab `cb83634` (wfrun guide);
pj-ponderer `a8275f8` (`agent/conduct.md`).

## Guidance: shared facts once, the conductor's own fact once

- **wfrun guide** (every run's shared guide), under *Presentations, shared
  data and the run's graph*:
  - what a human review is, and that the agent's work ends where it begins;
  - that the agent's own review and early cleanup come first and are
    allowed;
  - that `dispose satisfied` is the agent's judgement and never the
    person's review;
  - how requested changes come back: a continuation, local work or the
    same child with `--follow-up`, a new review, `--no-change`, and the
    pages left serving now being the agent's.
  - It cites fix6 trial A. Usage stays in `wfe review help`.
- **ponderer `conduct.md`**: commissioned work the person can look at goes
  back to them as a human review once ponderer has judged it.
  - `ponder --next review:<id>`, then `wfe review prepare … --subject d1`
    with a Japanese summary.
  - Stopping the child's page after its own look is fine.
  - It says in the room that the work awaits their review.
  - The run completes only on their approval.
  - It cites trial A (conduct/run-037, misc/freework/run-007).
- **Front**:
  - its guides were read; they say nothing about presentations or their
    review, and delegate commissioned work to ponderer/conduct or other
    projects, so the shared wfrun guide covers its use case;
  - nothing was duplicated there.
- **The child side** needs nothing new. A child presents as fix4–fix6
  guide it. It leaves an immutable bundle (`wfe present register`), and a
  review starts it again even after the child stopped it, so a stopped page
  is no longer a lost one.

## WFE

- **The catalog**:
  - a `human-review` example (type `review`) is offered by `wfe nodes
    example` and the editor's "Add example";
  - `review` is a template type;
  - it is not seeded into existing catalogs: the live catalog gets it at
    deployment (step 6), recorded as Omni's, as fix4's `work` was.
- **`wfe conversation ponder --next review:<node>[:<name>]`** adds a review
  node branching off the ponder node. The chain that follows (an announce,
  a later ponder) does not wait for the person.
- **Brief and continuation** (`reviewBrief`):
  - a ready review node and how to prepare it, then end the attempt;
  - every round's line;
  - requested changes in full: whose words, the feedback, how to append
    work and a new review node, `--follow-up` for the same child,
    `--no-change`, and the instances left serving now being the agent's.
  - The continuation job (`review:<ws>/<wf>/<run>/<rv>`) is queued after an
    attempt and again by reconcile.
- **Follow-up of a satisfied delegation**: `wfe delegate revise …
  --follow-up <rv>` (HTTP `followUp`).
  - It requires the round to be `changes-requested` and to have reviewed
    this child.
  - It moves the `satisfied` disposition to `reopened[]`, kept with the
    round, who and when. It does not erase it.
  - The completed delegate node waits again, and the relay index row opens
    again, which used to stick `closed`.
  - Delivery and the child's `correction.receive` are fix6's.
  - Only `satisfied` is reopened, and a closed or orphaned delegation is
    refused.
  - The file layer still refuses a cancelled, executing or unreadable child
    and reopens nothing in that case.
- **Assessments** (fix6) and **reminders** (fix5):
  - `review-unrecorded` is `deferred` while a round of the parent about
    that child is undecided. It is shown as DEFERRED and is never a
    reminder.
  - A decided round counts as the person's review. A synthetic actor is
    named as "a synthetic check actor, not the person".
  - A page a round serves (or a stopped one a round will start again) is
    the review's, not a forgotten cleanup or a missing presentation.
  - The run's own `presentation-serving` skips pages its own rounds serve.
  - Review nodes have the role `review`, never `work`.
  - Without a deferred handoff nothing changes: an agent's own review stays
    `unknown` under `accepts: person`, and the reminders for actual
    omissions stay as they were.

## Checks

- `test/humanReview.test.ts`, now 8 tests. The new one covers the follow-up:
  - a judged delegation is refused without `--follow-up`;
  - with it, the disposition is kept in `reopened`, the delegate node
    waits, a new correction request is recorded, and the review
    continuation is addressed.
- `test/assessment.test.ts`, one new test:
  - without a round: `unknown` and `serving`;
  - with a pending round: `deferred`, the page is "serving now for human
    review rv1", and there are no reminders;
  - decided: ok, with a synthetic actor named as such.
- Catalog tests now expect the new example. An unknown-type fixture moved
  from `review` to `inspect`.
- Full suite: 584 tests. Two catalog tests failed on these expectations
  and pass after the changes above.

## What this step does not cover

- The person's entry points (run view, review page, conversation room) and
  the conversation's view of a prepared review (step 4).
- Whether ponderer actually follows the new fact. That is the trial in
  step 6.
