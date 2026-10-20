# Workflow editor p16 — argues and shared memory

## Objective

Implement [braindump.md](braindump.md): a person and agents with different
missions and contexts develop ideas in a shared discussion. Preserve that
discussion and its durable findings in one system-wide `pj-argue` repository,
with access from the agdev UI.

The experiment asks whether interaction between differently situated agents
produces useful ideas beyond their separate contributions. Each participant
speaks through its own project and run. A facilitator helps the discussion
progress without replacing participants' voices or requiring consensus.

Keep the person's intentions and cross-project reasoning discoverable beyond
the lifetime of individual implementation projects. Projects retain their own
plans and results, linked to the discussion that motivated them.

This document plans implementation; creating it does not start implementation.

## Working conventions

- Follow `localrule.md`: generated files are English; communication with the
  Developer is Japanese. During implementation, ordinary development, trials,
  deployment and appropriate commits/pushes need no additional human approval
  checkpoints. Review and acceptance records are not permission gates.
- Deliver a deployed application accessible from other home-LAN machines,
  with direct review links and minimal manual setup. Perform verification
  and operational work the agent can perform itself.
- Backward compatibility is unnecessary. Update affected active clients
  together; preserve unrelated data and account for outstanding delivery.
- Obtain local service state through Nautobot/nctl before deployment work,
  following `pj-clusterintent/nctl/README.md`. Environment notes and previous
  episode reports are pointers, not current operational evidence.
- Keep addresses, credentials, machine-specific configuration and raw runtime
  evidence in ignored local files. Use portable references in tracked files.
- Preserve unrelated edits and active work. Use existing project/workflow
  facilities rather than introducing a second execution engine.
- Give agents tools with discoverable help and concise role/context facts.
  Do not prescribe fixed debate rounds, mandatory consensus or approval
  procedures. Add guidance in response to observed failures.
- Record Omni interventions in agent-owned work with the one-line handoff
  note required by `devdocs/README_DEV.md`.

## Scope and ownership

| Owner | Responsibilities |
| --- | --- |
| WFE / agdevworld | Durable discussion identity, membership history, messages, invitations, run correlation, dispatch, recovery and client API |
| Managed project `argue` / `pj-argue` | Facilitation, discussion records, synthesis, cross-discussion knowledge and provenance to downstream work |
| Participating projects | Their own missions, guides and context; attributed contributions; work they take on and its project-local results |
| Managed project `aghangout` | Discussion navigation, participants, speaker presentation, input, pending replies and links to records |
| WFE Home | An entrance to discussions and their durable records |

Create `argue` through the ordinary managed-project path. Its purpose is to
develop and preserve cross-project thought; its workflows may facilitate,
record and synthesize discussions. WFE still owns dispatch and execution.
The repository also contains its guides and deterministic recording tools.

Initially use the configured human identity, an argue facilitator and at
least two participating projects, starting with forge and archsage where
suitable. Record human participants explicitly; a general multi-user account
system is outside this phase.

Reuse aghangout's presentation and current WFE execution facilities.
Slack group discussions, importing retired Zulip argues, new artwork and
general organization/tenant management are outside scope. Existing direct
conversations and the Slack-to-Front path remain usable.

## Discussion and execution contract

- Give each discussion a stable ID independent of title, repository path,
  participant set or any run. Distinguish group discussion semantics from
  existing conversations with one fixed destination.
- Record participant identities and membership changes, including human
  authors, project/agent identities and the workflows used to participate.
  Agent identity is independent of the selected model.
- Preserve original message text, author, timestamp, reply references and
  originating run/attempt. Record the execution profile through the existing
  run evidence. Human words must not be attributed to a facilitator.
- The person or facilitator can invite a participant to address a question.
  Invitations have durable identities and record their target, source
  messages and resulting run. Posting ordinary discussion text must not
  automatically start every participant.
- Human input without a named addressee can start a facilitator turn.
  A directly addressed invitation reaches that participant. Settle exact
  routing and facilitator continuation semantics in Step 1; make them
  discoverable in API/CLI help and visible through recorded invitations.
- Each participant runs with its own project guide, mission and scoped
  context plus access to the shared discussion. Record the discussion
  horizon supplied at dispatch; later messages remain readable. Do not
  merge all participants' private working contexts into one facilitator
  prompt or substitute a facilitator summary for access to original speech.
- Contributions appear as the participant's own messages in the shared
  discussion, with explicit references to what they answer. Use existing
  delegation and continuation where applicable; extend source correlation
  where the current single-destination contract is insufficient.
- A participant result may continue the waiting facilitator through WFE's
  existing machinery. The facilitator then decides whether to ask another
  question, synthesize or finish. Avoid unconditional reply-to-reply loops.
- Reuse durable submission tokens, request receipts, waits, instructions,
  questions and run status. Retries and restarts must not duplicate
  invitations, contributions or downstream work.
