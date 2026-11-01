# Message file attachments — proposed WFE direction

Status: proposal; no implementation, deployment or agent request made by this document.
Source: [chatlog.md](chatlog.md). The Developer wants aghangout implementation
performed by in-system agents.

## Recommendation

Add ordinary file attachments as a first-class conversation capability owned by
WFE. Keep original bytes on a configured share, initially `agshare`; keep their
identity, hashes and message association in WFE and conversation archives.
Use the existing routing and delivery engine. aghangout owns picking, pasting,
previewing and displaying those files.

This adopts ponderer's dedicated-attachment direction, with two changes:
payloads use shared storage instead of a new shared Git repository, and storage
publication and message acceptance have explicit recovery semantics. Neither a
single HTTP request nor a Git commit makes filesystem and SQLite writes atomic.
Do not introduce challenges as an interim file container.

Shared storage already provides logical references, host-specific resolution and
downloads. It avoids making every archive/repository clone carry growing binary
history. Git remains a viable alternative when clone-contained recovery is more
valuable than that cost; it is not technically prohibited. With the proposed
share layout, a Git clone alone does not restore attachment bytes.

## Existing implementation and required extensions

- `server/conversations.ts`: attachment rows, views and pinning are challenge-only.
- `shared/intake.ts`, `shared/run.ts`, `server/runs.ts`: references, validation,
  labels and run input construction likewise assume challenges.
- `server/conversationService.ts`: message and request rows are created in one
  SQLite transaction, followed by dispatch. New runs and answers/instructions to
  existing runs already receive attachment references, through separate paths.
  Empty message text is currently rejected.
- `server/exec.ts`: initial and continuation briefs explain challenge access;
  merely widening an HTTP type will not make files available to agents.
- `server/conversationArchive.ts`: archives embed message attachment references.
- `docs/shares.md` and `server/shares.ts`: share resolution, staging, hashing and
  publication primitives exist. A conversation upload is not a run output and
  needs its own owner, manifest and publication logic.

## Storage and identity

Proposed logical layout (not a host-specific absolute path):

```text
shared://agshare/conversations/<conversation>/attachments/<bundle-id>/
  manifest.json
  <attachment-id>            original bytes; display name is metadata
```

WFE allocates bundle and attachment identifiers; names supplied by a client do
not become storage paths. Each attachment keeps `kind: file`, id, original name,
media type, byte count, full SHA-256 and logical reference. The bundle manifest
lists these records and has its own digest. Message association rows keep the
conversation, message id, attachment id and order. Public attachment records and
archive records preserve that provenance too.

Allocate the bundle before accepting the message. Its path need not contain a
message number: the authoritative association is committed with the message,
not inferred from names or directory placement. Identical names, identical bytes
in different messages, and several attachments in one message remain distinct
occurrences. Do not deduplicate away their source-message identity when building
run inputs.

Published bundles are never overwritten by WFE. SHA-256 detects corruption or
out-of-band changes on the share; NFS itself does not guarantee immutability.
Retain committed attachments after conversation closure, archival and project
retirement. Avoid automatic payload deletion in this phase.

## Submission contract and failure recovery

Start with the existing JSON message route plus optional
`files: [{name, base64, mime?}]`. This matches the existing challenge transport
and keeps the first client contract small. Return attachment metadata and URLs
with the accepted message; expose the same shape in history and CLI reads.
Allow empty text when at least one file exists; preserve the person's actual
text rather than inventing a placeholder message. Check first-message title
fallback and routing for this case too.

Proposed initial limits: 32 MiB per file, 64 MiB decoded total per message,
20 files. Publish the limits for clients. Enforce encoded request and decoded
limits before excessive allocation; account for base64 expansion in both WFE's
body reader and the proxy. Reuse the bounded challenge reader where suitable.
Streaming/multipart is a later transport improvement if memory/latency checks
show a need; it does not change attachment identity.

Acceptance sequence:

1. Validate files and destination; find or durably reserve an upload operation
   by conversation and submit token, with a digest of the submitted content and
   routing fields. The same token with different content is a conflict.
2. Write a staging directory on the destination share, verify sizes and hashes,
   persist the manifest, and publish the complete bundle by rename. Recovery
   must distinguish a completed matching bundle from an incomplete write.
3. Commit message, ordered attachment associations and response requests in one
   SQLite transaction. Only then expose the message and dispatch its requests.
4. A retry or restart recovers the operation and returns the same message and
   requests. Concurrent retries cannot produce two accepted bundles/messages.

