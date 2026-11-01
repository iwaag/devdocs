# p19 step 1 — trace and contract

Plan: [plan.md](plan.md), step 1.

## What the code does now (agdevworld `799c039`)

- **Submission.** `POST api/conversations/<id>/messages` (`server/api.ts`) reads
  JSON through a 2 MiB body reader and calls `ConversationService.post`. `post`
  refuses empty text, creates a new room from `with` (title: `firstLine(text)`),
  computes recipients with async run reads, pins challenges, then writes the
  message, its `attachments` rows and its requests in one SQLite transaction,
  and dispatches. A repeated token finds the message by `UNIQUE (conv, token)`
  and returns it **without comparing the content**.
- **Attachments are challenge-only everywhere.** `AttachmentRef` (`shared/intake.ts`)
  is `{kind: 'challenge', id, commit, title, sha256}`; the table `attachments`
  (store v11) has `commit_id`/`title` columns and `UNIQUE (message, kind, ref)`;
  `attachmentsOf` deduplicates by challenge id; `shared/run.ts attachmentsOf`
  rejects any other kind in `source.attachments`; `server/runs.ts withAttachments`
  writes each challenge's text into the run folder; `server/exec.ts`
  `attachmentBrief`/`attachedLines` name challenges only.
- **Delivery paths** (all carry `source.attachments` or `input.attachments`):
  `startRun` (new runs of every role, including a new standing conductor),
  `deliver` (answer to a question → `question.answer`; otherwise
  `instruction.receive`, which also wakes a resting conductor), and requests
  whose `re` names the message (`invite`). The reducers require non-empty text
  for `question.answer` and `instruction.receive`, and `inputOf` refuses an
  empty braindump — a file-only message would fail every path today.
- **Bounded reading and safe serving exist for challenges** (`server/challengeApi.ts`):
  `MAX_REQUEST = ceil(64 MiB·4/3) + 1 MiB`, a declared-length check, `nosniff`,
  `content-security-policy: sandbox`, active types served as text.
  `Buffer.from(base64)` there is lenient (malformed input decodes silently).
- **Shares** (`server/shares.ts`): `shareRoot` (marker-checked), `resolveRef`,
  `hashFile`, `copyVerified`, `sweepUnfinished` (dead-process staging only),
  and `publishOutput` (staging → verify → manifest → rename). Publication is
  tied to a run's records; a conversation upload needs its own owner.
- **Workers** (`cli/remote.ts`): on an execution node `wfe challenge` talks to
  the coordinator's HTTP routes (files written on the node); other commands are
  forwarded to the coordinator's copy. A conversation file fetch must be the
  former kind.
- **Archive** (`server/conversationArchive.ts`): transcript records embed
  `attachments` per message; no payloads.
- **Harness.** Runs use Claude Code (`agautolab/agents.toml` profile `sonnet`,
  `Read` allowed, permissions skipped): an agent can `Read` a local PNG/JPEG.
  The shared material reference is `agautolab/agent/guides/wfrun/reference/material.md`.
- **Locks** (`server/lock.ts`) are SQLite `BEGIN IMMEDIATE` file locks that the
  OS releases when the holder dies: recoverable ownership on one host, which is
  where the coordinator writes.

Baseline revisions and deployment facts (service, worker pins, agshare
placement, no existing share-payload backup) are in the ignored
`pj-agdev/.local/p19/notes.md`.

## The published contract

agdevworld `workflow_editor/docs/message-files.md` (linked from
`docs/conversations.md`) and the fixture `docs/fixtures/message-files.json`
(real bytes: a 4×4 PNG and a text file with their hashes, the limits, a
file-only post and its response, a retry, every error shape).

Differences from the plan's table, made because of the existing structure:

| Plan | Contract | Why |
| --- | --- | --- |
| `conversation post --file` | `--attach <path>` (repeatable) | `post --file <f>` already reads the message text from a file |
| errors | `invalid-attachment`, `attachment-limit`, `token-conflict`, `storage-unavailable`, `attachment-not-found`, `attachment-missing` (410), `attachment-integrity` | absent record and absent bytes are separate facts |
| limits | `attachmentFiles` in the list and the room view (also `available`/`problem`) | a client needs to know whether it may offer files now |
| — | `preview: image|text|null` per file, decided by WFE (image signatures checked at acceptance) | a MIME hint is not authority to render |
| — | `GET …/attachments/verify`, `wfe conversation attachments --verify`, `uploads`, `export`, `restore`, `backup` | explicit integrity, diagnosis and recovery commands |

Storage: `attachments.json {share, prefix}` (initially `agshare`,
`conversations`), bundles `shared://agshare/conversations/<c>/attachments/<b-…>/`
with `manifest.json` (`ag.message-files.v1`, canonical digest) and `<f-…>` files;
association rows in store v14 `message_files`; upload operations in `uploads`
(`reserved → published → accepted`, `failed` retried from `reserved`).

## Checks

None ran in this step (documentation and fixture only). The fixture's hashes
were computed from the bytes it embeds.

## Revisions

- agdevworld: contract and fixture commit (see the commit after this report).
- No deployment, no store change, no agent request in this step.
