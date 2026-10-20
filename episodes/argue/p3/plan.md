# Argue p3 — one execution committee trial

## Objective

Test the workflow-driven argue system described in
[braindump.md](braindump.md) and delivered in
[workflow_editor p16](../../workflow_editor/p16/report.md).

Run one discussion with Front, the RTS project and Jev lab: develop a
direction, let the projects execute their work, and return to the same
discussion to examine their reports. End after that examination, regardless
of the quality of the result. Do not start another planning/execution cycle.

The subject is argue's ability to connect distinct project contexts,
execution and shared result review. A weak game result, an unsuccessful
experiment or an observed coordination stall is a valid experimental outcome.
Do not rescue agent coordination to manufacture success.

This document plans the trial; creating it does not start implementation or
send messages to agents.

## Working conventions

- Follow `localrule.md`: generated files are English; communication with the
  Developer is Japanese. Implementation, deployment, verification and
  appropriate commits/pushes need no additional human approval checkpoints.
  Approval and acceptance records are not permissions.
- Before inspecting or changing deployments, obtain current state through
  Nautobot/nctl, following `pj-clusterintent/nctl/README.md`. Previous reports
  and ignored environment notes are pointers, not current service evidence.
- Deploy necessary support and verify it from another LAN machine. Prepare
  direct discussion, run, archive and result links for minimal manual review.
- Keep machine-specific details, credentials and raw operational evidence in
  ignored files. Preserve unrelated edits, records and active work.
- Backward compatibility is unnecessary; account for active delivery when
  updating contracts or services.
- Keep tool usage in help and shared facts in shared guides. Repair concrete
  capability gaps without teaching agents this experiment's solution.
- Identify automated checks and Omni interventions honestly. Record work done
  for an in-system agent with the handoff note required by README_DEV.

## Participants and initial request

The braindump's project names resolve, based on the inspected project
definitions, as follows. Verify these identities before starting:

| Braindump name | Managed workspace | Role |
| --- | --- | --- |
| front | `front` | Participant with cross-project coordination context |
| pj-rts | `rts-vs-bot` | Participant owning the RTS game |
| pj-javlab | `jevlab` | Participant owning Jev knowledge and experiments |
| Existing argue facilitator | `argue` | Facilitate discussion and its return from execution; preserve records |

Use the existing argue facilitator alongside the three named projects.
Front remains a participant; it does not silently replace the facilitator.
Do not preassign implementation responsibilities beyond project identity.

Submit the Developer's goal verbatim from the braindump, preserving its
wording and attribution. The goal is one experiment in which Jev is used
at least to some extent in AI-versus-AI RTS play; quality is not a requirement.
The source file remains authoritative for the exact Japanese text.

The Developer's later addition supplies the trial protocol: discuss,
execute, return to the same argue, examine reports only, then finish without
replanning or further execution. Record this separately from the task goal,
as an explicit instruction for this trial. It is not a permanent restriction
on argue.

Provide no model selection, game design, Jev integration strategy, task
decomposition, quality threshold or extra acceptance rubric. Agents choose
how to interpret and pursue the goal using their existing context. The
Developer will not provide intermediate answers or steering.

## Experimental boundary

### Necessary system support

The following are legitimate preparation or repair:

- Participant workflows and registration for all three projects.
- Guides that correctly describe discussion participation, attribution,
  available tools and existing project responsibilities.
- Access to existing project knowledge, workflow execution and ordinary
  infrastructure/configuration tools.
- Durable work-to-discussion links, result delivery and continuation so
  execution can lead back to review in the same discussion.
- Repairs to demonstrated delivery, permissions, persistence, scheduling
  or other system capability defects.

Before changing an alleged defect, record the attempted operation, observed
failure and why the existing capability cannot fulfill its documented
contract. Fix the smallest relevant mechanism, verify it and resume the
affected operation without duplicating the experiment. Record each such
intervention and its possible effect on the trial.

Installing or developing the experiment's own game/model integration is
project work for the participants. The mere absence of a ready-made solution
is not a system defect for Omni to solve.

### Results to observe without rescue

