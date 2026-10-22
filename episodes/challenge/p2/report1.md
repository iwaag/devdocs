# Challenge p2 — Step 1: inventory and the common contract

## Inventory (read-only)

- **Revisions**: WFE `aadee43` serving (store v11), aghangout `6e311e4`,
  wfrun guide agautolab `227ef1c` — as p1 left them.
- **Store**: 42 direct conversations, 20 discussions, 315 messages. No
  invitation pending or started; one failed invitation (lowerpoly in
  `p16-s3-room`) and one failed direct message (`p15-s6-unavail` to a
  lowerpoly/converse that does not exist). One recorded conductor
  (`ponderer/conduct/run-007`, check discussion). 20 discussion archives
  current; direct rooms have none.
- **Two engines** in WFE: `ConversationStore` (post → one run per message,
  receipt `conversation:<c>:<m>`, `reply`, `link`, its own view) and
  `DiscussionStore` (members, invitations, delivery to the asking run,
  conductor, activities, overviews, metadata, archive, workflow references,
  dialogue). They already share the table of conversations, messages, links
  and attachments; `notDiscussion()` / `need()` kept them apart.
- **Consumers**:
  - aghangout `room.ts` (445 lines) and `meeting.ts` (1092): duplicated
    polling, merge, composer, outbox, tray, playback, tabs; real differences
    are recipients/creation, playback of all speakers, status, panels,
    dialogue and workflow cards (discussion only);
  - the Slack bridge uses only `api/conversations` (`target`, `via`, `dispatch`,
    `run`; never `re`, `asking`, `destination`);
  - CLI `wfe conversation` (show, reply, link, post, dispatch) and `wfe
    discussion` (26 subcommands); `cli/run.ts` (`--source-message`, posting
    questions, completion warnings), `cli/wfe.ts` (reconcile, `wfe
    request`), `cli/evidence.ts`, `cli/dialogue.ts`;
  - guides: wfrun guide *A run that serves a conversation message* and *A run
    that speaks in a discussion*; pj-front `converse.md`, `discuss.md`,
    `guide.md`; forge and archsage `converse.md`/`discuss.md`; pj-ponderer
    `conduct.md`, `guide.md`, `synthesize.md`, `tools/discussions`
    (requires format `ag.argue.discussion.v1`).
- Every converse/discuss/conduct workflow runs on studio's `conversation`
  lane with one `talk` node (conduct: one `ponder` node); converse/discuss
  opt out of Zulip intake.
- In direct rooms a question was asked twice (`wfe run ask`, then `reply
  --kind question`) and answered by a *new* turn recording `wfe run
  answer`; in discussions WFE posts the run's question and delivers the
  reply to the asking run. The unified contract takes the latter.

## Decisions

Written in [contract.md](contract.md). In short:

- The discussion engine becomes the one service; a direct chat is a room with
  one project member and a default addressee. `kind` only names the archive
  namespace.
- Facilitation is an explicit record (brief, horizon, by, start/end), not a
  membership; ponderer can be consulted as a project without it.
- Recipient order: `to` → `runs` (explicit instruction) → questions named by
  `re` → active conductor → default addressee → refused with a recipient
  choice.
- Workflow per request is selected and recorded (`role`: converse, discuss,
  conduct, deliver, await); receipts are recorded per request, legacy ones
  kept exactly.
- Closing holds due continuations instead of dropping or running them;
  reopening replays nothing; `resume` runs held ones once. Ending
  facilitation never stops work.
- One CLI (`wfe conversation`, with `wfe discussion` as an alias), one HTTP
  tree (`api/conversations`), one archive writer with per-room paths
  (`discussions/<id>` kept, `conversations/<id>` new), manifest format
  `ag.conversation.v1`.
- ponderer gets a `converse` workflow for direct consultation.
- Trial direct rooms are classified `check` by explicit metadata writes at
  deployment.
