# p19 step 3 — reads, run delivery and image access

Plan: [plan.md](plan.md), step 3. Code in agdevworld `3fade95`/`47a83cc`
(deployed), agautolab `a9b82f8`.

## What changed

- **One attachment union.** `AttachmentRef = ChallengeAttachmentRef |
  FileAttachmentRef` (`shared/intake.ts`). Challenge hashes, revision pins and
  their run-folder files are unchanged. `shared/run.ts attachmentsOf` validates
  `kind: file` (ids, bundle, conversation, message, ordinal, size, sha256,
  `shared://` ref) in `source.attachments`; `InputAttachment` is a challenge
  with its file or a file reference (no bytes in the run's Git history).
- **Occurrences, not names.** `ConversationStore.attachmentsOf` returns each
  challenge once and every file occurrence by attachment id, in message
  order; `withAttachments` de-duplicates files by id only.
- **Empty words with files.** `inputOf` accepts an empty braindump/request when
  file attachments come with it; the `question.answer` and
  `instruction.receive` reducers accept empty text when the source carries a
  file. Existing records replay unchanged (all 20 workspaces `usable`, report6).
- **Every delivery path** already passed `source.attachments` /
  `input.attachments`; they now include files: new runs of any role,
  answers to an open question, explicit instructions, the standing
  conductor's instruction (which wakes it), and later requests whose `re`
  cites the message.
- **Briefs** (`server/exec.ts`): the initial brief lists *Files the person's
  words carried* — name, type, size, sha256 prefix, source message and
  position, and the exact `wfe conversation fetch …` line; continuation
  briefs show the same per answer/instruction.
- **Reads and retrieval.** View/history/post response carry file entries with
  `preview`, `api` and `url`. `GET …/messages/<n>/attachments[/<id>]` validates
  the whole tuple; `GET` re-hashes before sending (409 `attachment-integrity`),
  `HEAD` checks size; 410 `attachment-missing`; 503 when the share is not
  reachable. Served type is decided at acceptance from the bytes (PNG, JPEG,
  GIF, WebP signatures → inline image; `.txt/.md/.csv/.log/.json` → text/plain;
  everything else, SVG and HTML included, `application/octet-stream` as a
  download), always with `nosniff` and `content-security-policy: sandbox`.
  `GET …/attachments/verify` is the explicit integrity read.
- **CLI** (`cli/messageFiles.ts`): `attachments [--message] [--check-bytes]`,
  `fetch … [--out]` (share where mounted, else the coordinator; hash checked
  before rename into place; default `.local/attachments/<id>/m<n>/…`),
  `uploads`, `export`, `restore`, `backup`; `post --attach`; `show` prints a
  message's files. On an execution node `cli/remote.ts` runs
  `conversation attachments|fetch` locally against the coordinator's routes
  (like `wfe challenge`), so `--out` lands on the node.
- **Shared reference** (agautolab `a9b82f8`,
  `agent/guides/wfrun/reference/material.md`): one fact — files are named in the
  brief with their source, the bytes are on a share, `wfe conversation help`
  says how to fetch, a fetched image opens like any image, a name is not its
  content. No inspection sequence is prescribed.

## Checks that ran

- Unit/integration (`test/messageFiles.test.ts`, part of 651/651): served
  types and headers, `?download=1`, HEAD, the whole-identity 404, altered
  (same size) → 409, missing → 410, `verify` states; answer, instruction,
  later request and conductor wake each carrying the file with its source
  message; a view starts nothing; CLI post/list/fetch, and fetch from a
  process with no registry and no share mount through the coordinator.
- **Deployed harness, real agents** (WFE `47a83cc`, Claude Code runs):
  - The image (`check-image.png`, 800×500, features chosen at random and kept
    only in the ignored notes; neither its pasted name
    `clipboard-20261101-225158.png` nor the message describes it) was pasted
    in the served aghangout into `p19-check` #542, to front and forge.
  - front/discuss/run-014 (studio) fetched it and answered #543 in 16 s:
    light green background, purple circle, black "1315" — all four recorded
    features correct. forge/discuss/run-014 answered #545 the same.
  - ponderer/conduct/run-067, the led room's conductor, answered #548 for
    #547 with the same, correct description.
  - forge/freework/run-005 on **agpc** (worker gpupc) fetched #542 there
    ("read from the share on this host"), opened it with `Read` and described
    it correctly.
  - A text file (#544, `handover-notes.txt`): forge/discuss/run-015 fetched it
    and quoted the random code on line 3 exactly (#546).
- **Second host** (`pj-agdev/.local/p19/agpc-fetch.txt`): on agpc the worker's
  `wfe conversation attachments --check-bytes` reported both files verified;
  `fetch` through the mount, `fetch` from a process with no share and no
  registry (downloaded from the coordinator) and `curl` through nginx all gave
  sha256 `91a21064…` for the image (= the original file) and `a6103aa9…` for
  the text file. The HTTP response had `image/png`, `inline`, `nosniff`,
  `sandbox` and `x-attachment-sha256`.
- Existing challenge attachments on live: `20261024-101244-hqyvjt` #344 still
  reads `ch-0004` at `8c5743557006` with its latest `d994fdac`;
  ponderer/conduct/run-029 (with a challenge input) checks `usable`.

## Observed limits

- forge/freework/run-005 answered in its report without performing its nodes
  (held "interrupted"; cancelled by Omni as a finished check), and wrote its
  fetched copy under `devdocs/runs/…/files/` with `--out` — inside the run's
  tracked folder, though the default and the reference point to ignored
  `.local/`. ponderer/conduct/run-067 replied without starting its `ponder`
  node and was held likewise. Both are agent behaviour around nodes, not file
  access; they are recorded, not corrected here.