A crash after file publication but before the SQLite commit may leave an
unassociated bundle. That is recoverable storage, not a sent message. A failed
upload must not send a text-only substitute. Test interruption at both boundaries
and after commit before the response. Share unavailability is an explicit error;
text-only conversations continue using their existing path.

## Access by projects, runs and browsers

- Every participating project can list and fetch a message's attachments by
  conversation/message/id. Reading does not depend on being the original
  recipient, owning its run, or knowing another checkout's path.
- Add CLI list/fetch operations under `wfe conversation` with `--out` writing
  on the calling execution host, analogous to remote `wfe challenge` operations.
  Reuse share resolution for hosts with a mount and coordinator HTTP retrieval
  when a mount is absent. Verify the selected bytes against their recorded hash.
- Extend `attachments` to a discriminated union of challenge and file throughout
  validation, storage, views, archives, run creation and brief rendering.
  New runs, replies to questions, explicit instructions and standing-conductor
  wake-ups must all preserve source message and attachment metadata.
- Briefs name each file, its source and how to fetch/open it. Downloaded copies
  belong in ignored local storage, not run Git history. An agent may inspect or
  pass them to another tool as its task requires; do not mandate an analysis chain.
- Prove that an actual receiving agent can inspect an attached image. A URL in a
  text prompt is not evidence of visual access. Check the deployed harness/tool
  path and change the executor only where needed for that access.
- Serve browser downloads by attachment identity, without exposing host paths.
  Permit image previews for supported safe image types; serve active content
  inertly/as downloads, with existing untrusted-content response protections.
  Support arbitrary file storage; uploading never executes a file. Keep original
  bytes; any future thumbnail or conversion is a separately identified derivative.

## Archive, backup and deployment

Archive the complete message associations and bundle manifests in the existing
conversation archive repository. Keep payloads on the share. Document the
two-part restore requirement: archive/control records and attachment bytes.
Provide a way to export a conversation with its referenced originals for a
self-contained recovery package, checking hashes on export and restoration.
Restoration must not dispatch historical requests.

Establish attachment payload backup and restore verification as part of delivery;
neither the existing conversation Git archive nor a control-area backup can be
assumed to copy shares. Report archive publication and payload verification as
separate facts. Missing/unavailable/altered bytes must remain visible.

The schema/record change needs a backup, migration rehearsal, replay checks and
a documented recovery/rollback boundary. Old challenge pins remain truthful;
compatibility shims for older binaries are not a requirement.

## Work split and delivery order

1. WFE: implement storage/recovery, message and retrieval APIs, attachment union,
   CLI and both run-delivery paths. Publish the API shapes, error behavior,
   limits and fixtures in `docs/conversations.md` and command help. Update the
   shared agent material reference only where its factual contract changes.
2. In-system aghangout agent: receive the desired behavior and WFE contract;
   implement clipboard image paste, file selection and drag/drop, pending-file
   preview/removal, file-only submission, retry with the unchanged token/payload,
   image display and file download links. Let the agent choose its implementation
   and work graph. UI work can start against the agreed fixture while WFE is built.
3. WFE/archive work: add manifest export and payload recovery support. If ponderer
   archive readers need changes, request those changes from their in-system owner.
4. Integration: deploy compatible backend/worker versions, then the agent-produced
   aghangout result; verify from a second host and a real conversation. Inspect
   the agent's report and artifacts before reporting the result as complete.

This proposal does not itself commission the client work. On implementation,
send the task through the project's normal WFE entrance. Omni handles WFE and
integration; aghangout changes stay with its in-system agent.

## Evidence required for completion

- File-only paste and mixed text/files bind to exactly the accepted message;
  same names on adjacent messages cannot overwrite or exchange attachments.
- Two projects can obtain identical verified bytes, including from another
  execution host; an actual agent describes a visual feature of the test image.
- Existing-run answers/instructions and conductor wake-ups receive the files,
  as do newly started runs. Reading alone wakes nobody.
- Identical retry, conflicting retry, concurrent submissions and injected crashes
  preserve one message/request set; failed uploads deliver no partial message.
- Size limits, unavailable storage and altered/missing bytes produce useful errors.
- A fresh recovery package restores transcript, manifest and original bytes with
  matching hashes, without starting runs. Existing challenge attachment checks pass.

Implementation validation should include relevant store/delivery/archive tests,
the required WFE checks and agent-produced browser evidence. No runtime checks
were performed while preparing this proposal.