- Expose missing/unavailable participants and failed calls honestly.
  Never substitute another speaker silently.
- Closing a discussion ends new automatic discussion dispatch. Define the
  treatment of already-issued invitations explicitly, retaining their
  results and provenance. It does not cancel downstream work. Reopening
  resumes discussion without replaying previously completed invitations.
- Start with the existing execution capacity. Measure queued and running
  times separately; add capacity only if the trial shows the current
  conversation lane prevents usable interaction. Long specialist work
  should use ordinary delegated workflows and release the conversation slot.

## Records and knowledge contract

WFE's durable store is authoritative for live messages and delivery.
`pj-argue` contains reproducible archival copies and version-controlled
interpretation. Their roles must be explicit; no bidirectional transcript
editing or competing live message store is needed.

The initial repository layout is:

```text
discussions/<id>/
  manifest.json
  transcript.jsonl
  summary.md
knowledge/
  <topic>.md
```

Exact filenames can change in Step 1 if an existing convention fits better.
The following meanings remain:

- **Manifest and transcript:** stable discussion identity, participant
  history, source message IDs, exported horizon, format version and
  provenance to the relevant runs. An export records exactly what it
  includes; subsequent exports extend the record without rewriting speech.
- **Discussion synthesis:** the person's intent, alternatives, reasoning,
  conclusions, disagreements and unresolved questions, with source-message
  references. Distinguish a person's statement from an agent's inference
  or proposal. A synthesis may be revised without changing its sources.
- **Cross-discussion knowledge:** durable findings and policies with links
  to supporting discussions and explicit revisions when understanding
  changes. Do not indiscriminately promote every remark into global context.
- **Downstream handoff:** project-specific goals and constraints belong in
  that project's braindump/plan or normal work request. Include the source
  discussion, relevant messages and archived revision. Record the resulting
  project/run reference back in the discussion.

The canonical cross-project decision remains in argue; the implementation
plan remains in its implementing project. Necessary local explanation is
allowed, but independently editable copies of the same policy are avoided.
Existing Context blocks may expose selected knowledge through their normal
mechanism, with provenance; they must not become a second editable authority.

Support incremental archival during a discussion and a final catch-up on
close. Serialization, validation and export are deterministic tool work;
interpretation is agent work. Serialize repository publication through one
owned writer so concurrent discussions and synthesis runs do not race on a
shared checkout. Use ordinary WFE integration where it supplies this boundary.

Export and publication are retryable. Track the exported source horizon and
published Git revision separately from live message delivery. A failure to
commit or push must be visible and recoverable without losing accepted
messages or falsely claiming that they are archived. Prove recovery from the
repository's actual contents when a process stops after publishing but before
recording success.

## Step 1 — inspect and settle the boundary

1. Inspect current conversation store/API/CLI code, delegation and source
   correlation, project contexts, execution lanes and aghangout's data model.
2. Map the needed discussion features onto existing facilities. Choose an
   explicit group discussion representation and document which message and
   run primitives are shared with direct conversations.
3. Specify invitations, membership, reply correlation, closing/reopening,
   mid-flight human input and facilitator continuation. Include event traces
   for a normal exchange, a direct human question and a late reply.
4. Specify archival layout, writer ownership, export scheduling, retry and
   publication evidence. Establish how a later run discovers prior knowledge
   and follows an archived citation.
5. Inspect active clients and deliveries. Record required store/configuration
   changes, affected repositories and a coherent deployment/recovery path.

Deliverable: `report1.md` with contracts, ownership, concrete event traces,
implementation choices and focused verification cases.

## Step 2 — implement the WFE discussion surface

1. Implement durable discussions, membership history, invitations and message
   provenance using the chosen model.
2. Implement dispatch and recovery through ordinary workflow runs. Add the
   source references and participant reply operations needed by those runs.
3. Expose creation, listing, reading, participation, invitations, posting,
   closing/reopening and status through documented HTTP/CLI contracts.
4. Provide clients with pending/failed invitations, actual speaker/run
   identity and links to work and durable records. Reuse current browser
   origin and configured-author handling.
5. Verify retry, concurrent contributions, unavailable targets, membership
   changes, later human input and restart recovery with focused fixtures.
   Preserve the direct-conversation contract for active clients.

Deliverable: `report2.md`, working API/CLI and meaningful contract/recovery
tests.

## Step 3 — establish argue and real participant workflows

1. Create the managed `argue` project with its intent, guide, workflows and
   discoverable recording tools.
2. Implement facilitation: understand the person's intent, invite relevant
   perspectives, ask follow-up questions, preserve disagreements and produce
   a useful synthesis without requiring consensus.
3. Add or adapt participation workflows in at least two projects. Retain
   their professional contexts and make contributions directly attributable
   to their runs. Keep shared tool facts in shared documentation.
