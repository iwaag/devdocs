# Workflow editor p19 — message file attachments

## Goal and scope

Deliver ordinary file and image attachments in conversations, including clipboard
image paste in aghangout. Every attachment must remain attributable to its exact
message and readable by every participating project, including agents executing
on another host. Preserve original bytes and make recovery verifiable.

Sources: [chatlog.md](chatlog.md) and [proposal.md](proposal.md). This plan adopts
the proposal's dedicated file attachment type and share-backed payload storage.
WFE owns storage, message association, delivery, retrieval and archive records.
The in-system aghangout agent owns its client implementation.

Writing this plan does not execute its implementation or commission agent work.

## Working conventions

Follow `localrule.md`, `devpolicy/styles.md`, `devpolicy/terms.md`,
`devdocs/README_DEV.md` and `pj-agdev/.local/devenv.md`. Generated documents and
code-facing text are English; communication with the Developer is Japanese.
Backward compatibility is not required. Preserve existing authoritative messages,
challenge pins and run histories rather than fabricating or discarding records.

During implementation, perform necessary commit/push, integration, deployment and
verification autonomously. Do not add a human permission milestone. Any agreement
record in an agent workflow records direction; it is not execution permission.
Omni may answer an in-system agent's permission request under `localrule.md`.

Keep machine addresses, credentials, mount paths, deployment commands with local
arguments and backup destinations in ignored environment notes. Inspect current
service/deployment state through Nautobot/nctl according to
`pj-clusterintent/nctl/README.md` before operating services. Deploy for LAN access,
verify from another PC and leave a direct, minimally interactive check example.

## 1. Trace the implementation and establish the contract

Read current WFE conversation, challenge, share, worker and run contracts and trace
their actual code paths before editing. In particular inspect:

- `server/conversations.ts`, `conversationSchema.ts`, `conversationService.ts`,
  `conversationArchive.ts` and the HTTP handlers for message creation.
- `shared/intake.ts`, `shared/run.ts`, `server/runs.ts`, `server/exec.ts`,
  `cli/conversation.ts` and worker forwarding/materialization.
- Share publication/download primitives, bounded challenge request parsing,
  untrusted-content serving and existing backup/migration commands.
- The current shared wfrun material reference and harness image-reading tools.

Record the discovered baseline revisions and local deployment facts in ignored
p19 notes. Publish the following contract in WFE documentation and fixtures so
the aghangout agent can work against a concrete interface:

| Surface | Implementation contract |
| --- | --- |
| Message submission | Extend `POST api/conversations/<id>/messages` with optional `files: [{name, base64, mime?}]`; keep existing routing and challenge inputs. |
| File-only submission | Permit empty text when files are present. Preserve actual text; use the first original filename as an untitled room's fallback label. |
| Attachment identity | `kind: file`, globally unique attachment id, bundle id, conversation, message, ordinal, original name, media type, byte count, full SHA-256 and logical reference. |
| Reads | Message responses/history contain attachment metadata and retrieval URLs alongside challenge attachments. |
| Retrieval | Add `GET/HEAD api/conversations/<id>/messages/<message>/attachments/<attachment>` with optional `download=1`. Validate the whole identity tuple. |
| CLI | Add `wfe conversation attachments <id> --message <n>` and `wfe conversation fetch <id> --message <n> --attachment <id> --out <path>`; support files in `conversation post` with repeatable `--file`. |
| Limits | Initially 20 files, 32 MiB per file and 64 MiB decoded total. Expose machine-readable limits to the client. |
| Errors | Distinguish invalid input, size exceeded, token conflict, unavailable storage, absent attachment and integrity failure. Preserve existing routing errors. |

Update exact syntax if existing command structure requires it, then update help,
docs and fixtures together. Contract publication is a work dependency, not an
approval gate. Avoid a temporary challenge-based upload implementation.

## 2. Implement durable attachment storage and submission

Configure an attachment destination by logical share and relative prefix in the
operating area's ignored configuration, initially `agshare`. Reuse each host's
`shares.json`; do not hardcode a mount path in tracked code or run records.

Publish bundles under:

```text
shared://<share>/conversations/<conversation>/attachments/<bundle-id>/
  manifest.json
  <attachment-id>
```

WFE allocates identifiers; original filenames are metadata rather than storage
paths. The versioned manifest lists ordered attachment metadata and hashes and
has a canonical digest. Distinguish separate occurrences even for identical bytes
or names. Do not overwrite published bundles or garbage-collect committed files.

