# Challenge p4: titles from the person's wish

## Outcome

A person starts a discussion by writing their wish and sending it. A title
is supplied automatically so the conversation is recognizable in the list
and useful when revisited. Naming the conversation is never a prerequisite
for expressing the wish.

Source: `braindump.md` and the implementation proposal in the discussion.

## Existing implementation

- The managed `pj-aghangout` project's `app/src/room.ts` rejects a new
  group/facilitated room when its title input is empty. Its delivery path
  already has a fallback based on the first line of the outgoing message.
- WFE's `workflow_editor/server/conversationService.ts` requires a title
  when creating a room. Direct conversations already derive one from the
  opening message when none is supplied.
- A conversation has an original title and an optional display title.
  `wfe conversation meta --title` and `overview --title` can change the
  latter, with attribution and history. Lists and rooms use that display
  title when present, and the archive includes it.
- Ponderer's `agent/conduct.md` already owns the discussion overview and
  describes display-title editing.

## Intended behavior

1. The new-room setup has no title field. Focus starts in the message
   composer; existing participant and facilitation choices remain usable.
2. Sending a nonempty wish immediately supplies an original title from its
   first nonempty line, normalizing whitespace and limiting its length.
   Preserve readable Unicode characters when shortening it. The original
   message is stored without this title-specific transformation.
3. Ponderer supplies a short Japanese display title expressing the wish
   when preparing the first overview. This uses the existing conductor
   turn; there is no separate naming run, service or synchronous model call
   on submission. Until then, including when that turn fails, the original
   title remains usable.
4. People can edit the display title through the existing details panel.
   Automatic initial naming does not overwrite a title already edited,
   including an intentional reset to the original title. Later overview
   publication does not repeatedly rename the conversation.

## Implementation sequence

### 1. Remove the naming prerequisite in aghangout

- Update `app/src/room.ts`: remove the title input, its required validation,
  focus handling and title-specific draft wiring. Rewrite the new-room
  instructions to invite the person's wish.
- Derive the original title from the outgoing message before storing the
  outgoing creation request. Keep the same title, conversation identity
  and message token on delivery retries.
- Update related outgoing types, fixtures and browser checks as needed.
  Preserve pending messages and their attachments across the change;
  obsolete title-only drafts need not remain a supported feature.
- Cover both ordinary new discussions and the entry point that starts a
  discussion from a challenge. Keep the existing one-to-one behavior.
- Keep WFE's nonempty creation-title contract: the client supplies the
  derived value. No general creation API redesign is needed.

### 2. Complete titles through the existing conductor

- Update ponderer's conduct guide with the title's purpose, its ownership
  and the available command. Describe these as facts beside the overview
  responsibility, rather than adding an approval or mandatory workflow node.
- Reuse display-title storage, metadata history and archive export. Do not
  introduce a parallel title store or regenerate historical room titles.
- Inspect initial naming against concurrent manual edits. Existing
  `setMeta` is unconditional, and `publishOverview` can apply its title even
  for a duplicate overview. A guide-only read-before-write is insufficient
  to guarantee preservation of a human edit.
- Add a narrowly scoped conditional initial-title write if needed in the
  existing service/CLI path. Check title-edit history and write within the
  same transaction, so a manual edit or reset that wins the race is retained.
  Use existing metadata history rather than a schema migration where
  possible. Ordinary explicit title editing keeps its existing semantics.
- Ensure a skipped title completion does not prevent publication of the
  overview. Document any added command option in its help and the
  conversation contract, with one authoritative usage description.

### 3. Verify the changed behavior

- Run the affected projects' required checks and relevant existing browser
  checks. Add focused regression coverage for titleless submission and
  conditional naming where behavior or user data could regress.
- Verify leading blank lines, multiline Japanese text, long text and
  Unicode truncation. Empty/whitespace-only messages still do not submit.
- Exercise a failure after room creation but before successful message
  delivery. Retrying must preserve the wish and attachments and produce one
  room, one opening message and the existing response request semantics.
- Verify that the fallback title appears immediately, ponderer's normal
  first turn completes it, and the same display title appears in the room,
  conversation list, WFE home and exported archive.
- Check manual edits before and during completion, intentional title reset,
  duplicate overview submission and later overview updates. Human naming
  must survive these cases.
- Confirm that unavailable or failed facilitation leaves the room usable
  with its original title and that title changes themselves start no run.

### 4. Deploy and demonstrate

- At execution time, inspect service placement, observed state and relevant
  deployment revisions through `pj-clusterintent` Nautobot or `nctl`, using
  `pj-clusterintent/nctl/README.md`. Treat local environment notes as context,
  not proof of current running state.
- Publish the managed aghangout and ponderer changes using their existing
  project/deployment mechanisms. Deploy WFE and update affected worker pins
  if the conditional title command changes executable code.
- Record previous and new revisions and the recovery procedure. No database
  migration is expected; if implementation requires one, document and back
  up the affected data before cutover.
- Verify from the normal LAN entrance, including another LAN PC. Create a
  clearly attributed operational-check room, send a wish without a title,
  and observe the fallback and subsequent automatic completion. Preserve
  evidence and close the check room when finished.
- Update relevant development documentation and write `report.md` with the
  implemented behavior, validation, limitations and a concrete review path
  requiring few actions. Keep machine-specific addresses, local paths and
  operational evidence in ignored local files.

## Completion criteria

- Starting a discussion requires no human title input.
- Every submitted wish produces a recognizable fallback title immediately.
- A normal first conductor turn supplies a useful Japanese display title
  through the existing conversation metadata mechanism.
- Manual title changes survive automatic completion and retries.
- The deployed LAN interface demonstrates the behavior, with checks and
  recovery information recorded.

## Execution policy

Follow `localrule.md`: generated files are English and communication with
the Developer is Japanese. Implementation, commits, pushes, deployment and
verification proceed without adding human approval gates. Backward
compatibility is not a goal; preserve user content and existing evidence.
Keep the service accessible from other LAN PCs and finish execution with a
deployed result the Developer can inspect with few actions. This document
records the plan; creating it does not itself execute the implementation.
