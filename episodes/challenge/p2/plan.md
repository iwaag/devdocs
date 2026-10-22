# Challenge p2 — unified conversations

## Objective

Unify direct chat and facilitated discussion into one conversation model and
one aghangout room implementation. A conversation can begin with one project
agent and later include specialists or a conductor without changing its identity,
losing its history, or interrupting work already entrusted through it.

Preserve every conversation, including one-to-one chats, in a durable Git archive.
Reuse p1's challenge records, shared attachment table, pinned revisions, and
ChallengePanel.

## Basis and working rules

- Sources: `../p1/braindump.md`, `../p1/plan.md`, `../p1/report.md`, and the
  Developer's agreement to proceed after the p1 review.
- Read `devpolicy/styles.md`, `devpolicy/terms.md`, `devdocs/README_DEV.md`,
  `pj-agdev/.local/devenv.md`, and `localrule.md` before implementation.
- Write generated documentation in English; communicate with the Developer in
  Japanese. Keep local addresses, credentials, and machine-specific commands
  in ignored local notes.
- No human approval gates are needed for implementation or deployment.
  Backward-compatible APIs are not required; preserving existing information
  and active work is required.
- For live-service work, establish state through Nautobot or nctl, following
  `pj-clusterintent/nctl/README.md`. Deliver a deployed result accessible from
  another LAN computer with few operations needed to inspect it.

## Starting point

p1 delivered Git-backed challenges, conversation-store v11 attachments, pinned
run inputs, and deployed checks in both chat and discussion. Its report records
WFE `aadee43` and aghangout `6e311e4`; inspect actual revisions before changing
anything rather than assuming those releases are still current.

The two conversation kinds already share messages and attachments. Their
delivery semantics differ: direct chat starts a destination turn for each human
message; discussions use invitations and can deliver to an existing conductor
or the run that asked a question. Discussions also have membership, activities,
workflow references, overviews, and Git archives. Direct chat has no archive
export yet.

## Target contract

### One conversation, optional facilitation

A conversation has a stable ID, title, open/closed state, participants and their
membership history, messages, references, and an optional conductor. It may
also have a default addressee. Ordinary conversations and operational checks
remain distinguishable in lists.

- A one-to-one room defaults to its project participant.
- A facilitated room defaults to its conductor, normally ponderer.
- A group without a conductor uses its explicit default addressee or explicit
  recipients; an ambiguous message stays a draft with a recipient choice.
- Being present as a participant does not make ponderer the conductor.
- Starting facilitation is an explicit action with an initial brief and a
  recorded history horizon. Joining a member or changing room presentation
  alone starts no work.
- Adding participants or starting facilitation keeps the same conversation,
  messages, outstanding questions, run links, and challenge attachments.
- Ending facilitation changes future default routing, not ownership of past
  work. Outstanding results remain associated with their original requester.
  Stop/cancel work only through the existing explicit run operations.

Keep project identity separate from workflow identity. A project may initially
retain its existing `converse`, `discuss`, and `conduct` role guides where their
responsibilities differ. Select and record the workflow for each response
request explicitly; retain it for the request's lifetime. Consolidate guides
when their actual responsibilities coincide, without rebuilding two transport
implementations around their names.

### Shared delivery

Generalize the discussion invitation mechanism into the common response-request
mechanism. Keep message persistence, recipient resolution, delivery receipts,
retry, and recovery in one implementation used by UI, CLI, Slack, and agents.

| Situation | Required behavior |
| --- | --- |
| New message to a project | Record one response request per recipient and start the appropriate bounded turn |
| Reply to an open run question | Deliver the answer to the question-owning run; do not also start an independent turn for that recipient |
| Message to an active conductor | Continue its run with the answer or instruction |
| Explicit instruction to existing work | Preserve the target run, original words, source message, and attachments |
| Agent contribution | Record speech and provenance; do not automatically awaken other participants |
| Late result | Settle the original awaited request and use its recorded continuation policy, without guessing from the latest message |
| Unavailable recipient | Show the failure and support retry at that recipient; never redirect silently |
| Repeated submission or restart | Reuse the recorded request and receipt; do not duplicate speech, runs, or instructions |

