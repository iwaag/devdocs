# slack_connect p1 — Slack as a Front conversation surface

## Objective

Let the Developer use the iPhone Slack app to give Front instructions, chat,
answer questions, steer ongoing work and receive reports. Keep the continuing
conversation and execution records in WFE, shared with the existing Front Desk.

Implement the integration inside agdevworld and run it as a separate persistent
process. Connect outbound to Slack using Socket Mode; ordinary conversation
must not require the iPhone to join the home LAN or the WFE service to be exposed
to the public internet. Existing Tailscale access serves detailed run and
artifact links where available.

This is an implementation plan, not a statement that the integration exists.
It follows the discussion after project_migration p8.

## Working policy

Follow `localrule.md`, `devpolicy/styles.md`, `devpolicy/terms.md`,
`devdocs/README_DEV.md`, applicable repository instructions and
`pj-agdev/.local/devenv.md`.

- Generated files are English; communication with the Developer is Japanese.
- Agents implement, configure, deploy and verify the work. Do not introduce
  human approval checkpoints for routine implementation, deployment or trials.
- Request only genuinely unavailable account access, credentials or information.
  Prepare everything possible before handing off an account-only action. This
  is an access dependency, not a request to approve the design or deployment.
- Preserve unrelated working changes. Commit/push the intended changes and
  update submodule pins as appropriate. Backward compatibility is not required.
- Consult Nautobot or nctl before service operations. Record observation ages
  and distinguish catalog state from a fresh runtime check.
- Keep host names, addresses, workspace/user IDs, credentials and raw operational
  evidence in ignored local files. Use portable references in tracked documents.
- Keep existing services and review surfaces accessible from the home LAN;
  do not introduce localhost-only restrictions on those surfaces.
- Deliver a deployed, easily reviewable experience. Ask for experiential
  feedback after preparing working entry points; do not substitute instructions
  for operational preparation the agent can perform.

## Scope

P1 delivers a private Slack app installed in one workspace, serving one
configured Developer through direct messages with the Front bot. A new
top-level DM starts a conversation; replies in its thread continue it.

Included:

- Bidirectional text, concurrent conversation threads and asynchronous reports.
- Continuity with Front Desk, including browser-originated messages in a
  Slack-linked conversation.
- Durable incoming events, outgoing delivery records and restart recovery.
- Clear question/result messages and working links to the corresponding
  conversation, run and artifacts.
- Free-plan operation without Slack AI, paid workflow features or Marketplace
  publication.
- iPhone keyboard dictation, which produces ordinary text without new backend
  speech processing.

Deferred: channel conversations, multiple users/workspaces, automatic bridging
of every existing Front Desk conversation, audio-clip transcription, synthesized
speech, file uploads to Slack and a general-purpose multi-chat adapter framework.
Unsupported audio/file-only input must receive a useful explanation, not an
empty Front request or silent omission. Adding a caption must not imply that
Front has inspected an attachment it cannot read.

## Architecture and ownership

| Component | Responsibility |
| --- | --- |
| WFE conversation facilities | Authoritative messages, attribution, submit-token deduplication, Front turn receipts, links to runs and replies |
| Front managed project | Interpret intent, answer, delegate, steer, ask and retain knowledge |
| Slack integration process | Maintain the Slack connection, receive/send messages, map threads and identities, recover delivery |
| Front Desk | Read and write the same conversations through the shared API |
| Integration's local store | Transport identities, inbox, outbox, delivery outcomes and recovery cursors; no second execution ledger |

Suggested source home: `agdevworld/integrations/slack/`. Confirm the repository's
packaging conventions before fixing the layout. Reuse its runtime and build
tooling; use the Slack SDK for connection lifecycle where appropriate.

The integration calls WFE's conversation API. It does not open WFE's SQLite
database or launch Front independently. Small API additions belong to the
shared conversation facilities. Credentials stay in ignored service
configuration, outside Front prompts and run artifacts.

Deploy independently of WFE and its workers. A Slack disconnect or integration
restart must not stop browser conversations or ongoing workflow runs. Register
the service and its dependencies using the established cluster facilities.

## Conversation behavior

### Identity and routing

- Map `(workspace ID, DM channel ID, root message timestamp)` to one WFE
  conversation ID. Keep Slack timestamps as strings.
- Map each Slack message to its WFE message, including browser messages echoed
  into Slack and Front replies posted by the integration.
