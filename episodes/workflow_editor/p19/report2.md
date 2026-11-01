# p19 step 2 — durable storage and submission

Plan: [plan.md](plan.md), step 2. Contract: agdevworld
`workflow_editor/docs/message-files.md`.

## What changed (agdevworld `3fade95`, docs follow-up in the next commit)

- **Destination.** `<area>/attachments.json {share, prefix, backup?}` (ignored
  operating configuration; on agstudio `agshare` / `conversations`). Hosts
  resolve the share through their own `shares.json`; records hold only
  `shared://agshare/conversations/<c>/attachments/<b-…>/<f-…>`.
- **Store v14** (`server/conversationSchema.ts`, idempotent `IF NOT EXISTS`):
  `uploads` (one operation per conversation and token: author, submission
  digest, bundle id, share, prefix, allocated files, challenge pins, state,
  manifest digest, accepted message, error, tries; `UNIQUE (conv, token)`,
  `bundle UNIQUE`), `upload_events`, `message_files` (one row per occurrence:
  conversation, message, ordinal, attachment id `UNIQUE`, bundle, name, hint,
  bytes, sha256, preview, served type, ref; `UNIQUE (message, ordinal)`) and
  `file_backups`. Nothing existing is rewritten.
- **`server/messageFiles.ts`**: limits (20 files, 32 MiB, 64 MiB,
  request 90527062 bytes), strict base64 and name validation with every size
  checked before decoding, the submission digest (`canonicalJson` of author,
  text, ordered `{name, mime, bytes, sha256}`, challenges, `to`, `runs`, `re`,
  `with`, `title`, `via`), WFE-allocated ids (`b-<UTC>-<hex8>`,
  `f-<hex24>`), `publishBundle` (lock per bundle that the OS releases when the
  holder dies; dead-process staging swept; files written `wx`, re-read and
  hashed; manifest `ag.message-files.v1` with canonical digest; rename into
  place; an existing complete bundle with the same digest is re-hashed and
  adopted; another manifest or a manifest-less folder is never touched).
- **`ConversationService.post`** was split into `room` (creation, title
  fallback: given title → first text line → first file name), `route`
  (unchanged routing rules, plus a routing-state fingerprint), `accept` (the
  one transaction) and `afterAccept` (brief, standing sync, dispatch).
  Text-only messages take the same path as before; a token that a file
  submission used is refused for text (`token-conflict`).
  With files (`postFiles`):
  1. same token + same digest + same author and `accepted` → the accepted
     message and its requests (`duplicate: true`), no routing or publishing;
     otherwise a different digest/author, or a text message with that token,
     → 409 `token-conflict`;
  2. destination configured and share reachable, else 503
     `storage-unavailable` (nothing reserved);
  3. a routing pre-check (a message nobody can receive reserves nothing);
  4. reservation in a transaction (re-read under the uniqueness constraint),
     challenges pinned once and stored with the operation;
  5. `publishBundle` outside any transaction; `published` (or `failed` with
     the reason; the retry of the same payload starts again from the reserved
     identities);
  6. routing computed again, then one transaction re-reads the operation,
     compares the routing fingerprint (moved → computed again, at most three
     times, then 409 `routing-changed`), and writes the message,
     `message_files`, challenge rows, requests and `accepted`;
  7. dispatch through the existing receipts.
- **HTTP** (`server/api.ts`): the conversation POST routes read up to the
  request limit (declared `content-length` refused early; `attachment-limit`
  code), `files` on `messages` and on room creation, empty `text` allowed with
  files; the post response's `message.attachments`; `attachmentFiles` in the
  list and room view.

## Checks that ran

- `node --test test/messageFiles.test.ts`: 10/10. Covered in this step:
  - file-only first message (`text: ''`, room title `clipboard image.png`,
    ordered rows, bytes on the share equal to the originals, manifest digest
    recomputes, operation records the manifest digest);
  - mixed text/files and adjacent messages with the same name (`plan.txt`,
    and twice in one message): distinct ids, separate bundles, each run gets
    its own message's occurrences;
  - same retry (same message and request ids, no new rows), changed bytes,
    changed text, changed author, a retry without files, and a text token
    reused with files → `token-conflict`; three concurrent identical posts →
    one message, one request, one file row, one operation;
  - injected crashes: in staging (operation `failed`, no message, no staging
    folder left, the retry keeps the reserved bundle id), after publication
    (operation `published`, no message; the retry adopts the bundle), after
    acceptance before dispatch (request `pending`; `reconcile` dispatches it
    once; the client's retry returns the same message and request, one run);
  - the addressee leaves while the files publish → `left`, nothing written,
    bundle kept; after re-joining the same token succeeds; the default
    addressee changes between routing and writing → routed again from the new
    state (the hook ran twice); `attachments.json` changed mid-publication →
    the reserved destination is kept; a message to nobody reserves nothing;
  - limits: 20 files ok / 21 refused, exactly 32 MiB ok / one byte over
    refused, total over 64 MiB refused, malformed base64 (`abc`, `ab!=`,
    `data:` prefix, inner padding, line breaks), bad names (empty, `../x`,
    `a/b`, `a\b`, NUL, 256 characters), a bad hint, a non-list;
  - share marker removed → `storage-unavailable`, no operation, and a
    text-only message still started its run.
- `npm run check` (tsc, 651 tests, both vite builds): the first run had 2
  failures — store tests that reset `user_version` and reopen found v14's
  tables already present. The migration was made idempotent; both files then
  passed (23/23). The full check is repeated after the remaining steps
  (report6).

## Not done here, or deliberately left

- Memory at the limit and the proxy's limit against base64 expansion are
  measured against the deployed service through nginx (report6/report7),
  not in this step.
- Bounded streaming was not implemented: the JSON body is buffered (≤ 90.5
  MB). It is replaced only if the measurement shows a need.
- An unaccepted published bundle is kept (no garbage collection), as the plan
  requires. Cleanup removes only `.staging-*` of dead processes on this host.
