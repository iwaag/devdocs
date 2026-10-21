# Argue p4 ex2 — readable discussion discovery and entry

## Objective

Implement the UI direction agreed in the conversation following p4 ex1.
The Developer supplied a screenshot of the discussion list and said it
discouraged reading: truncated titles, technical last-message previews,
equally prominent cards and operational trials mixed with ordinary topics.

Make it easy to decide which discussion to read, understand its purpose
and current outcome, and participate when a response is needed. Carry that
clarity through the first screen inside a discussion.

This episode plans changes to the existing aghangout and WFE applications.
Writing the plan does not start implementation or deployment.

## Evidence and working conventions

The inspected aghangout implementation currently:

- shows a three-column discussion card layout over the lobby artwork;
- lists at most 12 discussions, with no way in that component to reach the
  remaining entries;
- puts member names, message counts, invitation counts and relative time
  ahead of a preview of the last message's first line;
- truncates that preview to 90 characters;
- receives state, members, counts and the last message in `DiscussionRow`,
  but no explicit purpose category or source-linked reading overview;
- provides History, Progress, Invitations, Members and Records inside a
  discussion, including verbatim speech and separately marked syntheses.

Use `devpolicy/styles.md`, `devpolicy/terms.md`, `devdocs/README_DEV.md`,
`pj-agdev/.local/devenv.md` and `localrule.md` as the development context.

- Generated documentation is English; communication with the Developer is
  Japanese. Japanese UI labels and Japanese reader-facing overview text are
  explicitly part of this agreed design; original speech retains its language.
- Proceed through authorized implementation, checks, deployment and
  appropriate commits/pushes without approval checkpoints.
- Backward compatibility is unnecessary; migrate existing data explicitly
  and remove superseded presentation paths where replaced.
- Before working on local services, inspect state through Nautobot/nctl
  using `pj-clusterintent/nctl/README.md`. Refresh relevant observations
  when their age makes them inadequate for the work.
- Keep machine-specific details, backups and raw evidence in ignored files;
  preserve unrelated edits and ongoing discussions.
- Finish with a deployed UI accessible from another LAN machine and direct
  review links. Record Omni work for in-system agents with the required
  one-line handoff note in episode reports.

## Product decisions

### A readable list

Replace the discussion grid with a single-column list on a quiet, opaque
surface. Remove the artwork behind reading content and the repeated purple
border/stripe on every item. Keep the broader character-based experience
where it helps the conversation; this episode concerns discussion discovery
and entry, not a redesign of every personal conversation.

Each row presents, in this order:

1. A descriptive title, allowed two lines at ordinary desktop widths.
2. One or two lines explaining the current conclusion or the purpose.
3. A compact status and, when applicable, who is expected to respond.
4. Secondary metadata: domain participants, message count and update time.
5. A clear action to read or continue the discussion.

Do not hide the information that distinguishes similar titles. On narrow
screens, allow additional wrapping rather than cutting every title at the
same point. Keep a complete accessible title and an easy path to its full
text. Ponderer's routine presence as conductor need not occupy the member
list; show the conductor in the room's details.

Use Japanese labels such as `議論`, `進行中`, `終了`, `あなたの返答待ち`,
`議論を読む` and `新しい議論`. Use color sparingly for meaningful state;
state must also be understandable from text.

A presentation title may omit a trial suffix or technical identifier only
when an explicit display title has been recorded. Keep the original title
and identifier in details; do not strip arbitrary parenthesized text or
rewrite historical speech using string heuristics.

### Separate ordinary discussions and operational checks

Provide a normal-discussion view, with open/closed filtering, and a clearly
accessible view for operational checks. Ordinary discussions are the default.
Show counts and preserve filter/navigation state when opening a discussion
and returning. Make older discussions accessible through pagination or a
load-more control instead of silently dropping everything after item 12.

Persist a small explicit purpose category, such as `discussion` or `check`.
New discussions default to ordinary discussion; automated fixtures and
operational trials declare their category through the creation interface.
Allow later correction without modifying messages or starting agent work.

Classify historical discussions from their briefs and episode evidence in a
one-time migration map. Prefixes such as Trial, Fixture and Check are useful
inventory hints, not a permanent runtime classifier. A substantive discussion
used in an experiment is not automatically an operational check. Preserve
uncertain cases in the ordinary view until their purpose is established.

### Purpose and outcome instead of the last-message preview

Use a short source-linked overview shared by the list and the discussion
entry. Its content can include a readable title, purpose, current outcome
and unresolved points. Keep it separate from original messages and from
authoritative execution state.

- WFE stores and serves the overview and its provenance. Ponderer can
  publish or revise its text through discoverable tools, using its existing
  conducting or synthesis work where appropriate.
- Record author, source message IDs, the covered conversation horizon and
  update time. An overview is an interpretation with sources, not a new
  participant utterance or proof of agreement.
- When later messages arrive, keep the earlier overview readable and mark
  its coverage, for example `ここまでの要約・以降に発言3件`.
- An absent overview falls back to a labeled excerpt of the opening request
  or brief. It must not invent a conclusion or treat the last utterance as
  the group's decision. If there is no brief, use an honest empty state.
- Do not invoke a model when someone opens or refreshes a list. Avoid a
  mandatory extra synthesis run for every message. Expose explicit refresh
  work and update opportunities within existing workflows.
- Preserve disagreements, uncertainty and material open work in summaries.
  Japanese condensation of English sources must remain attributable to the
  original text, accessible in one action.

Inspect existing synthesis and archive contracts before choosing the exact
overview schema and publication command. Reuse their provenance conventions
without requiring a Git clone or per-discussion model call for list reads.

### Immediate orientation inside the discussion

