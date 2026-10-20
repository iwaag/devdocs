# Workflow editor p15 — aghangout

## Objective

Implement [braindump.md](braindump.md) by establishing `aghangout` as an
ordinary managed project: a place to casually meet and talk directly with
characterized project agents. Initially support Front, forge and archsage.
Reuse the useful visual-novel presentation of the existing Front Desk while
making the application independently developable and deployable.

WFE owns durable conversations and workflow execution. aghangout owns the
experience of talking with those agents. Other clients, including a future
XR application, should be able to use the same conversation contract without
copying execution logic or requiring a new agdevworld UI implementation.

This plan does not start implementation.

## Working conventions

- Follow `localrule.md`: generated files are English and communication with the
  Developer is Japanese. Implementation, trials, deployment and appropriate
  commits/pushes proceed without additional human approval checkpoints.
  Records of review or acceptance are not permission gates.
- Prepare a deployed result accessible from other home-LAN machines, with
  direct review links and minimal manual setup. The agent performs the
  verification and operational steps it can perform itself.
- Backward compatibility is unnecessary. Update active clients together;
  do not retain obsolete routes, schemas or adapters solely for compatibility.
- Do not migrate Zulip-core-era conversations or build a historical reader
  for them. Workflow-era history is optional, not a completion criterion.
  Retain it if the existing store can be reused cheaply; otherwise start fresh.
  Historical cleanup is not a reason to delete unrelated source records.
- Keep machine-specific configuration, addresses, credentials and raw runtime
  evidence in ignored local files. Tracked documents use portable references.
- Before inspecting or changing local deployments, obtain current service
  state through Nautobot/nctl, following `pj-clusterintent/nctl/README.md`.
  Environment notes are pointers, not live-state evidence.
- Preserve unrelated edits and avoid interrupting unrelated active work.
  Distinguish historical retention from live delivery obligations.
- Give project agents discoverable tools and concise role descriptions.
  Keep tool syntax in help and shared facts in their owning documentation;
  avoid scripted decision chains or approval requests for routine work.

## Scope and ownership

| Owner | Responsibilities |
| --- | --- |
| WFE / agdevworld | Conversation identity, destination, messages, attribution, run links, dispatch, reply correlation, execution state and recovery |
| Managed project `aghangout` | Character selection, rooms, visual presentation, assets, playback, conversation navigation, input and client preferences |
| Managed projects `front`, `forge`, `archsage` | Conversational behavior, professional responsibilities, knowledge, work selection and replies |
| Deployment configuration | Persistent application hosting, API connectivity and links from Home |

The managed project is named `aghangout` (repository `pj-aghangout` under
the normal project creation convention). Autolab remains execution
infrastructure and is not a conversation character.

The first release includes character selection, conversation creation and
navigation, posting and reading replies, follow-up instructions, questions and
answers, current work status and result/artifact links. Preserve useful
existing artwork and room presentation. Settings necessary to use these
features belong in aghangout; WFE remains available for detailed administration.

XR, a theme marketplace, a general plugin framework, multi-agent group chat and
new character artwork are not required. The architecture must permit a
different presentation client without making these future features now.

## Conversation and client contract

- A conversation records its destination project and conversational workflow.
  Dispatch uses that recorded destination, including after restart or a
  configuration change. Existing configuration currently names one global
  entrance; generalize it without embedding the three project names in the
  conversation engine.
- Expose available conversation targets through a documented contract.
  Project/workflow identity belongs to the service; character names, artwork,
  backgrounds and presentation mappings belong to aghangout.
- Choosing another character opens or creates a conversation for that
  character. It does not silently retarget an existing conversation. A missing
  or unavailable destination is visible and never falls back to Front.
- Keep conversation identity separate from any one run. Preserve submission
  idempotency, durable dispatch receipts, reply provenance and references to
  the messages being answered. Concurrent replies must remain attributable.
- The UI posts the person's words. The target agent acts and replies through
  its own ordinary runs. Visual identity must not replace the actual author
  or execution identity.
