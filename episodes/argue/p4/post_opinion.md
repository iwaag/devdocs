# Argue p4 — post-trial assessment and improvement proposals

This is the Omni Agent's assessment of [report.md](report.md) and
[report6.md](report6.md), written after the Developer requested improvement
proposals. It is not a report of additional implementation or a new trial.

## Assessment

P4 established a useful separation of responsibilities and made discussion
progress observable. Front selected participants, ponderer chose questions
and proposals, and jevlab delegated execution to an ordinary workflow. The
activity history explains how the discussion reached its result.

The main unresolved issue is the distinction between obtaining an acceptable
task result and completing the agreed process. Ponderer explicitly planned
a review in activity 32, but activity 34 treated the apparent achievement
of the goal as sufficient to conclude. No review or summary followed.

That is stronger evidence than a generic complaint that the agent forgot
instructions: it formed the intention and then failed to carry it forward.
The delivered guide already said to carry owed follow-ups as planned nodes.
Adding the same sentence more emphatically is unlikely to be the most useful
next experiment.

The proposals below improve visibility and reliable recording while leaving
progression decisions with the agent. They do not prescribe mandatory debate
rounds, require unanimous agreement or introduce human approval gates.

## Priority 1 — preserve agreed work beyond the immediate next action

### Evidence

- P4 activity 32 said "then review the result".
- Activity 34 said nothing further was missing and selected a conclusion.
- Ex1 needed an Omni reminder to move from execution into the already-agreed
  review, and another to preserve a summary.

### Proposal

Represent explicit remaining commitments where the conductor reads its work
state. Prefer existing future nodes and run revision history over a parallel
task-management subsystem. Review should become a visible planned activity
when the agent commits to it, even if its detailed questions are not known
until the result arrives.

Distinguish two sets of facts:

- **Execution obligations:** outstanding invitations, questions, delegations
  and runs. WFE already derives these from operational records.
- **Agreed work:** review, a requested summary, an explanation to the requester
  or another follow-up that may not yet have a dispatched invitation.

The absence of running work does not mean that no agreed work remains.
Expose both in the conductor's normal state/context and conclusion input.
The agent should be able to complete, revise or abandon planned work with
an explicit reason and source reference. Such a revision is a decision to
record, not something that needs permission.

First inspect the actual graph revisions around activities 32–34. Establish
whether review was never materialized as a node, was removed, or existed
but was ignored. That determines the smallest useful change; the prose
record alone does not settle which mechanism failed.

Do not infer every passing suggestion in a conversation into a binding
commitment. The agent records what it takes on and its source. For the next
trial, test whether it does this itself after being given the capability
and concise usage facts.

### Verification

A result meeting the product goal must not make a previously agreed review
disappear silently. Inspect the node or commitment history, the final
decision and any explicit revision. A task with no requested review must
still be able to conclude without inventing one.

## Priority 2 — ground the final account in identifiable observations

### Evidence

- P4's "Still owed: none" contradicted the obligations snapshot captured
  by its own conclusion operation.
- Its branch statement became stale when integration completed one second
  before the conclusion was posted.
- Ex1's "not on main (checked)" claims were wrong because published state
  and local workspace state were apparently confused; those claims entered
  the preserved summary.

### Proposal

Attach a small authoritative state section to the conclusion view using
the snapshot already recorded by WFE: outstanding work, owners, recipients
and observation time. Keep agent interpretation alongside it, separately
attributed. This makes an incorrect prose claim visible without requiring
another agent to repeat the state lookup.

For artifacts, use the existing `--integration` observation and cite the
published revision and observation time. Keep these statements separate:

- files exist in a run copy;
- the run has an integration event;
- the integrated commit is reachable from the published target branch;
- a local workspace has or has not caught up.

A statement about a changing branch should describe what was observed at
a time, not promise that the branch can no longer change. A one-second race
is different from failing to inspect the available evidence. Repeated
polling cannot guarantee that a prose statement remains current forever.

The product can display the recorded snapshot and a current-state link.
No model-generated status sentence should become a second operational
authority. Do not add a generic natural-language contradiction detector as
a prerequisite; the available structured evidence is sufficient to start.

### Verification

Use a fixture in which integration happens near conclusion. The recorded
observation must remain true as an observation, and the UI must let a reader
distinguish it from current state. A nonempty obligations snapshot must be
visible even if the agent incorrectly writes "none".

## Priority 3 — handle ready-node leftovers without masking unfinished work

