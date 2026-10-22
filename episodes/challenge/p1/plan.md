# Challenge p1 implementation plan

## Objective

Preserve problems discovered during project work as durable, shared challenges.
A person or agent can record one from a conversation, find it from another
project, and include it in a later conversation or discussion with little effort.
Completing the original work does not resolve its remaining challenges.

This plan reflects `braindump.md` and the subsequent discussion with the
Developer, including the decisions to use free-form text and Git as the durable
source of truth.

## Scope and sequencing

Deliver challenge management through WFE and aghangout first. Both existing
chat and discussion interfaces receive the same challenge capabilities.
Unifying their conversation model and UI is a subsequent phase, described below;
it is not a prerequisite for delivering p1.

Follow `../../../../localrule.md`: generated files are English, communication
with the Developer is Japanese, backward compatibility is not required, and
implementation and deployment do not require human approval gates. Deliver a
deployed result that is easy to inspect from another LAN computer. Keep host
addresses, credentials, and machine-specific paths in ignored local records.
Before operating on live services, establish their state through Nautobot or
`pj-clusterintent/nctl`, following its README.

## Data and ownership

### Minimal record

Store each challenge as `challenges/<id>.md` in one shared Git repository.
Use frontmatter for only these management fields:

- `id`: an automatically allocated, immutable identity.
- `title`: a human-readable title.
- `state`: `open`, `resolved`, or `withdrawn`.
- `projects`: related project identities, used for discovery and filtering.

The Markdown body is free-form. It may explain the problem, desired outcome,
history, sources, possible solutions, experiments, and resolution evidence.
No mandatory headings, problem taxonomy, or template is imposed. Related projects
need not be divided into formal roles; explain their involvement in the body
when useful. Creation and edit attribution, times, and revision history are
recorded automatically, not entered as form fields.

A challenge is independent of any conversation, project lifecycle, or run.
Multiple projects and runs may investigate it. A completed run does not
automatically resolve it. Resolved and withdrawn records remain readable and
searchable, and can be reopened. Renaming a title does not change its identity.

### Durable source of truth

Create a shared repository, provisionally `pj-challenges`, in the existing Git
service. This is a data repository; p1 does not need a dedicated agent, workflow
project, or new service. WFE provides its editing and reading interfaces.

Git is authoritative. SQLite may index published records for search, but the
index must be rebuildable from the repository. Do not maintain independently
editable Git and SQLite versions of a challenge.

WFE uses a dedicated writer checkout and serializes its own writes. Each accepted
edit records the actual actor and originating run/message where available, and
automatically commits and pushes. Attribute agent edits to the agent, even when
made on the Developer's behalf. Keep attribution with the durable Git history.

Use base-revision checks to expose conflicting edits without losing a draft.
Use request identities to deduplicate repeated creates and saves. Preserve a
pending operation durably until publication is confirmed. A push failure must
show an unsynchronized state and retain the content for retry; it must not be
reported as durable publication. After a crash or an ambiguous push outcome,
inspect the remote history before retrying. Handle remote changes without
force-pushing or silently overwriting another edit.

Preserve enough explanation in the body for the challenge to remain useful if
its original conversation or run is removed. Store essential evidence files
under `evidence/<id>/` in the repository, referenced with relative paths. Assess
existing Git large-file support before copying large evidence; any separate
storage chosen must have a durable retention and backup policy. A temporary
presentation URL is not sufficient evidence preservation. Record and verify the
repository's backup/recovery arrangement as part of deployment.

## Implementation steps

### 1. Establish the contract and storage

- Inspect the current WFE Git integration, context snapshots, conversation store,
  archive writer, and aghangout code before selecting reusable pieces.
- Provision and configure the shared repository; keep environment-specific
  repository configuration outside tracked generic documentation.
- Document the Markdown format, identities, actor attribution, revision
  semantics, publication states, evidence paths, and recovery behavior.
- Implement repository reads, serialized publication, persistent pending
  operations, conflict handling, and an index that can be rebuilt from Git.
- Preserve records for retired or unregistered projects; project availability
  must not determine whether historical challenges can be read.

### 2. Expose WFE API and CLI

Provide equivalent HTTP and CLI operations for create, list/search, show, edit,
resolve, withdraw, reopen, and history. Proposed command group:
`wfe challenge`. Include project and state filters and ordinary text search.
All projects, including runs on remote execution nodes, use the same service
contract and shared identities.

Return the published commit with a record. Distinguish a current read from a
read pinned to a commit. Expose pending publication and conflicts clearly.
Status changes are ordinary versioned edits; keep explanations in the body
rather than adding a separate collection of required fields.

Creating or viewing a challenge starts no agent work. The API provides tools;
it does not assign work, mandate a review process, or automatically interpret
conversation text as a new challenge.

