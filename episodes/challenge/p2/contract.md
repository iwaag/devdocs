# Challenge p2 — the common conversation contract (step 1)

This is the contract the implementation follows. The canonical text after
deployment is agdevworld `workflow_editor/docs/conversations.md`; this file
records the decisions and the migration mapping made in step 1.

## Starting point (inspected 2026-10-23)

| What | State |
| --- | --- |
| WFE (studio, `service/current`) | `aadee43`, conversation store v11 |
| aghangout (`hangout/current`) | `6e311e4` |
| agautolab wfrun guide | `227ef1c` |
| Store | 42 direct conversations (41 open), 20 discussions (3 open), 315 messages |
| Open work in the store | no pending or started invitation; one failed invitation (`p16-s3-room` #15 → lowerpoly); one failed direct message (`p15-s6-unavail` → lowerpoly/converse, which does not exist) |
| Conductors | `20261020-233458-r93d5v` names `ponderer/conduct/run-007` |
| Archive | 20 discussions `current` in pj-ponderer `discussions/`; direct rooms have none |

Two engines existed: `ConversationStore.post/dispatch/reply/view`
(one destination, one run per message, receipt `conversation:<c>:<m>`) and
`DiscussionStore` (members, invitations, conductor, activities, archive). The
discussion engine is the richer one; the unified service is built from it.
Consumers: aghangout `room.ts` (chat) and `meeting.ts` (discussion), the Slack
bridge (conversations API only), the CLI (`wfe conversation`, `wfe
discussion`), `cli/run.ts` (`--source-message`, postQuestions), `cli/wfe.ts`
(reconcile, `wfe request`), `cli/evidence.ts`, `server/dialogue.ts`
(discussions only), `server/discussionWorkflows.ts`, the wfrun guide's two
sections, Front/forge/archsage `converse.md`/`discuss.md`, ponderer
`conduct.md`/`guide.md`/`synthesize.md`, and pj-ponderer `tools/discussions`.

## One conversation

A conversation (also "room") has a stable id, a title, `open`/`closed`, a
category (`conversation` | `check`), an optional display title, members with
an append-only membership history, messages, links, attachments, response
requests, an optional **facilitation** and an optional **default addressee**.
Nothing in its behaviour depends on how it began; the column `kind` survives
only as the room's origin (it names the archive namespace, below).

- **Members** are the person(s) and projects. A project member speaks through
  runs of its own workflows. ponderer may be a member like any project.
- **Default addressee** (`default_to`): a one-to-one room names its project.
  It is explicit: set when the room is created with one project, changed only
  by an explicit write.
- **Facilitation** is a record, not a membership: `{facilitator (ponderer/
  conduct), state active|ended, brief message, horizon, by, at, ended …}`.
  It starts only by an explicit action carrying a brief; the horizon is the
  newest message when it started. While active, the conductor is the default
  recipient. Ending it changes future routing only: what was asked of others
  stays the asking run's; nothing is stopped.
- **Mode** is a reading for clients: `direct` (one project, no
  facilitation), `group` (several projects, no facilitation), `facilitated`.

## Routing a person's message

Recipients are decided in this order; the first that applies wins.

1. `to` names participants → each is asked. Naming the facilitator while
   facilitation is active means the conductor.
2. `runs` names runs (explicit instruction to existing work) → the words go
   to each run, verbatim, with the message as their source and its
   attachments. A run that has ended fails visibly; nothing else is started.
3. `re` names open questions of runs still open → each such run gets the
   words as the answer to its question.
4. Facilitation active → the conductor: its open run gets the words (as the
   answer to its question that `re` names or its only open one, else an
   instruction); without an open conductor run a new conductor run starts.
5. A default addressee → it.
6. Otherwise the message is refused (409 `choose-recipient`, with the
   members): the client keeps the draft and asks whom to address.

For each project recipient, a question of that participant's open run named
by `re` is answered in that run instead of starting a new one (as before in
discussions). An unqualified follow-up is a new request; the receiving agent
decides what it means (it may `wfe run instruct|answer` other work).

## Response requests

The discussion invitation is the one delivery mechanism (table
`invitations`; "request" in the API). One row per recipient per message (or
per invitation by the person or conductor, or per awaited result):

| Field | |
| --- | --- |
| `participant`, `workspace`, `workflow`, `role` | whom, and the workflow selected for this request: `converse` (a project in a room with one project and no facilitation), `discuss` (otherwise), `conduct` (the facilitator), `deliver` (to an existing run), `await` (an awaited result). A project lacking the selected workflow uses its other one. Recorded once, kept for the request's lifetime. |
| `receipt` | the run request's receipt, recorded. Legacy direct turns keep `conversation:<c>:<m>`, legacy invitations `discussion:<c>:invitation:<n>`; new rows `conversation:<c>:request:<n>`. |
| `token` | the request's identity within the room (a resubmitted message finds the same rows) |
| `delivery` | `{question, message}` (answer to the asking run) or `{instruct: true, message}` (explicit instruction) |
| `awaited_by`, `held` | the conductor run that awaits it; `held` keeps a continuation that may not run now (below) |
| state | `pending → started → answered | unanswered | failed | withdrawn` (late answers kept) |

| Situation | Behaviour |
| --- | --- |
| New message to a project | one request per recipient, one bounded run by receipt |
| Reply to an open run question | delivered to the asking run (answer); no other run for that recipient |
| Message to an active conductor | its open run continues with the answer or instruction |
| Explicit instruction to existing work | `runs`: delivered to that run with the original words, source message and attachments |
| Agent contribution | speech with provenance (run, attempt, request); nobody is awakened |
| Late result | settles the awaited request; the continuation follows the recorded `awaited_by` run — instructed if open; if it completed, a new conductor turn while facilitation is active and the room open, else held |
| Unavailable recipient | `failed` with the reason, shown; reconcile retries up to 3 tries, then `retry` by hand; never redirected |
| Repeated submission or restart | same token → same message and requests; same receipt → same run |

**Questions** are one thing everywhere: `wfe run ask`. WFE posts every open
question of a run that speaks for an open room (its requests' runs and
conductors), in direct rooms too. `reply --kind question` is refused.

**Closing** withdraws requests not yet started (`pending`, `failed`), cancels
nothing, and continues nobody: a continuation that comes due while closed is
held (`held` = why) and shown among the room's obligations. **Reopening**
replays nothing; held continuations stay held until an explicit `resume`
(`wfe conversation resume <id> [<request>]`, `POST …/resume`), which runs
them once. Ending facilitation holds a continuation whose conductor run has
completed (a new conductor turn would restart facilitation); an open
conductor run is still told.

## Service, CLI, HTTP

One service (`ConversationService`, server/conversationService.ts) for
creation, reading, posting, membership, default addressee, facilitation,
requests, questions, activities, overviews, metadata, workflow references,
retry, close, reopen, resume. One CLI, `wfe conversation` (`wfe discussion`
is an alias of the same commands; `reply` is `contribute`). One HTTP tree,
`api/conversations[/<id>[/…]]`; `api/discussions` is gone. A room is
created by its first message with `with: <participant>` (one-to-one), or by
`POST api/conversations {id, title, members, defaultTo?, category?, text?, …}`.

## Configuration

`conversations.json`:

```json
{ "person": "iwaag",
  "projects": [{"workspace": "front", "converse": "converse", "discuss": "discuss"}, …,
               {"workspace": "ponderer", "converse": "converse"}],
  "facilitator": {"workspace": "ponderer", "workflow": "conduct"},
  "archive": {"workspace": "ponderer"},
  "dialogue": {…} }
```

A project whose `converse` is listed is offered for one-to-one rooms.

## Archives

WFE's `ConversationArchive` (the former discussion archive writer, same
single writer, lock and published-horizon recovery) exports every room to
pj-ponderer: legacy discussions stay at `discussions/<id>/` (paths and
commits unchanged), every other room at `conversations/<id>/`. The path is
recorded per room (`archives.path`) and never moves; promotion to a
facilitated room keeps it. Manifest format `ag.conversation.v1` (a superset
of `ag.argue.discussion.v1`, which readers still accept): adds `path`,
`defaultTo`, `facilitations`, `migration`; transcript adds `facilitation`
records and `request` fields on `invitation` records (`role`, `receipt`).
Export never starts a run. Dialogue lines are never archived.

