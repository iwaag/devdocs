# workflow_editor p16 — Step 2: the WFE discussion surface

The model settled in [report1](report1.md) is implemented in WFE, with a
documented HTTP and CLI contract and focused tests. Commits: agdevworld
`ded957c`, agautolab `6c17e17` (wfrun guide), pinned in pj-agdev.

## What was built

| Part | Where | What |
| --- | --- | --- |
| Store v4 | `server/discussionSchema.ts`, `server/conversations.ts` | `conversations.kind`; `messages.participant`, `.invitation`, `.addressed`; tables `membership` (append-only), `invitations`, `invitation_events`, `archives`. Additive migration; the conversation routes and CLI refuse a discussion (409 `is-discussion`) and conversation lists leave discussions out. Relation `handoff`, message kind `synthesis` |
| Discussions | `server/discussions.ts` | Create (creator, facilitator, named projects join); join/leave history; routing of the person's message (addressed → those named; nobody/facilitator → the facilitator); invitations with receipt `discussion:<id>:invitation:<n>`, horizon, outcome and events; contributions attributed to participant, run, attempt and invitation (one per key); participants cannot invite; synthesis only by the facilitator's project; continuation of the awaiting facilitator by one instruction when all it awaits is resolved (deferred while its attempt is open); close/reopen; reconcile (start pending, retry failed up to 3, mark `unanswered`, tell untold facilitators) |
| Archive writer | `server/discussionArchive.ts` | One writer, own clone (`<area>/.local/discussions/archive`), file lock; reads the **published** manifest's horizon, appends beyond it, rewrites the manifest, commits with a `Discussion-Horizon` trailer, pushes; rebuilds on a rejected push; records status from the repository; retries failures with backoff; due on new records or a state change (final catch-up on close) |
| Source | `shared/intake.ts`, `shared/run.ts` | `ConversationSource.invitation`, kept through `sourceOf` |
| HTTP | `server/api.ts` | `api/discussions` (list/participants, create with a brief), `/<id>` (view + archive), `/messages`, `/invitations`, `/invitations/<n>/withdraw`, `/members`, `/close`, `/reopen`, `/archive` (GET status, POST publish) |
| CLI | `cli/discussion.ts`, `cli/wfe.ts` | `wfe discussion participants|list|show|create|post|contribute|invite|withdraw|join|leave|link|close|reopen|dispatch|archive`. `show` defaults to the attempt's own discussion and marks messages after the invitation's horizon. The executor's `wfe exec reconcile` now also runs the discussion reconcile and the archive |
| Contract | `docs/discussions.md`, `docs/conversations.md` | Members, invitations and states, routing, continuation, contributions, close/reopen, archive, HTTP, hand-off |
| Shared participant facts | agautolab `agent/guides/wfrun/guide.md` *A run that speaks in a discussion* | How a participant reads, speaks, cites; that it starts nobody; long work goes to its project's workflows |

Decision made while implementing: a dispatch that fails at once (unknown
participant) resolves the invitation immediately, and the first version told
the facilitator before it had finished inviting. The facilitator is now told
only when no attempt of its run is open; reconcile tells it right after its
attempt ends (the waiting case), and a contribution arriving later tells it
at once.

## Verification

`npm run check` (tsc, 464 tests, vite build): **all pass**. New
`test/discussions.test.ts` (8 tests):

| Case (report1) | Test |
| --- | --- |
| 1, 2 create, routing, same token = one message and the same invitations; addressed invites only those named; conversation route refuses a discussion | *a discussion starts with its members…* |
| 4, 5 attribution (participant, run, attempt, invitation), retried contribution = one message, concurrent contributions, one continuation after both, a resume job queued; reconcile tells nothing twice; synthesis is the facilitator's | *the facilitator invites two projects…* |
| 6, 7 unavailable participant `failed` (others not blocked), retries stop at 3; run ended without contribution → `unanswered`; late answer → told again | *an unavailable participant fails visibly…* |
| 8 direct question reaches only its participant; leave/join history; a member who left cannot be addressed; late joiner reads everything | *a direct question reaches only its participant…* |
| 9 close withdraws not-started, keeps started, continues nobody; invite refused while closed; reopen replays nothing | *closing withdraws…* |
| 3 restart between request and record finds the same run | *a restart between an invitation's request and its record…* |
| 11 archive: incremental; interrupted after push → next export adopts the commit without duplicates; failed push keeps live messages and reports; another writer's commit on main is kept (rebuild); two discussions; fresh clone has every message once, horizon = newest message; close published as final catch-up | *archive: …* |
| 10 HTTP + CLI: participants, create with brief (words verbatim, untrimmed), addressed post with run URL, cross-site write 403, conversation route 409, view with members/invitations/runs/archive, publish, close; `wfe discussion show` / `archive` from the CLI | *HTTP: …* |

The existing conversation tests were updated only where they rewind the store
to older versions (they now also drop the v4 additions); every
direct-conversation contract test passes unchanged.

## Not yet

Deployment (store migration on the live area) happens in Step 3, where the
real participant workflows need the service.