At the top of the room, show a compact overview:

- what this discussion is trying to establish;
- the current conclusion and unresolved points, if available;
- the next recorded response or work that is awaited.

Below it, make the conversation directly readable and keep the composer easy
to reach. Opening a completed discussion should reveal its outcome and
conversation without requiring step-by-step playback. Preserve access to
character presentation, verbatim history, progress and detailed records.

Derive status and awaited responses from WFE's existing invitations,
questions, activities and obligations. Distinguish an open room from active
execution, and a closed room from completion of its commissioned work. A
human-answer badge appears only when the recorded requester/receiver is the
person; an agent requester must not be labeled as waiting for the user.

Metadata edits and reading the UI must not send messages, invite participants
or reopen a discussion. Existing reply routing, retries, drafts, IME input
and historical attribution continue to work.

## Step 1 — inspect contracts and specify the screens

1. Inspect current published aghangout/WFE revisions, project instructions,
   discussion APIs, list queries, creation paths and ponderer's guides.
2. Capture the current list and room at desktop and narrow widths, using
   the Developer's screenshot as the problem reference.
3. Produce concrete screen fixtures for the list and entry overview: long
   Japanese titles, mixed-language content, similar topics, missing and stale
   summaries, waiting for a person, failed work, and closed rooms with work
   outstanding. Use synthetic data clearly identified as fixtures.
4. Settle the category and overview schemas, publication/read paths,
   provenance, status derivation and update behavior. Identify precisely
   which changes belong to WFE, aghangout and ponderer.
5. Inventory existing discussions and propose their category/display-title
   migration map from evidence. Include ways to browse all historical rows.

Deliverable: `report1.md` with screen fixtures, data contracts, migration map
and affected components. Visual review is evidence, not an approval gate.

## Step 2 — implement metadata and overview publication

1. Add the minimal persistent category and overview support to WFE's normal
   store, CLI/API and discussion creation interfaces. Migrate stored data
   explicitly and retain original content and identifiers.
2. Include overview provenance and category in archive exports so the
   reader-facing interpretation can be studied and recovered later.
3. Extend the list read model with compact overview and meaningful status
   fields. Avoid fetching every transcript separately to draw the list.
4. Add discoverable overview publication/editing tools and update ponderer's
   relevant guides. Give the agent ownership of accurate summaries without
   imposing a universal discussion procedure.
5. Apply the reviewed historical classification and prepare source-linked
   overviews for a representative set of existing substantive discussions.
   Where Omni writes them, record authorship and the handoff note.
6. Test persistence, provenance validation, stale coverage, reopening,
   category correction, archive replay and metadata writes that do not
   dispatch agent work. Test human versus agent requester status.

Deliverable: `report2.md` with the implemented contract, migration results
and focused test evidence.

## Step 3 — implement the list and room entry

1. Replace the grid with the single-column hierarchy, opaque reading
   surface, Japanese labels, restrained status treatment and wrapping titles.
2. Add ordinary/check views, open/closed filtering, counts and access to
   older rows. Preserve filters and scroll position on return.
3. Render the shared overview in the row and at the top of the room, with
   visible missing/stale states and links to source messages.
4. Provide direct reading on entry and a clear route to participation;
   keep detailed execution information accessible without placing it before
   the purpose and outcome.
5. Check keyboard navigation, focus, contrast, touch targets, 200% zoom,
   narrow layouts and Japanese line wrapping. Do not rely on hover tooltips
   to recover essential text.
6. Run aghangout's normal check and browser fixture suites, extending
   meaningful cases for filtering, overflow, overview coverage, source
   navigation and unchanged posting/reply behavior.

Deliverable: `report3.md` with before/after screenshots and browser results.

## Step 4 — deploy and verify the reading experience

1. Inspect current Nautobot/nctl state and active work. Back up affected
   stores/configuration in ignored storage and record compatible code/data
   recovery steps.
2. Publish the project changes and deploy WFE and aghangout through their
   existing release mechanisms. Update project pins and service records
   where applicable.
3. From another LAN machine, verify normal discovery, filters, older rows,
   direct discussion links, source links and metadata persistence after
   reload. Confirm existing discussions and pending replies remain usable.
4. Exercise one bounded check discussion, explicitly categorized as such,
   to verify participation and overview publication through normal agent
   work. Keep its records in the check view and leave unrelated work alone.
5. Compare the deployed screen with the supplied screenshot using concrete
   reading tasks: identify a topic and its purpose, find its outcome, find
   what remains unresolved, and locate where to reply. Verify each answer is
   available without opening execution details or guessing truncated text.
6. Supply direct links and screenshots for immediate Developer review.
   Report objective checks separately from any unmeasured claim that users
   now find the UI more inviting.

Deliverable: `report4.md` with deployed revisions, LAN evidence and recovery
notes; `report.md` with outcomes, remaining limitations and review links.

## Completion criteria

- Normal discussion discovery shows readable titles and purpose/outcome
  text, with operational checks in a separate reachable view.
- Every discussion remains discoverable; the first 12 are not an implicit
  archive boundary.
- Overview text is attributable, source-linked and explicit about coverage.
  Missing or stale text never appears as a verified current conclusion.
- Reading and metadata updates trigger no agent work; status reflects
  existing WFE evidence, including closed rooms with outstanding work.
- Opening a room immediately explains its purpose, known outcome and next
  recorded need, with direct access to speech and participation.
- Desktop, narrow-screen, keyboard and Japanese text checks pass, along
  with the relevant WFE and aghangout tests.
- The result is deployed, checked from another LAN machine and reviewable
  through direct links. Documentation, guides and intended commits/pins are
  updated, with no duplicate legacy presentation path where replaced.