Examples include agents overlooking available tools, failing to delegate,
misunderstanding each other, making poor technical choices, repeatedly
discussing without moving forward, or waiting for a human choice despite
having the available authority and context.

Do not inject a plan, answer their substantive questions, rewrite their
implementation, or add trial-specific guidance to correct such behavior.
Do not change models or restart the trial merely to improve the result.
If the evidence establishes a coordination stall, report it and end the
trial. A successful round trip is also a stopping outcome.

Queueing, slow execution and a valid outstanding callback are not by
themselves stalls. Determine the state from run records, waits, delivery
evidence and progress over time. Record observation times and explain any
stall judgment. If the cause remains uncertain, report that uncertainty
rather than claiming either an agent failure or a proven system defect.

## Execution and review contract

- Keep one stable discussion ID from the initial goal through final review.
  Project implementation still belongs in its ordinary workflow runs.
- Preserve participant-specific guides and context. Each participant speaks
  under its own identity, with run/attempt and source-message references.
- The agents decide when the direction is sufficient and how to distribute
  work. Do not require unanimous agreement or an extra human confirmation.
- Record each commissioned work item and its originating discussion/run.
  Use existing WFE delegation, waits and continuation. Waiting releases the
  execution slot.
- The coordination path must keep ownership until results are accounted for.
  Determine whether the facilitator or a participating coordination run
  awaits each child; preserve that ownership through restarts.
- A request receipt or "started" post is not a delivered result. Return
  reports and artifact/run links to the original discussion, including
  unsuccessful and incomplete outcomes.
- Resume result review when the commissioned work has resolved, or when
  an execution limitation can be reported. Do not depend on another human
  message to resume. If agent coordination itself stalls before review,
  retain that as the terminal outcome rather than staging a review for it.
- During review, participants may ask what a report means, inspect existing
  artifacts and logs, and correct unsupported claims. They must not launch
  another experiment, rerun the game, repair the product or prepare a second
  execution cycle. Missing evidence remains missing.
- Implementation and verification needed for the initial attempt belong to
  its execution phase. The review boundary must not be used to move new work
  back into execution after results have been presented.
- Preserve disagreements and unknowns in the final synthesis. Suggested
  future improvements may be recorded as observations, without turning them
  into new work requests.
- Archive the review and final state in `pj-argue`. Closing the discussion
  does not automatically cancel its child runs; account for outstanding work
  explicitly when ending a stalled trial.

## Step 1 — inspect and prepare the missing support

1. Re-read p16's contracts, deployment evidence and current implementation.
   Check runtime state through nctl and WFE's read-only surfaces.
2. Verify participant identities, existing workflows, execution profiles,
   guides, project access and active runs. Treat environment facts as
   observations, not assumptions inherited from the previous investigation.
3. Add `discuss` workflows and participant registration for Front, RTS and
   jevlab if still absent. Adapt Front's direct-conversation-only guide
   assumptions so its discussion runs use the correct reply surface.
4. Preserve each project's own knowledge and responsibilities. Ensure generic
   reply, delegate and result-return operations are discoverable without
   supplying an RTS/Jev solution.
5. Trace the current facilitator/synthesize handoff path. The p16 guide says
   the facilitator turn can finish after requesting synthesis and is not
   notified when it completes. Establish a durable execution-to-review path
   rather than relying on that finished turn.
6. Implement only the missing mechanism using existing WFE facilities where
   possible. Specify correlation, continuation ownership, failure outcomes
   and duplicate suppression in the owning documentation/help.

Deliverable: `report1.md` listing readiness, concrete gaps, changes and the
selected execution-to-review path.

## Step 2 — verify and deploy before the trial

1. Use isolated fixtures or an unrelated trivial task to prove participant
   dispatch, delegated work, return to the same discussion and review
   continuation. Do not rehearse the RTS/Jev task or seed its answer.
2. Check unsuccessful child results, result/question attribution, duplicate
   delivery and restart recovery. Check that waits release the conversation
   slot and the review can occur without a new human post.
3. Verify the per-trial review-only instruction reaches the responsible
   runs. Do not hardcode it as a global argue restriction or introduce
   a mandatory debate procedure.
4. Complete checks appropriate to the touched repositories. Back up affected
   stores/configuration and record code-and-data recovery locally.
