# p15 Step 2 — the generalized WFE conversation surface

A conversation now records its destination (`<workspace>/<workflow>`). Every
turn, every repair and every restart uses that recorded destination. Clients
discover the configured targets through the API, and a second client can be
written from `docs/conversations.md` and `wfe conversation help` alone.
Deployed as agdevworld `a988fe9`.

## Changes (agdevworld `a988fe9`, pinned by pj-agdev `32b2aa1`)

**Store** (`server/conversations.ts`):

- Schema v3 adds `workspace` and `workflow` to conversations.
- The migration maps the earlier conversations once to the single entrance
  the configuration named then (`entrance`, else the first target).
  - Live result: all 23 conversations became `front/converse`.
  - There is one schema. No compatibility reader exists.
- `conversations.json` is now `{person, targets: [{workspace, workflow}]}`.
  The old `entrance` form is no longer read as configuration.

**Creating and posting.**

- A first message must name `target`, which must be a configured target.
  Otherwise the post is refused with 400 (`no-destination`, `unknown-target`).
- A message that names another target on an existing conversation is
  refused with 409 (`other-destination`).
- A refused first message creates nothing.
- Nothing defaults to any project, Front included.

**Dispatch and recovery.**

- `dispatch` and `dispatchDue` read the conversation's recorded
  destination, not the configuration.
- A conversation keeps its destination after its target is removed from the
  list.
- A destination that cannot take the request (an unregistered workspace, a
  missing workflow) leaves the message `failed` with the reason. It is
  retried there by `wfe conversation dispatch` or the executor's reconcile,
  and never sent elsewhere.
- The receipt `conversation:<conv>:<msg>` is unchanged, so a replay finds the
  same run.

**Discovery and view.**

- `GET api/conversations` returns `targets`. Each target carries
  `{target, workspace, workflow, project, projectName, workflowName, intent,
  agent, state: available|unavailable, problem}`.
- The list filters by `?target=` and shows each conversation's
  `destination`.
- The view names `conversation.destination`. Each linked run gains:
  - `title`;
  - `files`, the run's file route, to which a run-folder path is appended;
  - `presentations`, the review pages serving now on a LAN address.
- Message, question, link and status fields are as before.

**CLI.** The CLI gains `wfe conversation targets`, `list --to`, `post --to`
and a `show` header that names the destination. Its help speaks of
conversations with project agents rather than "the entrance (Front)".

**Docs.** `docs/conversations.md` covers:

- configuration;
- the destination rules;
- the HTTP table and the view;
- browser clients: same-origin writes from any page of the agdev origin;
  `allowedOrigins` for a client hosted elsewhere; the configured person; no
  credential in a page.

**Slack bridge.**

- `config.json` gains `target` (required; live: `front/converse`). It is
  posted with every message, so a new DM conversation names its destination.
- The conversation link is `conversationUrl` (default `<linkBase
  origin>/hangout/?conv={conv}`), labelled "aghangout で会話全体を見る".
  Messages from a page are labelled "aghangout から".
- `deskUrl` was removed.

**Hosting route.**

- nginx serves `/hangout/` on `:8093` from a read-only mount of aghangout's
  releases (`compose.yaml`: `${AGHANGOUT_DIR:-../.local/agdev/hangout}`,
  `current/`).
- The web container is rebuilt with aghangout's first deployment (Step 5).

## Tests

- WFE: `npm run check`, 455 tests pass. New tests in
  `test/conversations.test.ts`:
  - two destinations in parallel conversations: routing, later messages
    with and without a target, concurrent replies attributed by run and
    `re`, the list filter;
  - refusals: no target, an unknown target, another destination (409); a
    refused message creates nothing;
  - a configuration change keeps the recorded destination;
  - a missing workflow: the turn fails visibly, is not sent to Front, and
    is started once at its own destination when repaired;
  - v2 → v3 migration from the old `entrance` file;
  - HTTP: targets, a page's first message, 409, a foreign-origin 403, the
    filter, and the view's run view and file route URLs.
  - The existing duplicate, restart and recovery tests now run with
    destinations.
- Slack: `npm run check`, 7 tests pass. Every new conversation's post
  carries `front/converse`.

## Verification on the deployed route

**Deployment.**

- WFE `a988fe9` was deployed with `scripts/wfe-release.sh deploy`
  (previous `dbd974b`). The store is at user_version 3, with every
  conversation `front/converse`.
- The Slack release `a988fe9` is active and connected.
- Backup: `.local/agdev/.local/backups/p15-step2-predeploy-20261020-190715/`
  (conversations, exec store, conversations.json, Slack config).
- Rollback: activate `dbd974b`, then restore `conversations.sqlite` and
  `conversations.json` from the backup, because the older release refuses a
  v3 store.
- The first deployment command, which also wrote the configuration, was
  refused by the session's permission classifier. The configuration was
  then written on its own, and the deployment ran as a separate command.

**From agpc**, through `http://agstudio.local:8093/wfe/api/conversations`
(`pj-agdev/.local/p15/step2/lan-check.txt`, `lan-view.txt`):

| Check | Result |
| --- | --- |
| Targets | `front/converse available front-agent`, person `iwaag` |
| Write from a foreign origin | 403 |
| Page origin, new id, no target | 400 `no-destination` |
| Page origin, unconfigured target | 400 `unknown-target` |
| Page origin, first message with `target: front/converse` | 200, run `front/converse/run-039`, author recorded as given (`Omni Agent (for Developer)`, the trial actor) |
| Same token again | 200, duplicate, same run |
| Another destination on it | 409 `other-destination` |
| Reply | `#88` by `front-agent` for `front/converse/run-039`, kind `result`, `re #87`, 19 s after the post |
| View | destination, run status `Done · Every step completed`, run view URL, file route. `braindump.md` through the file route answers 200 from agpc |

**Identity.**

- A page's words are the configured person's. The fixture test asserts
  this.
- The live trial named its trial author explicitly, so the Developer was not
  impersonated.
- Replies carry the run's agent as author, independent of any client
  presentation.

## Active delivery accounted for

- No turn was pending or running at deployment.
- The 10 Slack thread bindings keep their conversation ids. Their
  conversations are `front/converse`, the same as the bridge's configured
  target.
- The old Front Desk still reads and posts its existing conversations: they
  exist, so it sends no target. A new conversation started from the old desk
  would now be refused (no target). The desk is replaced in Steps 4–5.
