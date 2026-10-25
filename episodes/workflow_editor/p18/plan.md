# Workflow editor p18 — one conductor run per conversation

## Goal

Make a conversation's progression readable as one growing ponderer graph.
Keep the conductor's run across replies, questions, delegated outcomes and
human review. Keep each recipient project's runs scoped to the work it was
asked to perform, with their own outcomes and lifecycle.

The Developer reported that response-sized ponderer runs make aghangout
history difficult to follow. The agreed direction separates two records:

- **Conductor run:** how this conversation developed, what ponderer decided,
  whom it asked, and how it took up the results.
- **Recipient run:** execution of a particular commission in its owning
  project, including artifacts, failures, corrections and completion.

One conversation normally has one conductor run. A new reply or a new topic
inside that conversation does not by itself create another conductor run.
This does not require one continuously running process or one unbounded
model session. Attempts remain bounded and release their executor slot.

## Working conventions

Follow `localrule.md`, `devpolicy/styles.md`, `devpolicy/terms.md` and
`devdocs/README_DEV.md`. Write generated files in English and communicate
with the Developer in Japanese. Backward compatibility is not required.
Implementation, appropriate commit/push, deployment and verification have
no human permission milestone. Human review of a produced result remains
a product interaction and an assessment record, not deployment permission.

Read machine-specific details from `pj-agdev/.local/devenv.md` and other
ignored notes. Before inspecting or changing running services, obtain
state from pj-clusterintent's Nautobot or nctl, following
`pj-clusterintent/nctl/README.md`. Keep addresses, credentials and local
deployment paths out of tracked documents. Deploy the finished work for
LAN access and verify it autonomously; leave the Developer a direct entry
point requiring minimal interaction.

This plan defines implementation work; writing it does not itself execute
that work.

## Existing foundations

Read and trace the current implementation before making changes:

- `pj-agdev/agdevworld/workflow_editor/docs/conversations.md` and `runs.md`.
- WFE `server/conversationService.ts`, `conversations.ts`,
  `conversationWorkflows.ts`, run operations/reducers and execution recovery.
- agautolab's executor and shared wfrun guide.
- The managed ponderer project's `agent/conduct.md` and conduct workflow.
- The managed aghangout project's room and workflow graph views.
- p17 and p17/ex1 review, cleanup and continuation behavior.

Current routing already sends messages to an open conductor run. Completed
conductors are replaced on a subsequent message or awaited result.
`ponder --next` already extends the run graph; a waiting attempt can end
without occupying an executor slot. Build on these facilities. Establish
where current completion happens rather than assuming every reply creates
a run directly.

## 1. Define conductor ownership and lifetime

Give each conversation a durable conductor-run identity. Create it when
ponderer first begins conducting, including when facilitation starts in an
existing direct or group conversation. Earlier messages remain context;
do not invent conductor activity for them. Direct consultation with
ponderer through `converse` and ordinary participants' conversational runs
are outside this change unless needed to preserve existing routing.

Separate conversation availability, facilitation, conductor lifecycle and
individual work obligations. Define the following transitions explicitly:

| Event | Required behavior |
| --- | --- |
| Ponderer sends a reply | Finish that activity and attempt as appropriate; retain the conductor run. |
| No immediate work remains | Become idle for conversation input; no executor slot and no polling LLM turn. |
| Human answer or delegated outcome arrives | Record its source and continue the same conductor run once. |
| A new topic begins in the room | Append a new stretch of progression in the same run. |
| Facilitation ends | Suspend unsolicited conductor routing; preserve its identity and outstanding obligations. |
| Facilitation starts again | Reuse the conductor identity and explicitly resume applicable work. |
| Conversation closes | Suspend intake/continuation according to the room contract; preserve pending work and late outcomes. Do not cancel recipient runs or approve reviews. |
| Conversation reopens | Reuse the conductor run; process the new request without replaying prior work or automatically releasing held outcomes. |
| Conductor explicitly cancelled | Preserve cancellation; make any replacement explicit and linked to its predecessor. Never silently revive it. |

Closing an idle room may finalize its conductor record, provided reopening
has a recorded same-run continuation transition. Closing a room with open
obligations must not assert that those obligations succeeded. Choose and
document the exact state representation before implementing consumers.
Do not weaken ordinary recipient-run completion to implement conductor
reopening. Keep this lifecycle specific to the conductor's role.

Conversation input readiness must not require a fabricated question or an
unfinished ordinary task. Distinguish idle conversation input from a named
answer wait, delegated result wait, human review, execution failure and
explicit hold. The scheduler and observers must understand that distinction.

## 2. Implement durable continuation and graph growth

Update WFE's routing, operations, reducers and reconciliation together.
Persist conductor identity and lifecycle transitions so a restart cannot
replace an idle conductor or lose the event that should wake it.

Reuse message tokens, request receipts, attempt fencing and outcome
identities. Repeated delivery must not append duplicate nodes, start a
second conductor or commission the same child twice. Handle messages that
arrive while an attempt executes or ends, and human input concurrent with a
child result. Each accepted event must remain attributable and be consumed
or visibly held, even if several events share one subsequent attempt.

Record new activities through existing versioned graph adjustments. Keep
completed nodes and their evidence; append new nodes or record explicit
corrections. Link messages, asks, decisions, reviews and results to the
activities they concern. Temporal succession is not automatically a work
dependency: avoid making independent topics wait behind unrelated reviews.