An unqualified follow-up is interpreted by the receiving agent, not by
heuristics that silently choose a related execution. Explicit question/run
references take precedence over default routing. Keep invitation creation
available to the person and conductor; avoid automatic participant cascades.

Closing a room withdraws response requests not yet dispatched, does not cancel
active runs, and does not erase obligations. Reopening replays nothing. Late
results remain readable while closed; any continuation obligation is retained
and shown rather than silently starting new conversation work in a closed room.
Specify and test the explicit resume behavior before migrating existing rooms.

### Durable archives for every room

WFE owns export and publication. Use the existing archive repository in
pj-ponderer for both direct and facilitated conversations, with a common
manifest/transcript contract. Archive ownership is independent of the presence
of a conductor: export never launches ponderer or requires a synthesis run.

Keep the existing discussion archive paths and historical commits valid. Add
direct conversations under a distinct namespace such as `conversations/<id>/`
and record each room's archive location. Promotion to facilitated conversation
does not move its archive or duplicate its history. Both namespaces use the
same exporter and read contract after migration.

Export verbatim messages, authorship, message IDs, membership and facilitation
events, delivery outcomes, run links, challenge commits, and applicable
overviews, activities, annotations, and workflow snapshots. Retain the existing
single-writer and published-horizon recovery behavior. Report lag and failures
without blocking conversation use. Historical source links remain resolvable
through ID-based navigation even if the old UI routes disappear.

Document archive backup and restore. A fresh clone must recover readable
transcripts and pinned references. Challenge contents continue to live in their
own repository; archive recovery must state that dependency. Archive-only
restoration restores records, never replays delivery or executes historical work.

## Implementation sequence

### 1. Inventory and finalize the common contract

Inspect WFE conversation/discussion stores and routes, invitation receipts,
question routing, conductor activities, exports, workflow references, character
dialogue, Slack integration, and aghangout's room/list/composer implementations.
Inventory active turns, questions, conductors, and pending deliveries read-only.

Write the common contract and a migration mapping for every affected record.
Resolve receipt preservation, role selection, stopping facilitation, closed-room
results, and archive paths in this step. Record implementation choices in the
episode rather than adding process approvals.

### 2. Implement common storage and response delivery

Extend the shared schema for optional facilitation and unified delivery. Migrate
legacy direct destinations into participants/default recipients and legacy
dispatch records into response requests. Keep existing run IDs and receipt keys
for migrated work; historical deliveries must never appear newly pending.

Preserve active conductor and invitation state, requester identity, pending
questions, all run links, and challenge revision references. Introduce no model
calls or dispatch side effects during migration itself.

Provide one service layer for creation, reading, posting, membership, recipient
selection, facilitation, questions, retries, closing, and reopening. Retain
conductor-specific activities as optional capabilities of the same conversation.
Adapt CLI and HTTP contracts, and remove obsolete duplicated delivery paths once
all callers have moved.

### 3. Generalize archives and backfill direct history

Extend the archive writer to every room, including closed direct chats and
rooms whose project is retired. Backfill using stable identities and export
horizons. Preserve existing discussion archives and their publication evidence.

Verify interruption after push, concurrent export attempts, and a second backfill
produce no duplicate transcript entries. Include attachments and migration
provenance without inventing historical membership timestamps or actor actions.
Show archive status and source links in the common room.

### 4. Update agents, clients, and shared guidance

Update shared wfrun guidance, Front and ponderer guides, and participating
project workflows as needed for the common commands and input contract. Preserve
each project's mission and context when entering a group room.

Update Slack to the common posting and reply path, retaining user attribution,
message deduplication, existing destination mappings, and links into aghangout.
Update request/delegation callbacks, archive readers, workflow-reference views,
and character-dialogue consumers. Do not start models merely to migrate data or
render a list.

