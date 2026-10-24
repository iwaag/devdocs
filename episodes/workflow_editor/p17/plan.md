# Workflow editor p17 — deferred human review

## Goal and evidence

Let ponderer review delegated work, judge it satisfactory and finish the
agent's work without waiting for the person. Preserve a final Human review
node where the person can start a review with one button, reopen the exact
presentations, approve and trigger cleanup, or request changes that extend
the workflow and lead to another review.

This implements the Developer's proposal and the discussion following
[fix6/report.md](../fix6/report.md). In fix6 trial A, ponderer accepted
misc/freework/run-007 itself and asked the child to stop its presentation.
The child complied; the person never received a usable review session.
Agent review and early cleanup are useful behavior under the proposed
operation. What was missing was a durable, person-controlled path to review
the completed result later.

## Agreed behavior

- Agent acceptance and human review are distinct facts. `accepts: parent`
  remains a valid delegation choice; do not force every child to wait for
  the person or warn merely because an agent reviewed it first.
- Present the aggregate state as **Work complete · Awaiting human review**.
  The pending review does not occupy an executor slot. Avoid reporting the
  entire run as finally completed while its review node remains pending.
- The person presses **Start review** once. WFE starts the referenced
  presentations, verifies availability and displays the review material.
  No terminal command or manual agent instruction is required.
- **Approve** records the person's decision for the reviewed versions and
  starts cleanup. Finish the node after stopping its review instances is
  verified. A cleanup failure never asks for approval again.
- **Request changes** records feedback and closes this review round with
  that outcome. Resume the responsible agent, which appends work and a new
  Human review node. Preserve the rejected round and its evidence.
- Use one visible workflow node for preparation to view, human feedback and
  cleanup, with internal lifecycle facts. Do not require separate visible
  present/confirm/cleanup nodes for this review session.
- Human approval here records assessment of a result. It is not permission
  to run agents, revise workflows, deploy changes or integrate existing
  results. Existing integration semantics remain independent.

## Working conventions

Follow `localrule.md`, `devpolicy/styles.md`, `devpolicy/terms.md` and
`devdocs/README_DEV.md`. Write generated files in English and communicate
with the Developer in Japanese. Implementation, appropriate commit/push,
deployment and verification need no approval milestone. Backward
compatibility is not required; retain history and prepare matching code/data
recovery for changed event or store schemas.

Read local environment details from ignored notes. Before inspecting or
changing running services, obtain state through `pj-clusterintent` Nautobot
or `nctl`, following `pj-clusterintent/nctl/README.md`. Keep machine-specific
addresses, credentials and deployment/evidence paths out of tracked files.

## 1. Define the node and durable review contract

Trace node compilation, reducers, run completion, questions/decisions,
executor scheduling, observation, delegation and conversation lifetime
before editing. Define Human review as a first-class workflow capability,
available through the node catalog, editor and CLI/API. Prefer a dedicated
node type if existing types cannot express host-managed waiting and cleanup
truthfully; settle the representation before implementing its consumers.
Do not implement this as a prompt convention on an ordinary agent task.

Record a review round's owning run/node, prepared-by actor, review summary,
subject references, responsible agent and relation to any previous round.
Each subject identifies its workspace, run identity, presentation id and
manifest revision, plus the result/repository revisions it represents.
Freeze that set when the round is prepared. A later child result must not
silently replace the material the person is about to approve.

Keep agent acceptance, the person's answer, review outcome and cleanup
outcome separate. Reuse question/answer provenance and result binding where
appropriate, but do not equate a parent's review of several children with
each child's acceptance or initiate integration merely from review approval.
An agent's own assessment must not be displayed as a human answer. Preserve
explicitly attributed, evidence-backed relay of actual human feedback.

Model at least these internal states; final names follow project conventions:

| State | Meaning and next action |
| --- | --- |
| Awaiting review | Agent work and review subjects are ready; person may start review; no agent process waits. |
| Starting | Durable start request; host starts and verifies each presentation. |
| Reviewing | All required subjects are available; person can approve or request changes. |
| Start failed | Identify unavailable subjects and allow retry without rebuilding the result or losing the round. |
| Approved, cleanup pending | Decision recorded once; host stops review instances and retries failures. |
| Approved, complete | Required cleanup verified; node complete. |
| Changes requested | Feedback recorded; this round closes and a durable continuation hands work back to the responsible agent. |

Define cancellation, stale actions, service restart and partial availability
explicitly. Never treat a click on Start review as proof the person viewed
or approved the result. Do not allow an unavailable required subject to be
silently omitted from an approval.

## 2. Make presentations reopenable and owned

Reuse the existing immutable presentation bundles and `Presenter`
start/status/stop machinery. They already preserve hashes after stopping
and support starting another instance of the same revision. Verify the
actual implementation for completed child runs and remote execution;
extend routing only where necessary. Do not rebuild from a mutable worktree
or select the latest presentation when starting an older review.

Ensure referenced bundles remain readable and retained while their review
is pending or active, including after child integration and ordinary
workspace cleanup. Missing or corrupt bundles yield a visible recoverable
failure, not a substitution. Resolve logical references to current serving
addresses; keep host-specific process and port details in host stores.

A parent's node may collect presentations from several delegated projects.
Lifecycle effects run through each presentation's owning service/run rather
than transferring ownership to ponderer. Record which instances the review
started or reused. Prevent one review's cleanup from stopping an instance
still needed by another active review or explicitly retained on request;
use durable usage/retention records and display any outstanding ownership.
In the ordinary case, approval stops every instance started for the review.

