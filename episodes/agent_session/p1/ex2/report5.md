# Agent session p1 ex2 — step 5: phase boundaries (design only)

Date: 2026-10-15. No experiment was run for this step.

## The limitation

Neither p1 nor this follow-up tested the person's normal practice: a planning
session, then a fresh implementation session that receives the approved
plan. Both compared only the two extremes:

- `fresh`: a new conversation every attempt;
- `run`: one conversation for the whole run.

The `run-optimized` arm in step 4 kept a single session from planning to
acceptance. Nothing here measures a split policy, and none of step 4's
figures is a saving from splitting.

## What the optimized trial's phases suggest

These figures come from ex2-run/run-001 (step 4, one session, four attempts):

| Phase (attempt) | Context at start → end | Steps | Cached tokens read | Reported |
| --- | --- | --- | --- | --- |
| Planning (1, new) | 42.7 k → 56.1 k | 11 | 0.52 M | $0.29 |
| Implementation → presentation (2, resumed) | 57.4 k → 103.5 k | 40 | 3.36 M | $1.03 |
| Correction loop (3, resumed) | 104.8 k → 144.3 k | 37 | 4.74 M | $1.31 |
| Acceptance, cleanup, final report (4, resumed) | 145.8 k → 151.2 k | 11 | 1.62 M | $0.39 |

- Planning added about 13.4 k tokens of conversation: reading the request,
  the code survey, `plan.md` and the question. Every later step carried
  that 13.4 k again: 88 steps, about 1.2 M cached tokens, roughly 12 % of
  the run's 10.2 M.
- Implementation and its correction share most of their useful context:
  the files changed, the tests and the presentation. The retained
  correction attempt took the answer up, recorded the graph adjustment, and
  re-read only the parts of `main.js` it was about to change (3.4 k chars).
  It did not reread the plan, the definition or the history. p1's fresh
  correction attempt first read the 8.6 k-char history, the definition
  twice, `plan.md` and the implementation report.
- So the candidate boundary is plan → implementation. Implementation, its
  short correction loop and the close-out stay together.

As an **estimate, not a measurement**, splitting at the plan boundary would:

- remove ≈13 k tokens from each later step;
- in return, pay one new-session bootstrap (≈17–22 k tokens written, as in
  the fresh arm) plus a handoff;
- cut cached reads by ≈1 M on a run shaped like step 4's.

The bootstrap write is paid once and the carried planning history on every
step, so the split favours long implementation phases. A wait of more than
an hour before implementation starts gives a split session nothing to
lose: it starts cold anyway. The planning conversation itself would have
been rewritten to cache in full.

## The handoff

A new implementation session must not depend on `plan.md` alone. A plan
can fail to restate a constraint the person gave in the planning talk. In
p1, a fresh session dropped the unit count after a correction that never
mentioned it. The handoff is the new session's whole brief (step 2 already
names the input and the agreed plan) plus a section that names everything
agreed so far:

1. the original input (`braindump.md` / `request.md`, and
   `original-input.md`), by reference and hash;
2. the agreed `plan.md` revision: the question it was agreed in, and the
   document hash recorded with that question;
3. **every answer taken up and every instruction acknowledged up to the
   boundary, verbatim**, with ids and authors. This holds the constraints
   the plan may not repeat. It is bounded by the planning talk's size;
4. artifact references: reports and presentations written so far;
5. the current state: the compact state and pending input of step 2.

Items 1, 2, 4 and 5 already exist. Item 3 is new: a "what was agreed"
list, built from `questions[*].takenUp` and `instructions[*].ack` up to the
boundary seq. The full transcripts stay in the records for a targeted
reread.

## Smallest policy and UI

- **Definition**: `execution.session` gains one value, `phase`. It means
  `run`, except that the first attempt claimed after the plan step
  (`_plan`) completes starts a new session. Any later attempt resumes
  that new session.
  - A workflow without a plan step treats `phase` like `run`, and the
    editor's validation warns about it.
  - The value is frozen with the run and covered by the definition digest,
    like the other two.
- **Claim**: `sessionFor` already replaces a session with a recorded
  reason. The new reason is `phase boundary: _plan completed (agreed in
  q<n>)`. The new session gets the whole bootstrap plus the handoff
  section (item 3).
  - Retirement, configuration change, missing store and mismatch keep
    their current rules.
  - A boundary that falls inside one attempt is crossed at the next claim.
    An agent that agrees the plan and starts implementing within the same
    attempt keeps the planning session until that attempt ends.
- **UI**: Detail → Workflow settings → Execution → Session offers three
  choices:
  - New conversation every attempt (`fresh`, the default);
  - One conversation for the run (`run`);
  - **New conversation after the plan is agreed** (`phase`).

  The run view's session panel already lists sessions with their
  replacement reasons, so the boundary is visible there without new UI.
- **Observer, records, fencing**: unchanged. A phase boundary is one more
  recorded replacement.

Nothing in this design was implemented. `run-optimized` carries no hidden
split.

## Evidence needed before adopting it

- Matched runs on at least two tasks of different shape:
  - one with a real correction loop;
  - one with a long implementation;
  - one in which the person gives a constraint in the planning talk that
    the plan omits on purpose. This tests the handoff for silent loss.
- Three arms: `fresh`, `run` (optimized) and `phase`, with the same
  scripted inputs recognised by meaning.
- Measured per phase:
  - first-request context;
  - cached reads and writes;
  - steps;
  - reported cost;
  - rereads at the boundary;
  - retention of every planning-talk constraint through implementation
    and correction.
- One condition with a real delay (more than an hour) before
  implementation. It needs no special wait if a natural delay occurs, and
  shows whether `phase` merely ties `fresh` there.
- Accept a split only if it keeps quality and lowers whole-run cost across
  the tasks. Removing the planning history from later steps alone does
  not justify it.
