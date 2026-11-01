# p19 step 5 — archive, export and payload recovery

Plan: [plan.md](plan.md), step 5. Code in agdevworld `3fade95` (deployed in
`47a83cc`).

## What changed

- **Archive** (`server/conversationArchive.ts`). The record construction was
  extracted into `records(conv, from)` (used by the writer and by export).
  Transcript `message` records carry both kinds of attachments; each accepted
  bundle's `manifest.json` is written beside the transcript
  (`<path>/attachments/<bundle>/manifest.json`, byte for byte the share's),
  and the archive manifest lists `files` (bundle, share, path, digest, the
  manifest kept, messages). Bytes stay on the share. Nothing deletes
  attachments on close, archive or retirement.
- **Export / restore** (`server/messageFiles.ts`, CLI `wfe conversation
  export|restore`). A package is a folder: `transcript.jsonl` (the whole
  history), every bundle manifest, every original file, and `package.json`
  (`ag.conversation-package.v1`: every hash, the message ↔ file associations,
  pinned challenges as references with the note that their content needs
  `pj-challenges`). Export reads each file, checks size and sha256 and checks
  the written copy; a missing or altered file stops it with the reason.
  Restore writes only into a new folder, checks every hash and each manifest
  digest, cross-checks transcript associations, and reports; it opens no
  store and starts nothing.
- **Payload backup** (`server/fileBackups.ts`, `wfe conversation backup`).
  There was no existing backup of share payloads (report1). Each accepted
  bundle is copied with `rsync --ignore-existing` to the destination in
  `attachments.json` `backup` and every copied file's sha256 is read back
  there over ssh; the state is recorded per bundle (store v14
  `file_backups`: verified/failed, destination, tries, time) and retried with
  back-off. The executor's reconcile runs it. `backup --restore <bundle>`
  brings one bundle back and verifies it.
  - On agstudio the destination is agpc's local disk
    (`~/backups/agshare-message-files`), not the share agpc mounts: a
    different host and disk from agstudio's `/opt/nfs/agshare`. Host, path
    and key are in the ignored `attachments.json`; the failure domain is in
    `pj-agdev/.local/p19/notes.md`.
  - Scope: message-file bundles only (`conversations/<id>/attachments/`), not
    the rest of agshare.
- **ponderer's readers**: `tools/discussions` ignores `attachments` (it already
  ignored challenges), so `validate` keeps passing; but a file-only message
  reads as empty in `show`/`cite`. Commissioned to ponderer's own agent
  (ponderer/freework/run-001, request in `pj-agdev/.local/p19/step5/`);
  its result is assessed in report6.

## Checks that ran

- Unit (`test/messageFiles.test.ts`): archive transcript and manifests in a
  clone of the archive origin (each manifest digest recomputes); export of an
  intact conversation, restore with every hash verified and associations
  consistent, nothing dispatched; a tampered package reported; export of a
  conversation with altered bytes refused (`attachment-integrity`).
- Live (deployed `47a83cc`):
  - `p19-check` archived by the executor's reconcile. A fresh clone of
    `developer/pj-ponderer` has `conversations/p19-check/attachments/<b>/manifest.json`
    for both bundles, identical (`cmp`) to the share's; transcript #542 and
    #544 carry their file with id and position; manifest `files` names
    messages 542 and 544.
  - `wfe conversation export p19-check` → 2 manifests, 2 files; `restore` into
    a new folder → 5 files hash-verified; restored image sha256 `91a21064…`,
    text file `a6103aa9…` (the originals).
  - Backup: the reconcile had already backed up `p19-check`'s two bundles
    (13:53:12, 13:53:13 UTC); a manual `backup` added the led room's bundle:
    3/3 verified at the destination. `backup --restore
    b-20261101t135159z-61fbc59d` from agpc into a scratch folder verified the
    manifest digest and the file (`91a21064…`).

## Recovery boundary (portable procedure)

1. Records: a clone of the archive repository restores transcripts, message ↔
   file associations and bundle manifests; pinned challenges need
   `pj-challenges` beside it.
2. Bytes: from the payload backup (`wfe conversation backup --restore`) or a
   recovery package (`restore`); every file is checked against its manifest
   hash. A clone alone restores no bytes.
3. Neither path starts runs or replays delivery.
