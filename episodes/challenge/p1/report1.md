# Challenge p1 — Step 1: contract and storage

## What was inspected

- **WFE Git integration**: `server/git.ts` (argument-array Git, per-command
  config), `server/gitea.ts` (token as a scoped HTTP header through
  `GIT_CONFIG_*`, never in a URL or config; `createRepo`).
- **Archive writer** (`server/discussionArchive.ts`): one owner, its own
  disposable clone, a cross-process lock, "the repository is the truth"
  (publication reads back the pushed horizon), rebuild on destination change,
  retry on a rejected push. This is the pattern the challenge writer reuses.
- **Context snapshots and blocks** (`server/contextStore.ts`, `cli/context.ts`):
  revisioned writes with `--base` and conflict refusal; `--service` for
  writers on other hosts. The challenge CLI follows the same shape.
- **Conversation store** (`server/conversations.ts`, `server/discussions.ts`,
  `server/discussionSchema.ts`, store v10): messages, receipts, how a turn and
  an invitation become a run (`exec.request` → `createRunInCopy` →
  `inputOf`), and how deliveries to a working run carry a
  `ConversationSource`. This is where attachments go in step 3.
- **aghangout** (`app/src/{api,room,meeting,main,local}.ts`, fixture server,
  browser check): the room and meeting composers, outbox and drafts, panels.
  The shared components go there in step 4.

## Decisions

- **Repository**: Gitea `developer/pj-challenges` (a data repository; no
  agent, workflow project or service). Seeded with a README describing the
  layout and that writes go through WFE. Its URL is in the area's ignored
  `challenges.json`; tracked docs name no host.
- **Identity**: `ch-0001`, `ch-0002`, …, allocated in the journal's write
  transaction past every published and journaled id. Immutable.
- **Large files**: the Git service has LFS disabled. Evidence is ordinary
  Git content, capped at 32 MiB per file; anything larger belongs in a
  durable store linked from the body (documented).
- **Backup**: the store can push every publication to configured
  `mirrors` (best effort, state in `status`). The mirror on another LAN host
  is set up and verified at deployment (step 6).

## Delivered (agdevworld `82bd397`)

- `server/challenges.ts` — `ChallengeStore`:
  - **journal** (`<area>/.local/challenges/challenges.sqlite`): every create
    or edit is accepted with its request identity, actor, on-behalf-of, run,
    attempt, source message, full fields, evidence bytes (blobs until
    published) and base revision; states `pending → published | unsynced |
    conflict | refused | discarded`;
  - **publication** under one cross-process lock in a disposable writer
    clone: fetch, look up each request in the remote history first (crash
    after push / unseen push outcome → recorded, never rewritten), check the
    base against the record's latest revision there, commit authored by the
    actor with `Challenge-*` trailers, one push, never forced; a rejected
    push starts again from the remote; failure → `unsynced` with the
    content kept;
  - **reads**: an index rebuilt from the repository whenever its head moves
    (and `reindex` from a fresh clone), current reads, pinned reads by
    commit, evidence bytes, history from the log with each revision's
    title and state;
  - **refusals at acceptance**: a stale base (409 `conflict` with the current
    record, nothing journaled), a second open edit of one record
    (`pending-edit`), an unchanged edit, malformed names, oversize files.
- `docs/challenges.md` — the contract: record format, identities,
  attribution trailers, revisions (current vs pinned), publication states,
  evidence paths, recovery, carrying challenges into work, HTTP, config.
- `test/challenges.test.ts` — 8 checks, all passing:
  format round trip; create/read/edit/pin/history with a pinned read still
  returning the original after edits; resolve/reopen as versioned edits and
  resolved records found by text, project and state; concurrent edits (stale
  base refused; remote change after acceptance → conflict keeping the draft;
  discard); a failed push stays unsynced across a restart and publishes once
  on retry; a crash after a successful push is recognised from the remote
  history with no second commit; the index rebuilt from a fresh clone keeps
  records, evidence and history; mirrors catch up after a failure.

Retired or unregistered projects do not affect reads: `projects` are names,
never resolved against the registry.

## Next

Step 2: HTTP routes and `wfe challenge`.