A persistent conductor is not a persistent executor process, worktree lock
or model context. Audit attempt context construction, checkout ownership,
workflow version pinning, revision storage and cleanup for a long-lived
run. Build subsequent briefs from recorded state, relevant conversation
history and outstanding obligations; a summary must retain links to the
underlying evidence. Avoid loading every historical graph snapshot on each
turn or making old completed nodes executable again.

## 3. Preserve recipient-project work boundaries

Do not combine recipient runs into the conductor run. Each commission keeps
its owning workspace, workflow/run identity, input, acceptance semantics,
artifacts and execution history. The conductor's ask/delegate node links to
that identity and records how its result was assessed and used.

| Situation | Recipient behavior |
| --- | --- |
| New independent commission | Create a new recipient run. |
| Clarification or additional conditions for existing work | Deliver to the existing run with message provenance. |
| Review requests changes to the same result | Use the existing correction/follow-up mechanism for that child. |
| Separate deliverable or unrelated work | Create another run, even within the same conversation. |
| Recipient finishes | Complete its own work independently of the conductor's lifetime. |
| Room closes or ponderer becomes idle | Do not infer child cancellation, acceptance or integration. |

Preserve p17 human-review rounds, exact presentation versions, cleanup
ownership and p17/ex1 continuation after approval. A resolved review may
complete the relevant stretch of work while the conductor returns to idle.
Changing a reviewed result creates the appropriate correction and a new
review round; previous approval does not cover it. Readiness to receive a
future chat message must not keep child acceptance or cleanup pending.

Update ponderer and shared agent guidance with these ownership facts.
Keep shared facts in one place and command syntax in tool help. Do not
replace agent judgment about commissions and topic boundaries with a fixed
conversation script.

## 4. Show one progression in aghangout

Make the conductor graph the conversation's primary progression view.
Append its activities as the conversation develops, with direct navigation
between a node and its related messages. Keep the existing recorded-version
view: the graph at a past utterance and its current state are distinct.

An ask/delegate node shows the recipient, purpose, state and link to its
run. Expand or open recipient details on demand instead of embedding every
child node in the primary graph. Support multiple recipients and repeated
corrections of the same child without implying that they are new jobs.

Let readers collapse completed stretches and focus on current activities,
open questions, ongoing commissions and human reviews. Topic labels and
summaries aid navigation; they do not create new run identities. Use clear
Japanese product labels for idle input, active execution, external waits,
review, suspension and failure. An idle conductor must not appear as a
stalled job or inflate the count of actively executing work.

Update WFE run views, lists, focus cards, notifications and observation
alongside aghangout so they express the same state. Preserve explicit
replacement links for exceptional cancelled conductors and access to
historical runs.

## 5. Transition existing conversations honestly

Prepare a deterministic transition for existing rooms and test it against
a copy of current data. Keep old run identities, messages, artifacts,
workflow snapshots and provenance intact; do not concatenate past execution
histories into a fictional single run.

Reuse an existing open conductor where valid. For rooms with completed
conductors, establish the continuation owner through an explicit recorded
transition, reusing the latest suitable run or linking a new persistent
conductor when reuse is invalid. Retain earlier runs as historical stretches
in the room view. Resolve multiple candidates explicitly and visibly.
Migration itself must not execute work, replay messages or send replies.

No compatibility layer is required. Document changed schema/event readers
and prepare a matching code-and-data recovery point. Preserve authoritative
records across deployment and recovery; do not restore a database snapshot
over newly accepted live messages without accounting for them.

## 6. Verify behavior, deploy and leave a usable example

Add focused tests at lifecycle, delivery and graph boundaries, using the
repository's required checks. Verify at least:

1. Several human messages and ponderer replies keep one conductor identity
   while attempts end and nodes accumulate. A second room can use the freed
   execution slot while the first is idle.
2. Two independent commissions create separate recipient runs. Clarification
   and correction reach the original child; each child can complete while
   the conversation remains open.
3. Concurrent human input, child completion and attempt termination lose no
   event and create no duplicate conductor, child or graph activity.
4. Coordinator/executor restarts preserve idle, waiting and active states;
   delivery retries remain idempotent.
5. Facilitation end/start and room close/reopen retain conductor identity,
   preserve held outcomes and do not silently cancel children or replay work.
6. Human review still starts the exact presentations, records changes or
   approval, cleans up owned instances and continues the responsible work
   once. The conductor can return to idle afterwards.
7. Historical graphs remain readable; long conversations can be navigated
   without expanding or fetching all child details and snapshots at once.
8. Existing-room transition creates no agent execution or fabricated history.

Check current service state through Nautobot/nctl, deploy coordinator,
executors, guides and aghangout as required, and verify the deployed paths.
Use a clearly attributed check conversation and synthetic review decisions;
never record a trial decision as the Developer's. Exercise multiple actual
ponderer turns, a recipient commission and correction, an idle interval,
and close/reopen. Inspect evidence and browser behavior autonomously.

Leave a compact deployed example where the Developer can immediately see
one growing conductor graph and open the separately owned recipient runs.
Record releases, relevant run/message identities, validation results,
remaining limitations and recovery references in `p18/report.md`, with
machine-specific details in ignored notes. Update the development reference
and local environment notes to describe the final behavior.

## Completion criteria

The ordinary facilitated conversation no longer creates a ponderer run for
each response. Its graph retains the conversation's progression across
attempts and idle periods. Recipient projects still have independently
managed runs for real commissions. The deployed UI makes both levels clear,
and recovery, corrections and human review preserve their recorded meaning.
