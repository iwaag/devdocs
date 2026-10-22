# Challenge p2 — Step 3: archives for every conversation

In agdevworld `9c776c0` with step 2 (`server/conversationArchive.ts`). The
contract is `docs/conversations.md`, *Archive*.

## What changed

- **Every conversation is due**, whatever its kind: one-to-one rooms, closed
  ones, and rooms with a retired project such as lowerpoly.
- **The archive path** is recorded per conversation (`archives.path`) at its
  first publication and never moves:
  - conversations that began as discussions keep `discussions/<id>/`, so
    their commits and citations are unchanged;
  - every other conversation goes to `conversations/<id>/`;
  - promotion to a facilitated room keeps the path.
- **Manifest** `ag.conversation.v1`, a superset of `ag.argue.discussion.v1`.
  New fields: `path`, `origin`, `defaultTo`, `facilitations[]`, and
  `migration` (what store v12 derived). Requests carry `role`, `receipt`,
  `instruct` and `held`. The commit trailer is `Conversation-Horizon`.
- **Transcript.** A request's first event carries what the request is. A
  migrated one-to-one turn has only its `migrated` event, so it is still
  readable. Facilitation start and end are activity records. Messages keep
  their attachments, pinned revisions included.
- **Unchanged:**
  - the single writer under a file lock;
  - appending from the published horizon (the repository is the truth);
  - rebuilding on a rejected push;
  - backoff on failure;
  - destination verification;
  - export starts nothing.
- **pj-ponderer `tools/discussions`** reads both namespaces and both formats,
  and cites `ponderer@<commit>:<path>/transcript.jsonl#<msg>`.

## Checks

| Plan item | Evidence |
| --- | --- |
| Interruption after the push | `discussions.test.ts` *archive…*: a crash after the push publishes nothing twice |
| Concurrent export attempts | `conversations.test.ts` *archives…*: two writers at once; one publishes, the other finds it published; no failure |
| A second backfill adds nothing | the same test (`unchanged`), and the rehearsal below |
| Publication failure keeps the room | the same test: `failed` with the reason; messages untouched; the next publish succeeds |
| Fresh-clone restore with authorship, history and references | the same test reads a fresh clone: every message once with its author, the facilitation event, requests with role and receipt, the manifest's path, default addressee and facilitation |
| Existing discussion archives preserved | `archiveDestination.test.ts`: pre-p2 discussions stay at `discussions/<id>` |

**Rehearsal on real data.** The live store's backup, migrated to v12, was
published into a scratch bare copy of pj-ponderer:

- 43 conversations were published and 19 were unchanged; there were no
  failures, and it took 4.0 s;
- a second backfill published 0;
- 42 conversations went to `conversations/`, 20 to `discussions/`;
- in a fresh clone, the updated `tools/discussions validate` passed for all
  62. One example: `20261022-161753-chp1fg` reads with its migrated
  membership, its turn link and request 164 answered by #313.
- No request became pending.

## Status and source links in the room

The view's `archive` (state, commit, url, `path`) and the list's `archive`
are what aghangout shows (step 5). Old `?discussion=<id>` links resolve to
the room by id (step 5).

## Recovery facts (documented in docs/conversations.md, *Restoring*)

- A fresh clone of pj-ponderer restores readable transcripts: verbatim
  messages, authors and ids, membership and facilitation history, request
  outcomes, links, overviews and workflow versions.
- Pinned challenge references resolve only with `pj-challenges` (and its
  agpc mirror) beside it.
- Restoring the archive restores records only. It never replays delivery or
  executes historical work.
