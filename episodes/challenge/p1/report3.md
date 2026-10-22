# Challenge p1 — Step 3: carrying challenge references into work

## Delivered (agdevworld `0bffa86`)

**Structured attachments on messages.** Conversation store **v11** adds
`attachments`: per person's message, each attached challenge with the
commit it was sent with, the title it had then and the sha256 of its file at
that commit. They are written in the message's own transaction and never
changed. Conversations and discussions share the table.

- HTTP: `challenges` on `POST api/conversations/<id>/messages`,
  `POST api/discussions/<id>/messages` and the discussion-create route with a
  first message: `["ch-0001", "ch-0002@<commit>", {id, at}]`. Without a
  commit the current revision is pinned at acceptance.
- CLI: `wfe conversation post … --challenge <id>[@<commit>]`,
  `wfe discussion post … --challenge …`.
- An unknown, unpublished or unresolvable reference refuses the message
  (nothing stored); a resubmitted message (same token) keeps one set of
  attachments.

**The receiving run gets the pinned revision.**

- A conversation turn, a discussion invitation's new run, and an ordinary
  request (`wfe request … --challenge`, the request route's `challenges`)
  pass each attachment's text at its commit to run creation. The run folder
  gets `attachments/<id>@<commit12>.md` (hash-checked against the pinned
  sha256), and `run.json` `input.attachments[]` records
  `{kind, id, commit, title, sha256, file}` — what the run received is in its
  input record.
- The attempt brief names them: the file, `wfe challenge evidence <id> --at
  <commit>` for evidence (on demand, never copied), and `wfe challenge show
  <id>` as the separate read of the latest revision.
- A reply delivered to a run already working (a discussion answer to the run
  that asked, or an instruction) carries the references in its
  `source.attachments`; the answer and instruction briefs list them with
  `wfe challenge show <id> --at <commit>`.
- A later edit of the challenge never changes an earlier turn's file or
  input. The conversation and discussion views list each message's
  attachments with the **latest** revision beside the attached one, so a
  client can tell "the version sent" from "the version now".

**Exports.** Discussion archive transcript `message` records carry
`attachments` (challenge id and commit, which resolve in the challenge
repository whether or not the source project still exists). Direct
conversations have no export yet (archive ownership for unfacilitated chats
belongs to the chat/argue unification phase); their attachments stay in the
store and in each run's input.

**Project context.** No new mechanism: a context block can carry a short
pointer (`ch-0001`, `wfe challenge show ch-0001`); the body stays in the
challenge. Nothing dumps open challenges into prompts.

## Checks

- `test/attachments.test.ts` (4): a conversation turn receives the pinned
  file and input record, the brief line names them; a later edit leaves the
  earlier turn's file unchanged while the view reports the newer latest;
  an explicit `@<commit>` pins that revision; unknown/malformed/absent
  references refuse the message; a discussion invitation's run receives its
  revision; a reply to the run that asked carries the references in the
  answer's source; the discussion view and the archive transcript keep
  them; reading and attaching create no job.
- Old-store simulations in two existing tests now also drop `attachments`.
- Whole WFE suite: 531/531; `tsc` clean.

## Compatibility

Store v11 is a migration: an older WFE release refuses the migrated store,
so a rollback needs the pre-deployment store backup (step 6).

## Next

Step 4: aghangout components for discovery, editing and attaching.