## Character dialogue

The worker follows new agent speech in every room from the cutover (history
is not rendered wholesale); reads queue at most a window, as before.

## Migration mapping (store v12, no model call, no dispatch)

| Record | Mapping |
| --- | --- |
| direct conversation | `default_to` = its destination workspace; membership `join` rows for the person (its creator, and any other author) and the project, `by: wfe migration v12`, `at` = the conversation's creation, note naming the migration; category `conversation` |
| direct person message | one request: role `converse`, workflow = destination, `receipt conversation:<c>:<m>`, token `message:<m>:<ws>`, state from `dispatch_state` (`started` with its run, `failed` with error and tries, `pending`); `answered` when that run spoke a reply/result after it (`answered_by` its first); one event `migrated`; message `participant` = author, `addressed` = [ws] |
| direct agent message | `participant` = its run's workspace, `invitation` = the request whose run wrote it |
| `turn` links | kept |
| discussion | facilitation row `active` from creation (`by` = creator, brief = its brief message, horizon 0, note: migrated); category `discussion` → `conversation` |
| invitations | unchanged; `receipt` = `discussion:<c>:invitation:<n>`, `role` = `conduct` for the facilitator's, `await` for awaited results, `deliver` for delivered ones, else `discuss` |
| membership kind `facilitator` | kept as written; read as a project member |
| archives | `path` = `discussions/<id>` for existing rows |
| messages' dispatch columns | kept as history, no longer read |

Completed work is never pending after the migration (only `pending`
messages, of which there are none live, become pending requests).
Trial direct rooms are classified as `check` at deployment by explicit
metadata writes (recorded with author and reason), not by the migration.
