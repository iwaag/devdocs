# Challenge p2 — Step 2: common storage and response delivery

agdevworld `9c776c0` (not deployed yet; from `aadee43`).

## What changed in WFE

- **Store v12** (`server/conversationSchema.ts`):
  - `conversations.default_to`;
  - `invitations.role`, `receipt`, `held`;
  - `facilitations`;
  - `archives.path`.

  Category `discussion` is now `conversation`.
- **Migration**, a pure SQL step: no model call, no request, no dispatch.
  - Each direct conversation gets members and a default addressee (its
    destination). Each of its person messages gets one `converse` request.
    The request keeps the receipt `conversation:<c>:<m>`, its run, its
    tries and its error. Its state comes from the message's dispatch state:
    `answered` when that run spoke a reply or result after it. Each request
    gets one `migrated` event.
  - Each discussion gets an active facilitation from its creation.
  - Existing invitations get their receipts and roles.
- **One engine.** The former discussion store is now
  `ConversationService`. The direct engine in `conversations.ts` (`post`,
  `dispatch`, `dispatchDue`, `reply`, `view`) is deleted;
  `conversations.ts` is the store and the configuration.
- **Routing**, in order:
  1. `to` (participants);
  2. `runs` (an explicit instruction; an ended run fails and is never
     redirected);
  3. `re` naming an open run's question (the answer goes to that run);
  4. the active conductor;
  5. the default addressee;
  6. otherwise 409 `choose-recipient`, and nothing is recorded.
- **Workflow selection** per request is recorded:
  - `converse` for the only project when nobody leads;
  - `discuss` among several projects or under a conductor;
  - `conduct`, `deliver` and `await`.
- **Facilitation**:
  - starting it takes a brief and records the horizon, and keeps the room,
    its history and its work;
  - ending it changes routing only;
  - a conversation created with `facilitate` takes its first message to the
    conductor as the brief.
- **Held continuations.** While closed or no longer facilitated, outcomes
  that come due are recorded with `held` and shown among the obligations
  and in `status.continuations`. `resume` runs them once. Reopening replays
  nothing.
- **Retry** at the same recipient, by hand, after reconcile's 3 tries.
- **One CLI and one HTTP tree.**
  - `wfe conversation`; `wfe discussion` is the same command. `reply` is
    the same as `contribute`. New: `start`, `end`, `default`, `retry`,
    `resume`.
  - `api/conversations[/<id>/(messages|facilitation|default|members|requests|resume|close|reopen|meta|overview|workflows|dialogue|archive)]`.
    `api/discussions` is gone.
- **Configuration** `{person, projects[{workspace, converse, discuss}],
  facilitator, archive}`. The shape before p2 is read as this one.
- **Clients in the same commit:**
  - Slack posts `with` (the project) and reads `requests`;
  - Home's card lists conversations and links `?conv=`.

## Checks

- WFE suite: every test passes after the port. It covered 531 tests before
  this step; the direct-conversation tests were rewritten as the contract's
  scenarios in `test/conversations.test.ts` (13 tests), listed below.
- Tests ported to the new vocabulary: discussions, activities, reading,
  workflows, archive destination, dialogue, attachments and challenge API.
- Slack: 7/7.

| Scenario (plan step 6) | Test |
| --- | --- |
| One-to-one message/reply, no conductor | *one-to-one: the first message creates the room…* |
| Direct consultation with ponderer, no facilitation | *direct consultation with ponderer…* |
| Room gains a conductor and a specialist; ids, history and work kept; a pending question answered after it | *a one-to-one room gains a member and a conductor…* |
| Instructions reach a working run; an ended one fails, never redirected | *an explicit instruction reaches the run it names…* |
| A conductor finishes before the awaited result returns, continued once; ending facilitation; close and reopen; resume | *a conductor that finished before its awaited result returns…* |
| Unavailable recipient, retries, restart between request and record | *an unavailable recipient fails visibly…* |
| Migration of pending, failed and completed turns; receipts kept; nothing replayed | *a v11 store: direct conversations become rooms…* |
| Group without a default addressee | *a group without a default addressee asks whom…* |
| CLI from an attempt; questions are `wfe run ask` | *from an attempt: show finds its conversation…* |

## Rehearsal on a copy of the live store

`.local/challenge-p2/rehearsal/rehearse.ts`, run on a `.backup` of the live
store with a scratch bare copy of pj-ponderer:

- v11 → v12: 42 direct conversations became rooms with 71 `converse`
  requests: 63 answered, 7 started, 1 failed. No request is pending; the
  migration made no job.
- 20 discussions got their facilitation.

## Decisions taken while implementing

- Membership kind `facilitator` in old rows is read as a project member.
  Facilitation is its own record, not a membership.
- A request's receipt is recorded on the row (`receipt`) rather than
  derived. This keeps the legacy direct receipts exact.
- The writer clone stays at `.local/discussions/archive`, so no reclone is
  needed.