- Accept human input only from the configured workspace and Developer Slack ID;
  map that identity to WFE's configured person. A display name is not identity.
  The current HTTP route records the configured person, so validate this mapping
  explicitly rather than treating arbitrary Slack input as that person.
- Use stable source-message tokens for WFE posts and Slack event IDs for inbox
  deduplication. Multiple event deliveries must start only one Front turn.
- New root messages start separate conversations. Thread replies use the root
  mapping even if other threads have newer activity.
- Thread membership identifies a conversation, not a particular pending
  question. Do not set WFE `re` to an arbitrary latest message. Use the existing
  Front interpretation and clarification behavior; add explicit question-target
  interaction only if a real trial demonstrates the need.
- Ignore bot-generated messages as human input. Browser-originated text is
  visibly attributed to the Developer via Front Desk when echoed by the bot.

### Presentation and continuity

Preserve the Developer's original text and source identity. Convert Slack
formatting only as necessary for useful plain text; retain the raw source in
transport evidence. Convert Front Markdown into readable Slack messages,
splitting long replies deterministically and providing a full-conversation
link. Do not run a second model merely to relay or summarize replies.

Post answers, questions and results in the originating thread. Coalesce repeated
progress updates without dropping substantive answers or questions. Include
accessible fallback text for formatted blocks. Clearly distinguish transport
receipt from Front starting work or completing it.

For Slack-linked conversations, mirror subsequent Front Desk human messages and
Front replies into the same thread, without mirroring already received Slack
messages back as new posts. Unlinked browser conversations remain unlinked.

Editing or deleting a Slack post does not undo work already requested. P1 does
not interpret such changes as new instructions. Explain that corrections and
cancellations should be sent as new messages, and preserve accepted WFE records.

Use existing reply kinds (`reply`, `question`, `progress`, `result`) to present
reports. This delivers reports Front actually records; it does not introduce
new scheduled reporting or promise a response when no worker can execute.

### Links and notifications

Resolve the current absolute-link behavior (`viewBase` / `answerBase`) against
an iPhone-reachable base. Verify conversation, run and artifact routes; do not
assume LAN `.local` names work over Tailscale. Keep URL construction consistent
with existing browser access rather than blindly replacing strings in text.

Text conversation must work with the iPhone VPN disabled. Detailed links may
require Tailscale, which must be made clear. If the existing tailnet route is
insufficient, prepare the smallest suitable route and record its configuration
locally; public exposure is not a prerequisite.

Use Slack's ordinary mobile notifications. Verify DM thread notifications on
the actual device, including when the app is backgrounded. Document the user's
Slack/OS notification settings where they affect delivery; do not claim that a
successful API post proves an iPhone notification appeared.

## Transport reliability

1. Persist valid received events before acknowledging them. Keep Slack
   acknowledgement independent of model execution and WFE availability; verify
   the SDK's actual acknowledgement behavior and deadline.
2. Process the inbox asynchronously, preserving order within each conversation.
   Retry WFE submissions with the same token. Advance processing state only
   after recording the result.
3. Read new WFE messages using `GET conversations/<id>?since=<id>`. Initially
   use bounded polling of linked conversations, with backoff on failure.
   Atomically enqueue outgoing messages with cursor advancement. Do not stop
   watching a conversation merely because one run completed or it was closed:
   late replies and browser messages must remain discoverable.
4. Persist outgoing messages with stable WFE IDs and deterministic chunk IDs.
   Record returned Slack timestamps. Respect rate limits and `Retry-After`.
5. Treat a timeout after sending to Slack as an unknown outcome. Reconcile
   against accessible thread history and durable delivery identity before
   retrying. Verify supported Slack metadata/API behavior; do not assert
   exactly-once Slack delivery based on a local outbox alone. Leave unresolved
   deliveries visible for recovery instead of silently dropping or repeatedly
   posting them.
6. Reconnect Socket Mode automatically. Do not assume Slack replays every event
   missed during downtime. Inspect available DM history/replies APIs, scopes
   and rate limits; backfill roots and mapped threads using overlap and source
   deduplication. Persist reconciliation progress and record any retention or
   permission gap that cannot be recovered.
7. Recover from WFE, Slack and process outages independently. Logs/status must
   distinguish disconnected, queued, delivered, failed and unknown outcomes,
   without logging tokens.

Slack history is a recovery aid, not the authoritative archive. WFE retains
accepted conversations independently of Slack free-plan history limits.

## Implementation steps

### Step 1 — inspect and prepare the connection