- Reuse WFE's run instructions, answers, delegation, waits and result links.
  Do not add a parallel execution engine or a second authoritative message
  store in aghangout.
- Use current WFE execution evidence for progress. Do not carry forward the
  old Zulip mirror as the new application's progress source.
- Document the browser connection contract, including write-origin handling,
  configured user identity and asset/result access. Prefer a same-origin
  reverse-proxy deployment initially; keep the API base configurable so
  independent hosting is possible. Do not expose service credentials in
  browser bundles or bypass origin handling through accidental behavior.

## Step 1 — inventory and settle the implementation boundary

1. Inspect the current Front Desk scene, input, playback, character/settings
   sources, room adapters and remaining relay dependencies.
2. Map useful behavior to aghangout, a common WFE API, or an existing WFE
   administration link. In particular resolve progress, Context, settings,
   character assets and result rendering.
3. Inspect conversation store/HTTP/CLI contracts, Front's conversational
   workflow, forge/archsage workflows and current execution-lane behavior.
   Identify which parts are already general and which assume one entrance.
4. Inventory active clients and links, especially Slack. Choose the cheapest
   workflow-history treatment and record it. Do not spend effort importing
   Zulip-era conversations. Account separately for active runs, pending
   replies and Slack thread bindings.
5. Identify the ordinary managed-project creation and permanent hosting
   paths. Record needed client operations and their current access behavior;
   prove access through those paths instead of assuming Front's privileges
   automatically imply browser access.

Deliverable: `report1.md` with the feature/owner map, API gaps, history
decision and deployment approach.

## Step 2 — generalize the WFE conversation surface

1. Add durable per-conversation destinations and target discovery/configuration.
   Update HTTP and CLI together. Select an existing destination explicitly
   for active clients such as Slack.
2. Route messages and recovery by the conversation's recorded destination.
   Keep dispatch receipts stable and make invalid destinations diagnosable.
3. Expose the message, question, linked-run, execution-status and artifact
   information needed by an independent client, reusing existing APIs.
4. Verify client identity, actual reply author, write-origin behavior and
   result links through the planned deployment route.
5. Apply the Step 1 store decision. If retaining workflow-era rows, map their
   existing Front destination once rather than maintaining two schemas.
   Any fresh-store cutover must explicitly handle active delivery obligations.
6. Update API documentation and CLI help so a second client can be implemented
   without importing server internals or querying SQLite directly.

Deliverable: `report2.md`, updated contracts and focused tests for routing,
duplicates, concurrent replies, missing destinations and restart recovery.

## Step 3 — provide direct conversations with the three projects

1. Keep Front on its ordinary managed conversational workflow, adapting it to
   the generalized contract where necessary.
2. Add conversational workflows to forge and archsage. Each reads its
   conversation and linked work, answers within its role, and can initiate
   or delegate appropriate specialist work using ordinary WFE facilities.
3. Return questions, progress and results to the original conversation.
   Carry follow-up instructions and answers to the intended work with
   source-message attribution.
4. Put common conversation-tool facts in shared help/documentation and
   project responsibilities in their own guides. Keep character presentation
   settings separate from professional role instructions.
5. Measure queueing and responsiveness while specialist work runs. Reuse the
   existing conversation lane where appropriate; adjust scheduling only when
   needed to keep the three conversation targets usable.

Deliverable: `report3.md` and bounded trials proving all three agents answer
directly, with at least one specialist-work/question/result round trip.

## Step 4 — build the managed aghangout application

1. Create `aghangout` through the normal managed-project path, with project
   intent, development guidance, workflows and build/run documentation.
2. Move useful scene, input, playback and asset code into the project.
   Remove obsolete room adapters and assumptions about a single Front agent.
3. Provide a clear choice of Front, forge and archsage, with their existing
   character presentation. Make the current conversation partner evident
   throughout navigation and posting.
4. Implement conversations, replies, pending questions, follow-ups, linked
   work status and artifact access against the common API.
5. Separate API data handling from scene presentation. Keep user display
   preferences in the client/project layer and authoritative work state in
   WFE. Render ordinary text and links usefully when no special presentation
   metadata exists.