### 5. Consolidate aghangout

Use one room shell, message list, composer, work/questions panel, challenge
panel, and conversation list. Keep existing character presentation where useful.
Present one-to-one and facilitated modes as views of the same room, with members
and the current conductor visible. Reveal conductor progress when applicable.

Provide direct actions to invite a project and ask ponderer to lead. Show who
will receive a message before sending. Preserve drafts, attachment selections,
outbox tokens, scroll/history navigation, and ongoing work while the room changes
mode. Ordinary users should not need to understand invitations or database kinds.

Preserve exact wording and access to original messages when character dialogue
is shown. Keep operational checks out of the ordinary default list. Resolve old
conversation/discussion links into the new room; API compatibility is optional,
but existing citations must continue to identify their records.

### 6. Verify migration and behavior

Use fixtures and a copied store for migration tests. Cover these scenarios:

- One-to-one message/reply, with no conductor created.
- Direct consultation with ponderer without implicit facilitation.
- A one-to-one room gains a conductor and specialist without changing IDs or
  losing history, drafts, or challenge attachments.
- A pending question is answered after the room gains participants.
- Instructions and pinned challenges reach an already working run.
- A conductor finishes before an awaited result returns; the recorded
  continuation handles it once. Also cover closing or ending facilitation.
- Duplicate submits, restart between request and acknowledgement, unavailable
  recipients, retries, and close/reopen do not duplicate work.
- Migration of pending dispatch, active turns, questions, and conductor state
  preserves receipts and does not replay completed messages.
- Direct and facilitated archives survive publication failure and restore from
  a fresh clone with authorship, history, and references intact.
- Slack and aghangout use the same semantics; character dialogue and workflow
  evidence still point to their original messages and versions.

Run the affected repository checks and meaningful browser tests. Then verify
the deployed end-to-end flow with an explicitly named test actor and a check
room. Include a Challenge attachment and inspect actual receiving run input.
Do not solve the original image-extraction challenge as part of this trial.

### 7. Deploy and report

Observe live service state through Nautobot/nctl. Record the current releases,
back up stores and configuration, and prepare recovery commands before changing
the live schema. Rehearse migration on a backup first.

Coordinate WFE, executors, agent guides, Slack, and aghangout updates. During
cutover, briefly pause new intake/claims and let in-flight writer operations
settle so old code cannot write the migrated schema. Preserve pending input and
resume it using its existing tokens. Do not cancel useful work merely to make
migration easier; drain affected executions or explicitly support their callbacks.

Prefer forward repair after new messages arrive. Restoring a pre-migration
backup alone would lose new records and cannot undo external work: document how
post-cutover messages and effects are retained if rollback becomes necessary.

Verify the full flow from another LAN computer and inspect archive publication,
pending deliveries, and service state after deployment. Keep local evidence and
recovery commands in ignored files. Write step reports and `report.md`, update
development documentation, and give the Developer a short path to inspect the
deployed result. Remove obsolete code after verification rather than retaining
two active conversation engines.

## Completion criteria

- Chat and argue use one conversation service, dispatch mechanism, and room UI;
  facilitation is optional and explicit.
- A direct room can gain participants and a conductor without losing its ID,
  messages, active work, or pinned Challenge references.
- Questions, instructions, late results, retries, and restarts preserve the
  correct recipient/run and do not duplicate execution.
- Both existing and new direct/facilitated rooms have durable Git archives,
  visible publication status, and verified recovery.
- Existing history, authorship, run IDs, receipts, and citations survive migration.
- Slack, agent tools, remote executors, and aghangout all use the common contract.
- The deployed LAN flow is verified, and reports include evidence, known limits,
  and recovery instructions.

## Outside this phase

Challenge automatic triage, automatic assignment, image-extraction improvements,
and a new dedicated challenge agent are not needed for conversation unification.
p1's evidence-upload size limit and remote `evidence --out` destination behavior
remain separate improvements unless they directly obstruct a p2 verification.