4. Run a bounded exchange with a human-authored brief, facilitator and two
   projects. Exercise both facilitator invitations and direct addressing.
5. Verify follow-up instructions and specialist questions reach the intended
   run with source attribution. Measure responsiveness and adjust capacity
   only if the evidence warrants it.

Deliverable: `report3.md` and a trace proving independent project participation
in one shared discussion.

## Step 4 — archive, synthesize and hand off

1. Implement incremental transcript and manifest export, serialized Git
   publication and visible export status.
2. Implement source-linked discussion synthesis and cross-discussion
   knowledge maintenance in `pj-argue`.
3. Prove that two discussions can publish without losing either record,
   and that interrupted publication resumes without duplicates or missing
   messages. Verify a fresh checkout can read the archived material.
4. Hand one concrete outcome to an appropriate project through ordinary WFE
   work, including the pinned source revision. Link its work back to argue.
5. Start a later discussion that discovers and cites earlier knowledge.
   Exercise a changed conclusion while retaining its prior rationale.

Deliverable: `report4.md`, published records, downstream provenance and
archive recovery evidence.

## Step 5 — build the discussion experience

1. Extend aghangout with discussion creation/listing, participant selection,
   membership visibility and shared conversation presentation.
2. Show the actual speaker for every contribution. Support human questions
   to particular participants, follow-ups and reading the original speech.
3. Show queued/active/failed invitations, linked runs, unresolved questions
   and available summaries without confusing a summary with original text.
4. Provide readable archived records, archival status and stable links to
   their Git revisions and downstream work.
5. Add an entrance from WFE Home. Verify long replies, narrow screens,
   keyboard input, reload, preserved drafts and failed-post retry.

Deliverable: `report5.md` and browser checks against fixtures and the
integrated API.

## Step 6 — deploy and evaluate the experiment

1. Read current service state through Nautobot/nctl. Back up affected stores
   and configuration; record code-and-data recovery instructions locally.
2. Deploy WFE, project workflows and aghangout through their established
   release paths. Account for outstanding deliveries at cutover. Update
   Nautobot records if deployment ownership or service placement changes.
3. Verify the deployed discussion and durable-record links from another
   LAN machine. Check existing direct conversations and Slack-to-Front.
4. Conduct the trials below, fix demonstrated failures and rerun affected
   checks. Use truthful automated-trial authorship.

| Trial | Evidence |
| --- | --- |
| Distinct perspectives | Two projects use their own context and respond to each other's actual contributions |
| Human participation | Direct questions, mid-flight clarification and later participation are attributed and reach their intended recipients |
| Open-ended discussion | Useful synthesis preserves disagreements and unresolved questions; closing/reopening does not require consensus |
| Membership and failure | A new participant can obtain the discussion context; an unavailable participant remains visible without blocking unrelated contributions |
| Recovery | Replay and bounded restart produce no duplicate invitations, messages or downstream requests |
| Durable memory | Published transcript matches its declared horizon; a fresh checkout contains the discussion and source-linked synthesis |
| Publication recovery | Concurrent exports and interrupted Git publication retain all records and accurately report what is published |
| Handoff and reuse | An implementing project cites a pinned argue record; a later discussion finds and uses prior knowledge |
| Responsiveness | Actual queue and execution times, including behavior while specialist work is active |
| LAN experience | Another machine can start/read a discussion and follow record/result links with minimal setup |

Evaluate whether interaction produced a new option, exposed a conflicting
assumption or changed a proposal. Cite the contributions that support that
assessment and record unproductive repetition too. A small trial demonstrates
behavior, not superiority over a single frontier model; a controlled model
comparison is optional future work. Do not require a positive creativity
finding to report the experiment honestly.

Complete repository checks appropriate to touched components. Update API
help, project guides, `devdocs/README_DEV.md` and ignored environment notes.
Commit/push intended changes and update parent pins where applicable,
preserving unrelated work.

Deliverable: `report6.md` with trial evidence and `report.md` with delivered
behavior, deployed revisions, review links, findings and remaining limits.

## Completion criteria

- A deployed discussion supports a person, a facilitator and at least two
  project agents speaking through their own contexts and runs.
- WFE owns delivery and execution; aghangout owns presentation; one managed
  argue repository owns cross-project synthesis and durable knowledge.
- Original speech, participant history, reply references and run provenance
  survive retries, concurrent contributions and restart.
- Discussion logs and source-linked synthesis are published in Git, with
  inspectable archival progress and proven recovery from publication failure.
- A project receives useful work with pinned provenance to argue, and a later
  discussion can discover and reuse accumulated knowledge.
- The UI supports direct human participation and access to records through
  the agdev entrance, verified from another LAN machine.
- Existing active clients continue to function; no obsolete compatibility
  machinery or human approval gate is introduced.
- The final report gives an evidence-based account of the experiment and
  direct review links. Optional feedback is separate from technical completion.