### Evidence

In p4, jevlab completed the delegate node it had added but left its original
talk node ready. Automatic recovery completed the run. Related attempt-end
problems appeared in both p3 and ex1.

### Proposal

Inspect the graph-adjustment and completion command path used in the trial.
Make it easy for an agent to replace its original talk activity with
delegated work while recording what happened to that original node in the
same revision. Existing atomic revision/completion operations should be
reused where they fit.

At attempt end, report the specific ready or running nodes and their
recorded relationship to delivered speech. Give recovery enough evidence
to distinguish bookkeeping from actual remaining work.

Do not mark every leftover node complete merely because the run posted a
result: a ready verification node may represent genuinely unperformed work.
Keep automatic recovery in the evidence and count its assistance separately
from independent completion by the participant.

### Verification

Cover both cases: a superseded talk node with a recorded disposition, and
a real verification node that remains to be executed. Neither case should
produce duplicate speech, and the latter must not be silently discarded.

## Priority 4 — preserve conclusions with source quality intact

### Evidence

P4's transcript and evaluator analysis are available, but no agent-authored
summary was requested. Ex1 produced a summary after a hint, yet that summary
preserved incorrect integration claims.

### Proposal

When a discussion explicitly includes durable synthesis, carry it as agreed
work under Priority 1 and commission argue's existing synthesis workflow.
Do not require knowledge promotion for every casual discussion.

A synthesis should distinguish:

- original participant claims;
- records or artifacts inspected to check them;
- conclusions supported by those checks;
- unresolved or contradicted claims;
- later evaluator corrections.

Prefer source-linked revisions or correction annotations to rewriting the
original transcript. Existing erroneous summaries can acquire an explicitly
dated correction without making the historical trial appear more successful.

For operational facts, synthesize from the relevant observations as well as
speech. Agreement among participants is not independent evidence when they
all rely on the same report or stale checkout.

## Next experiment

First isolate the most consequential change: carrying agreed work into the
conductor's visible remaining work. Repair deterministic bookkeeping issues
with focused fixtures, but avoid changing the model, roles, guide structure
and task all at once if the aim is to understand the effect.

Suggested substantive trial:

1. Give a bounded goal and explicitly include report review and a preserved
   account in the initial request. Keep the technical solution unspecified.
2. Let Front select participants and ponderer plan progression.
3. Provide no intermediate hints. Leave prior artifacts available, recording
   reuse rather than pretending the trial is independent of them.
4. Observe whether review and synthesis become explicit remaining work,
   survive the execution result, and are completed or deliberately revised.
5. Check the final account against captured obligations and published-state
   observations. Record automatic recovery and any intervention separately.

Use a separate small fixture with no review requirement to detect whether
the improvement has become an indiscriminate ritual. Other fixtures can
exercise a failed child, a late result and a conclusion while work remains.

Measure more than apparent success:

- Was promised work represented before the result arrived?
- Did the conductor explain a change of plan rather than silently omit work?
- Did specialist work return to the right owner?
- Did speech, workflow state and published artifact claims agree at their
  stated observation times?
- How many hints, recoveries and cleanup operations were needed?
- What information changed the conductor's next action?

If the same omission recurs despite visible planned work, that is useful
evidence. Compare the next decision's inputs and outputs before adding more
rules. Subsequent model/profile comparisons should keep the task and
available evidence as stable as practical and repeat trials; one run per
condition cannot rank workflows or models.

## What not to prioritize yet

- A universal debate state machine or mandatory review approval.
- A new generic node/plugin framework: p4's activity metadata already
  expresses the required operations.
- More participants or higher model capacity merely to compensate for
  missing completion bookkeeping.
- Large-scale automated scoring before the observations and provenance are
  reliable.
- Improving RTS/Jev performance as a proxy for improving argue. Product
  quality and coordination quality remain separate outcomes.

## Recommended order

1. Inspect the lost-review graph history and implement the smallest useful
   representation of agreed remaining work.
2. Present conclusion snapshots and timestamped integration observations
   clearly, reusing the existing authoritative records.
3. Resolve the demonstrated graph-adjustment/leftover-node case with focused
   verification, without auto-completing real work.
4. Run one no-hint trial and preserve its successes and failures, followed
   by a source-linked synthesis and external analysis.

The next useful milestone is not a more elaborate meeting. It is a
discussion that can account accurately for what it agreed to do, what it
actually did, and what it deliberately leaves unresolved.