5. Deploy the required changes and verify the discussion UI/API and result
   links from another LAN machine. Confirm participant availability and
   that unrelated active clients still work.
6. Record deployed revisions and establish passive observation through
   discussion, invitation, run, child-run and archive records. Determine
   whether any automatic recovery agent could affect the trial and record
   its actions as part of the evidence.

Deliverable: `report2.md` with readiness evidence, revisions and verification
results. Technical readiness is not an additional human approval gate.

## Step 3 — launch once and observe discussion and execution

1. Create one discussion with the identified participants. Record the source
   braindump revision, exact goal and explicit trial protocol. Distinguish
   Developer-authored instructions from automation metadata.
2. Submit the initial request once through the normal discussion interface,
   using its idempotency mechanism.
3. Observe without steering: invitations, contributions, decisions, delegated
   requests, outstanding questions, execution and result delivery.
4. When an obstacle appears, apply the experimental boundary above. Repair
   a demonstrated system gap with evidence; do not rescue coordination.
5. Keep an intervention ledger: time, affected operation/run, evidence,
   classification, change, verification and resume outcome. Record whether
   a correction altered the agents' available capabilities during the trial.
6. Follow commissioned work beyond the end of a facilitator turn. Do not
   mistake discussion silence or successful dispatch for task completion.

Deliverable: `report3.md` with the trial identity, chronology, work links,
interventions and observed execution outcome.

## Step 4 — observe the result meeting and terminate

1. Let the implemented continuation bring reports back to the same argue.
   Each responsible participant reports its actual work and limitations;
   other participants can examine and question the existing evidence.
2. Observe whether the group establishes what happened, what remains
   unsupported and whether its reports address the original goal. Judge the
   accuracy of those reports separately from the quality of the game.
3. End after report examination, with no second plan/execution cycle. If
   agents attempt new work contrary to the explicit protocol, record that
   behavior and terminate the trial rather than letting another cycle run.
4. If coordination stalled earlier, report that terminal outcome without
   supplying the missing coordination. Explain which part of the intended
   round trip was not reached.
5. At termination, inspect outstanding runs and requests. Stop/cancel only
   trial-owned work that would otherwise continue beyond the boundary,
   recording that operational action. Do not interrupt unrelated work.
6. Verify archival catch-up and published revisions. Preserve participant
   synthesis where produced; keep the external evaluator's conclusions
   separate from agent-authored speech and records.

Deliverable: `report4.md` with the result-review transcript or the reason it
was not reached, final work state and archival evidence.

## Step 5 — report the experiment

Write `report.md` with:

- The exact request, protocol, participants and deployed system revisions.
- The observed path from discussion through execution and review, including
  any stage not reached.
- What the agents claimed, what existing evidence supports, and what remains
  unknown about AI-versus-AI play and Jev's actual involvement.
- System defects and repairs, agent coordination behavior, automatic recovery
  actions and external interventions, clearly distinguished.
- Whether result delivery resumed the same discussion, preserved identities
  and provenance, and respected the review-only stopping instruction.
- Timing and waits with their evidence; no quality-based success requirement.
- Direct links to the discussion, work runs, existing results and published
  archive, plus limitations and observations for future work.

Update affected developer documentation and ignored environment notes.
Commit/push intended changes and parent pins where applicable, preserving
unrelated edits. Do not start follow-up improvement work as part of closure.

## Completion criteria

- Required system capabilities were prepared and verified independently of
  solving the RTS/Jev task.
- The initial goal was submitted once with the Developer's explicit
  execution-and-review protocol and no added solution instructions.
- Agents were allowed to determine the approach and work allocation without
  human or Omni substantive steering.
- Either the discussion/execution/review round trip was observed, or its
  terminal coordination/experiment limitation was documented with evidence.
- Demonstrated system defects were distinguished from agent behavior, and
  every external intervention was recorded.
- No second planning/execution cycle was performed after result review.
  Trial-owned outstanding work is accounted for.
- Records are archived, final evidence is accessible with direct links, and
  the report states what was and was not established.

A high-quality game, a successful Jev integration, unanimous conclusions
and a favorable assessment of argue are not completion requirements.