### 3. Carry challenge references into work

- Add structured challenge attachments to both conversation and discussion
  messages. Persist the challenge ID and published commit at send time.
- Provide the receiving run with the pinned title and body, and a route to its
  evidence. Keep large attachments available on demand.
- Record what the run received in its input/snapshot. A later edit must not
  change an earlier turn's meaning; latest content is a separate explicit read.
- Preserve message attachment references in applicable conversation exports and
  discussion archives. Keep references readable if a source project retires.
- Support explicit links to challenges in ordinary work requests as well as
  interactive conversations.

Project context may carry a short pointer to a relevant challenge. It should
not become a second editable copy of the challenge body or an automatic dump of
every open problem.

### 4. Add aghangout discovery and editing

Build shared challenge components usable in both chat and argue:

- A cross-project list with text, project, and state filters.
- A detail view with the body, evidence, history, and source links.
- A simple editor for title, projects, state, and Markdown body.
- An action to record a challenge from a selected message, retaining its source.
- An action to attach a challenge to the current message draft.
- An action to start a discussion with a challenge attached.

Show the attached title and allow removal before sending. Show publication
failures and preserve edits for retry. Distinguish the attached historical
version from the latest version. Opening a list or adding an attachment to a
draft does not launch a model.

### 5. Teach the tools and migrate the first real example

Add concise discovery guidance and `--help` references to the relevant shared
agent guide. Explain when a durable challenge is useful without prescribing
mandatory extraction, triage, or resolution steps for every run. Front and
project agents may record problems; ponderer can use them as discussion material.

Use the object-extraction boundary problem from the conversation started by
`front/converse/run-053` as the first substantive record:

- The Developer's feedback is message #310 of conversation
  `20261022-230244-zlcew6`; Front's recording response is #311.
- The problem is residual surrounding image content on extracted objects,
  producing visible artifacts during compositing and animation.
- Preserve the relevant findings and evidence from
  `gimmickimg/freework/run-006`. Inspect the existing `improvement-feedback`
  context block before replacing duplicated material with a challenge pointer.
- Keep the challenge open and the original work complete. Improving the image
  extraction itself is outside this implementation episode.

### 6. Verify, deploy, and report

Use focused automated checks for the storage and delivery contracts:

- Concurrent edits conflict without losing either draft.
- Repeated requests and restart recovery do not duplicate records or edits.
- Push failure, and a crash after a successful push, recover with correct
  publication status.
- Rebuilding the index from a fresh clone retains records and history.
- An attachment still resolves the original content after subsequent edits.
- Both chat and discussion runs receive their attached challenge revision.

Use disposable fixtures for destructive and failure-path tests. Verify the real
example through the deployed UI: record/read it, find it from another project's
view, attach it, and inspect the receiving run's input. Use an explicitly
identified test actor for any automated conversation posts; do not attribute
them to the Developer. Verify ordinary reads and saves create no unsolicited
runs. Confirm resolved records remain discoverable using a fixture.

Deploy WFE and aghangout using their existing release mechanisms. Capture the
pre-change service state through Nautobot/nctl, back up affected live stores,
and document recovery of code, pending publications, configuration, and data.
Verify the deployed result from another LAN computer and record post-deployment
service evidence. Keep local logs, addresses, and screenshots in ignored local
storage; put portable findings in `report.md` and update development docs.

## p1 completion criteria

- A challenge can be recorded from a conversation without filling a detailed
  structured questionnaire.
- A successful publication is present in the shared remote Git repository,
  including the history and essential evidence needed to understand it.
- Another project can discover and read it through the same API/CLI and UI.
- Both chat and argue can attach it to a message, and the receiving run sees
  the recorded revision.
- The first real challenge is preserved with its source and evidence; its
  project context points to that record.
- The deployed LAN UI supports the whole flow with few operations, and the
  report includes verification evidence and recovery instructions.

## Subsequent phase: unify chat and argue

After p1 is usable, create a separate implementation plan for a common
conversation model with optional facilitation. Existing chat and discussions
already share message storage, but their dispatch behavior differs: chat starts
a turn per human message, while discussions route through invitations and may
continue an existing conductor or question-owning run.

Unify participants, messages, attachments, response requests, and delivery.
A one-to-one conversation has a default addressee; a facilitated conversation
has a conductor such as ponderer. Ponderer's presence alone does not define
facilitation. Allow a conversation to gain participants or a conductor without
losing its identity or history, then consolidate the UI around that model.

That phase must cover question answers, instructions to active work, duplicate
dispatch prevention, late results, archive ownership for unfacilitated chats,
Slack, and migration of existing conversations. Preserve authors, message IDs,
run references, and challenge revision references. Backward-compatible APIs
are not required; deliberate data migration and coordinated client updates are.
Reuse the challenge components delivered in p1.