Add a durable upload-operation record keyed by conversation and submit token.
Persist author, normalized submission digest, allocated identities, state,
manifest digest and accepted message identity. Include text, ordered file hashes
and metadata, challenges and explicit routing fields in the submission digest.
Determine challenge revisions once and persist those pins for recovery. Exclude
derived display URLs, host paths and request arrival timestamps from the digest.

Implement and test this sequence:

1. Validate metadata, base64, limits and destination. Reserve/reuse the operation
   under a database uniqueness constraint. Identical retries share one operation;
   changed content or author under that token returns a conflict.
2. Outside the SQLite write transaction, write into staging on the destination
   share, verify landed bytes, write the manifest and rename the complete bundle
   into place. Serialize concurrent work for the same operation using recoverable
   ownership; do not hold a database transaction across filesystem/network work.
3. Revalidate mutable routing state and create message, ordered attachment rows,
   response requests and the operation's accepted state in one transaction.
   Preserve the existing recipient-selection rules. A route that became invalid
   must not create a partially accepted message.
4. Dispatch only accepted requests, through existing receipt/reconcile behavior.
   Return the accepted message and attachments. A retry after a lost response
   returns the same accepted result without rerouting or republishing it.

Define operation states such as reserved, published, accepted and failed with
explicit retry transitions. On restart, adopt and verify a matching complete
bundle. An incomplete upload may require the client to resend its original
payload; expose that condition rather than pretending bytes are recoverable.
An unassociated bundle is retained for retry/diagnosis and is not a sent message.
Cleanup may remove only abandoned staging owned by a demonstrably inactive
operation. Never replace an attachment-bearing request with text-only delivery.

Bound encoded-body reads and decoded allocations, reject malformed encoding and
validate total decoded bytes. Reuse the challenge parser where appropriate.
Check proxy limits against base64 expansion and measure peak memory at the limit.
If the existing buffering approach is unsuitable, implement bounded streaming
without changing the public identity/retry semantics.

## 3. Extend reads, run delivery and image access

Make attachments a discriminated union of challenge and file across schema,
validation, reducers, request creation, source references, view formatting and
brief construction. Preserve existing challenge hashes and revision semantics.
Use file occurrence identity to avoid losing source-message associations during
aggregation; do not reuse challenge-id deduplication for files.

Carry file references through all existing paths: a new recipient run, a reply
to an open question, an explicit instruction to an existing run, an active or
idle standing conductor, and later requests that cite an attached message.
Reading history or fetching a file must not start a run.

The brief names each file, its source message and the commands/tools that read
it. The CLI reads local files for posting and writes fetched bytes on its calling
host, including execution workers. Resolve a local share when available and use
coordinator retrieval when a mount is absent. Hash-check fetched bytes before
publishing a local copy for use; retain downloads in ignored local storage.

Serve originals by attachment identity with correct lengths, safe content types,
download names and existing untrusted-content protections. Permit safe supported
image previews; keep active content inert or downloadable. A client MIME hint is
not authority to render active content. No upload executes the file or modifies
its original bytes. Ordinary metadata/history reads should not hash every file;
provide explicit integrity verification and report unavailable/missing/altered
states accurately.

Test the actual deployed harness's image-reading path. Update agautolab only if
needed for usable retrieval/tool access. Put shared attachment facts in the shared
material reference and syntax in CLI help. Give agents tools and useful context,
without requiring a fixed inspection sequence or adding repeated prompt rules.

## 4. Commission aghangout changes through its in-system agent

After step 1 publishes the contract/fixture, start the managed aghangout project's
normal WFE workflow request. Supply the user's desire, API contract, fixture,
limits, deployment expectations and evidence requirements. Record the request/run
identity in this episode's implementation report. Let that agent choose and
record its implementation graph and UI approach. It may develop against the
fixture while backend work continues.

Ask it to deliver clipboard image paste, file selection, drag/drop, pending-file
preview/removal, file-only sending, unchanged token/payload on retry, attachment
history display, inline supported images and original-file downloads. Preserve
normal text paste and recoverable composer contents after failed sends. Editing
the submission after a failure creates a new token; a retry does not.

Require its browser evidence and report to name changed revisions, actual checks,
artifacts and remaining limits. Omni reviews the artifacts and behavior, requests
corrections from the same owner as needed, and integrates the assessed result.
Do not implement or silently repair aghangout from outside the system. If a WFE
capability blocks the agent, fix that capability and let it continue. Resolve
routine permission questions autonomously under `localrule.md`.

## 5. Implement archive, export and payload recovery

Extend conversation transcript records with file associations and preserve bundle
manifests in the existing archive repository. Keep original bytes on the share;
archive publication status and payload verification are separate facts. Closing,
archiving or retiring a project never deletes its message attachments.