- Read current source, p8 reports, applicable instructions and local environment
  notes. Check repository status and deployed versions; inspect relevant
  services through Nautobot/nctl.
- Trace WFE write-origin checks and service access, reply cursors, person
  attribution and URL generation. Choose an explicit supported service-client
  path; do not bypass existing checks by fabricating browser headers.
- Prepare a versioned Slack app manifest for Socket Mode and DM message events.
  Configure the bot's message surface. Determine required scopes from the
  actual calls: connection opening, DM events/history/replies and message
  posting. Avoid subscriptions that deliver the same input through two routes.
- Provision the app using available account access. Store app/bot tokens in the
  ignored configuration. If a workspace/account action cannot be automated,
  provide the prepared manifest and exact remaining action, and continue all
  independent implementation work.
- Record final API/scopes, packaging and recovery choices in `report1.md`.

### Step 2 — implement the adapter and shared gaps

- Add the independently runnable integration, configuration validation,
  transport store, inbox/outbox and status/help.
- Implement DM/thread mapping, actor validation, formatting, browser mirroring,
  difference polling and receipt/error presentation.
- Implement reconnect, backfill and uncertain-send reconciliation. Expose
  actionable recovery operations for demonstrated failures.
- Add only required shared WFE API support and keep it documented in its
  existing API reference. Keep Front's guide transport-independent unless a
  demonstrated user-facing issue requires a change.
- Record changes and focused verification in `report2.md`.

### Step 3 — deploy and prove the experience

- Run the repository-required checks appropriate to changed code and the
  meaningful transport tests below. Do not add tests that merely mirror
  formatting implementation or repeatedly rerun unchanged broad suites.
- Deploy the WFE changes, if any, and the separate integration service. Register
  placement/state, configure restart behavior and verify runtime connectivity.
- Execute bounded real conversations using Front and existing managed projects.
  Identify automated trial actors honestly; do not attribute agent-written
  trial messages to the Developer without recording their actual provenance.
- Prepare direct Slack and Front Desk review links. Complete available
  end-to-end checks before requesting only the iPhone interactions that require
  the Developer's device. Device-only evidence remains explicitly pending if
  unavailable; do not describe server-side tests as mobile verification.
- Update developer docs and ignored environment notes, then write `report3.md`
  and a concise `report.md` with revisions, evidence, limitations and recovery.

## Verification and completion criteria

| Scenario | Required evidence |
| --- | --- |
| New root and thread reply | One continuing WFE conversation, one Front turn per accepted message, replies in the right Slack thread |
| Two concurrent threads | Follow-up and mid-flight steering reach the intended conversation/run |
| Question and answer | A real Front question reaches Slack and the answer continues the appropriate work; ambiguous references are not guessed by transport |
| Front Desk continuity | Slack history appears in the browser; browser text and Front replies appear once in the linked thread |
| Retry and restart | Duplicate events and crash windows do not duplicate WFE input or Front runs; queued output survives restart |
| Unknown Slack send | Simulated response loss is reconciled or exposed as unresolved, not blindly resent |
| Disconnection | Missed roots/thread messages are recovered within available history; unrecoverable gaps are explicit |
| Input boundaries | Bot echoes, unrelated users, edits/deletes and unsupported attachments do not trigger unintended turns |
| Delivery | Japanese text, long answers, questions/results and actual run/artifact links remain readable and correctly associated |
| iPhone | Text input and keyboard dictation work; an actual background notification is observed; text exchange works without VPN |
| Service isolation | Restarting the Slack integration does not interrupt Front Desk or running work |

Completion means a deployed usable integration, passing relevant automated and
live checks, documented operational recovery and honest status for any remaining
device/account dependency. No paid Slack feature is required for the delivered
scope. Existing Front scheduling limitations remain visible rather than being
misreported as Slack failures.

## References

- `devdocs/episodes/project_migration/p8/plan.md` and `report.md`.
- `pj-agdev/agdevworld/workflow_editor/docs/conversations.md`.
- [Slack Socket Mode](https://docs.slack.dev/apis/events-api/using-socket-mode/).
- [DM message events](https://docs.slack.dev/reference/events/message.im/).
- [Posting messages and thread replies](https://docs.slack.dev/reference/methods/chat.postMessage/).
- [Free-plan limits](https://slack.com/help/articles/27204752526611-Feature-limitations-on-the-free-version-of-Slack).

Recheck current Slack contracts during implementation, especially scopes,
history access and retry behavior; documentation research does not replace a
live test of the installed app.