6. Replace relay-backed panels with current information or direct WFE links,
   according to the feature map. Include visible loading, failed-post,
   retry and unavailable-target states without losing unsent input.
7. Verify desktop and narrow layouts, readable long replies, keyboard input,
   navigation and recovery after page reload.

Deliverable: `report4.md`, the managed application and browser verification
against isolated fixtures before live integration.

## Step 5 — deploy and retire the embedded Front Desk

1. Inspect deployment state through Nautobot/nctl. Back up affected current
   stores/configuration and record a coherent code-and-data recovery path
   in ignored notes.
2. Deploy WFE changes, project workflows and aghangout. Serve aghangout
   persistently beyond the lifetime of a development run, accessible over
   the LAN, using the selected API/proxy configuration.
3. Add the aghangout entrance to WFE Home and update active generated links
   and Slack link configuration where relevant. Legacy URL compatibility
   is optional and must not become a migration subproject.
4. Cut over active delivery ownership without duplicate dispatch or stranded
   callbacks. Historical conversations need no reconstruction.
5. Remove the superseded embedded Front Desk application and its exclusive
   dependencies after the replacement works. No Arguing Room history viewer
   is required. Retire its dedicated path if it has no current consumer.
6. Review remaining relay consumers before removing shared services. Usage
   or other active functions may still need them; keep only dependencies
   justified by current consumers.
7. Record the service placement in Nautobot/nctl and verify actual deployed
   browser/API behavior from another LAN machine.

Deliverable: `report5.md`, deployed revisions, runtime evidence and direct
review links. A converged placement record alone does not prove the UI works.

## Step 6 — prove the experience and close

| Trial | Required evidence |
| --- | --- |
| Three conversation partners | Each selected project receives and answers its own conversation, without an intermediary Front run for forge or archsage |
| Useful work | A request produces inspectable specialist output and a reply linked to the work that produced it |
| Questions and steering | Answers and mid-flight changes reach the intended work with their original message attribution |
| Concurrent conversations | Switching characters or conversations does not misroute messages or replies |
| Responsiveness | A conversation receives service while another request's specialist work remains active; record actual timing and limitations |
| Replay and recovery | Retry, reload and bounded restart do not duplicate messages, work or replies |
| Unavailability | Missing target or failed dispatch is visible and recoverable without silently changing the recipient |
| Live clients | Slack's active Front path and WFE links still work with the deployed conversation contract |
| Independent application | aghangout can change and redeploy its presentation without rebuilding agdevworld |
| LAN review | Another machine can open the prepared entrance and inspect conversations and artifacts |

Use fixtures for failure/race cases and bounded live trials for actual agent
behavior. Identify automated trial authors truthfully. Record any Omni
intervention in agent-owned work rather than presenting it as agent autonomy.
Fix demonstrated failures and rerun affected checks; complete repository
checks appropriate to the touched components.

Update developer documentation, API help, project guides and ignored
environment notes to match the delivered ownership. Commit/push intended
changes and update parent pins where applicable, preserving unrelated edits.
Write `report6.md` for trials and `report.md` for the final outcome, evidence,
review links and remaining limitations.

## Completion criteria

- `aghangout` is an ordinary managed project with an independently deployed
  application and a documented development path.
- Front, forge and archsage are directly reachable as distinct conversation
  partners with useful character presentation; autolab is not offered.
- WFE owns conversations and execution through a documented client contract
  that supports multiple project destinations without UI-specific routing.
- Requests, follow-ups, questions, progress and results remain correctly
  correlated across concurrency, retries and restart.
- The application uses current workflow evidence and does not depend on a
  Zulip-era conversation import or historical progress model.
- No unnecessary backward-compatibility layer or human approval gate is added.
- Active client delivery remains accounted for; optional history treatment
  and removed legacy surfaces are documented.
- The embedded Front Desk is superseded, shared dependencies have explicit
  remaining owners, and the result is deployed and verified over the LAN.
- The Developer can inspect the result through prepared links with minimal
  actions. Pending optional feedback is separate from technical completion.