Extend conversation archive tooling with a self-contained export option that
includes the transcript, required manifests and original file payloads. Verify
hashes while packaging and when unpacking into an isolated recovery directory.
Include existing pinned-challenge recovery dependencies in its documentation;
do not imply that bundling message files embeds challenge repositories too.
Recovery must preserve message associations and must never replay deliveries.

Identify the existing payload backup mechanism. Extend it, or establish an
attachment backup, outside the live payload tree and document its failure domain
in ignored notes. Demonstrate restoration from that backup, not just rereading
live files. A control-area backup or archive Git clone alone is insufficient.
Document the portable restore procedure and the scope of bytes actually backed up.
If ponderer's archive readers need changes, commission those through their owning
in-system project and assess the delivered result.

## 6. Verify invariants and rehearse the transition

Use focused automated tests for the persistence/delivery behavior and the existing
required WFE checks. Verify these cases with fixtures and failure injection:

| Case | Required evidence |
| --- | --- |
| File-only first message; mixed text/files; adjacent duplicate filenames | Exact ordered message association, original bytes and correct fallback title. |
| Same retry; changed retry; concurrent identical retry | One accepted message and request set; changed content conflicts. |
| Crash during staging, after publication, after acceptance before response | Recoverable state, no text-only substitute, no duplicate request or overwrite. |
| Restart after acceptance before dispatch | Existing receipt/reconcile path dispatches accepted work once. |
| Destination changes while files publish | No partial acceptance or silent delivery to another destination. |
| New run, answer, instruction, conductor wake-up | Each receives the correct file occurrences and source provenance. |
| Two projects and a second execution host | Both obtain hash-identical bytes; fetch works without a local share mount. |
| Boundary sizes/counts, malformed input, share outage | Enforced limits and useful errors; ordinary text-only sending still works. |
| Unsupported/active media, missing/altered bytes | Safe download behavior and explicit failure, preserving metadata/history. |
| Archive/export and backup restore | Hash-identical originals and message links restored without execution. |
| Existing challenge attachments | Pins, run inputs, history and archive reads remain truthful. |

Run the repository's `npm run check` and the relevant conversation, run-delivery,
archive and worker/browser checks. Have the aghangout agent run its client suite.
Repeat or broaden checks only when a change, failure or unresolved concern calls
for it. Do not treat fixture checks as proof of deployed image access.

Rehearse schema/record migration on a copy of current data and run WFE's run
replay/check command for each affected workspace. Migration creates no messages,
agent requests or synthetic attachment provenance for old records. Prepare a
matching code/control-data backup and identify the first new record an older
release cannot read. Backward compatibility shims are not required.

## 7. Deploy, exercise and hand over

Inspect current deployment state, commit/push the finished repositories and deploy
the coordinator, compatible worker WFE versions and any necessary executor/guide
changes. Deploy the assessed aghangout result through its owning workflow or its
documented deployment path. No human approval step is needed. Use managed host
configuration for pins and read back actual running revisions and service health.

Keep the LAN entrance reachable and preserve share availability. Record a brief
write pause if required for a consistent control-data backup/migration. Verify
service operation and historical record replay before resuming normal intake.
After accepting new-version messages, recovery must preserve them; do not restore
an old database over new records or assume a code-only rollback is sufficient.

Use a clearly attributed check conversation and actual agent requests. From the
deployed aghangout, paste an image containing an unpredictable visible feature
not described by its filename/text, and send a separate ordinary file. Have two
projects retrieve the attachments and an actual receiving agent describe the
image feature. Exercise a continuing conductor and an existing-run delivery.
Check from another LAN PC, inspect message associations/download hashes, archive
the conversation and verify the exported/restored bytes. Inspect agent reports
and resulting artifacts before recording acceptance.

Leave the deployed check conversation directly accessible so the Developer can
inspect the pasted image, original download and agent response with few actions.
Write `p19/report.md` with changes, revisions, delegated-work disposition, actual
tests, deployed evidence, backup/restore results, open limits and recovery boundary.
Update `devdocs/README_DEV.md`, relevant WFE documentation and ignored
`pj-agdev/.local/devenv.md` to match the delivered behavior. Keep local evidence
and commands in ignored p19 notes and link their role from the report.

## Completion criteria

The deployed LAN client accepts pasted images and ordinary files, including files
without text. Every occurrence remains tied to its exact message. Participating
projects and remote agents can retrieve verified originals, and a real agent can
inspect image content. Retry/crash recovery preserves one accepted message and
request set. Archive and payload backup recovery preserve bytes and provenance.
aghangout was implemented by its in-system agent, its result was assessed, and
the Developer has a working example to inspect immediately.