Start and cleanup are durable host operations, not LLM turns. Reconcile
partial starts, restarts and uncertain stops using existing instance
identity checks. Double clicks and retries must not create duplicate
instances or decisions. Never infer ownership from a PID or port alone.

## 3. Integrate deferred review into agent work and corrections

Teach ponderer to prepare the final Human review node with a short summary
and selected child presentations after its own review. Agent-side cleanup
before that point is allowed. The child must leave an immutable presentation
bundle and usable review information, rather than only transient files or a
dead URL. Inspect Front and shared wfrun guidance for the same use case;
keep shared facts in one place and tool usage in command help.

Preparing a valid deferred review makes the handoff visible in the parent
run, conversation and brief. Update fix5/fix6 assessment to distinguish
human review pending in this node from a lost presentation. A stopped bundle
attached to a pending review is expected, not a missing presentation or
forgotten cleanup. Agent-only review without a deferred handoff must not be
misrepresented as human approval. Preserve reminders for actual omissions.

On Request changes, persist feedback and queue the owning agent once. It
may do local work or return work to a child. Reuse fix6's same-child
correction mechanism, but account for its current restriction to open
delegations: by human review time the parent may already have disposed the
child as satisfied. Provide an explicit follow-up transition referencing
that closed result, preserving the earlier disposition, to permit correction
of the same child where appropriate. Do not silently erase closure or force
an unrelated new run merely because review was deferred. Define behavior for
cancelled/archived children and unavailable workspaces without fabricating
successful reopening.

Append new pending work and a new review node using ordinary immutable run
revisions. Previous completed work and review rounds stay historical. A
newly frozen presentation represents changed output; old human approval
never covers it. Keep the parent in a correction-pending state until the
continuation is addressed, so closing the old round does not accidentally
complete the whole run or send a final-success notification.

After changes are requested, the agent may keep the review instance for
ongoing comparison or stop it when no longer useful. Record responsibility
either way. No failed/rejected round may leave an unowned serving process.
Waiting for the next human review releases the executor slot again.

## 4. Build the one-button review experience

Add the Human review node to the editor, graph and run details. Display
agent work completion and human review state separately in lists, focus
cards, observations and notifications. Pending human review is an owned
human wait, not a stalled agent or a reason to repeatedly launch attempts.

Expose the review entry point in the person's conversation, including the
aghangout room used by ponderer, and in the WFE run view. A single Start
review action starts all subjects and opens the review UI when ready. Show
progress and retryable failures in place. Provide a concise overview and
links/tabs for multiple subjects, with what changed and what to check.
Preserve the existing isolation of served content from the editor origin.

Place Approve and Request changes in the review UI, with a feedback field
for the latter. Keep the person in this flow through cleanup confirmation
or correction acknowledgement. CLI/API expose equivalent operations for
tools and verification; synthetic actors remain explicitly identified.
Do not auto-approve from a page load, agent message or presentation stop.

If the person leaves and returns, restore the durable state and resolve
working links rather than replaying old addresses. Show approval immediately
when recorded, with cleanup pending/failed separately. Keep past rounds
inspectable and clearly separate from the current round.

## 5. Verification

Add focused behavioral tests and run affected projects' required checks:

- A parent accepts child work, stops the initial page, prepares deferred
  review and releases its execution slot. The review remains discoverable.
- One Start review click serves the exact frozen revision from a completed
  child; changed worktree files do not change what is reviewed. Cover remote
  worker publication, retention and multiple child subjects.
- Partial startup, unavailable/corrupt bundles, double clicks and service
  restarts recover without duplicate processes, lost state or false approval.
- Human approval binds the round once; cleanup failure retries without
  asking again. Concurrent review/retention owners are respected.
- Feedback closes the old round, queues one agent continuation, extends the
  workflow and produces a distinct review of corrected output. Exercise
  already-satisfied delegation follow-up, prior integration and repeated
  feedback/delivery after crashes.
- Old-browser actions cannot approve superseded rounds. Agent acceptance
  remains distinguishable from human review; synthetic answers are labeled.
- Cancellation and abandonment leave explicit cleanup ownership. Human
  waiting does not trigger executor churn or false stalled-work incidents.
- CLI, API, remote routing, editor and conversation entry points agree;
  report boundaries recognize valid deferred review without hiding genuine
  missing handoffs. Existing ordinary presentations keep working.

## 6. Deploy, demonstrate and report

Read Nautobot/nctl state and local release notes, prepare necessary backups
and matching rollback instructions, and deploy affected WFE, executor,
guidance and conversation UI releases. Update applicable worker pins through
the established deployment paths. Verify actual deployed versions and
behavior, not only local source tests.

Run a facilitated trial with a small viewable delegated deliverable. Let
ponderer perform its own review and prepare the deferred human review
without the Omni Agent constructing it on ponderer's behalf. Verify the
waiting state, start from the conversation button, inspect the presentation
from another LAN host, request a small concrete change as a named check
actor, and then review the corrected round, approve and verify cleanup.
Record autonomous behavior separately from controlled recovery fixtures.
Never impersonate the Developer or count synthetic approval as theirs.

Leave a small completed-work example awaiting the Developer's review, with
its presentation stopped and preserved for one-button startup. Provide its
conversation/run entry point and what to check. This waiting example is the
requested product behavior, not a deployment approval gate; automated and
identified-actor checks must already have exercised approval and cleanup.
Keep services accessible from the LAN rather than restricting to localhost.

Write `report.md` with implemented contracts, validation, deployed trial
references, autonomous outcomes, limitations and recovery references. Update
development documentation and ignored environment notes. Completion means
the deployed one-button flow, correction loop and cleanup are verified and
the Developer can inspect the example with minimal operations; do not
claim success solely because a node was added or unit tests passed.
