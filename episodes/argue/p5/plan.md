# Argue p5 — character dialogue as the short reading view

## Objective and scope

Source: [braindump.md](braindump.md) and the subsequent discussion with the
Developer. Make discussions easier and more enjoyable to read by presenting
each agent's contribution as a short, natural character utterance, with a
portrait and a chat bubble. Selecting the bubble switches to the original
full text in place. The display language may differ from the source language.

Character dialogue is presentation only. Original speech remains the input
to practical work, discussion progression, synthesis and replies. Manage
reusable character definitions as an independent managed project, and keep
generated dialogue outside version control beside the discussion archive.

This plan covers implementation, trials, deployment and reporting. Writing
this plan does not start implementation or deployment.

## Conventions and current foundations

Follow `devpolicy/styles.md`, `devpolicy/terms.md`, `devdocs/README_DEV.md`,
`pj-agdev/.local/devenv.md` and `localrule.md`.

- Write generated documents in English and communicate with the Developer
  in Japanese. Start the reading experience with Japanese character dialogue.
- Once implementation is requested, carry authorized work through testing,
  deployment and verification without introducing approval checkpoints.
  Commit/push as appropriate; record reviews as evidence, not permissions.
- Backward compatibility is unnecessary. Preserve source speech, authorship,
  identifiers, repository history and unrelated work.
- Before operating local services, inspect Nautobot or nctl following
  `pj-clusterintent/nctl/README.md`. Refresh observations when needed;
  recorded convergence alone is not an end-to-end browser check.
- Keep addresses, credentials, backups and machine-specific evidence in
  ignored local files. Make deployed results usable from another LAN PC.
- Give project agents discoverable tools and concise contracts. Do not
  prescribe a rigid creative process without evidence that it helps.

The inspected implementation has these starting points:

- WFE owns discussion records and their archive publication. The archive
  destination is now `pj-ponderer`; the former argue project is retired.
- aghangout owns the reader experience. Its normal discussion transcript
  currently displays full original speech; its overview is a separate,
  discussion-wide interpretation. There is no existing per-message generated
  summary to replace mechanically.
- `agdevworld-settings` contains character lore, portraits, room assets and
  a manifest. aghangout also embeds character definitions in
  `app/src/characters.ts` and copies assets into its build.
- The older argue p2 implementation demonstrated separation between real
  speech and character rendering, source references and restart recovery.
  Reuse applicable contracts and lessons, not the retired Zulip memo path.
- The planning investigation found aghangout and agdevworld-web converged
  in nctl. Recheck at implementation and deployment time.

## Experience and semantic contract

The ordinary transcript becomes a readable sequence of portrait thumbnails
and short character bubbles. Begin with one bubble per original message;
do not merge speakers or invent intervening responses. Aim for one to three
short sentences, allowing enough space to preserve an essential question
or qualification rather than enforcing a harmful hard length limit.

Preserve the speaker's position and the conversational function: proposal,
objection, request, reported result, uncertainty and unresolved question.
Details may be omitted; a proposal must not become an action, an attempted
action a success, or an unverified result a verified one. Character style
may change phrasing and emotional expression without adding factual claims
or changing agreement and disagreement.

Use original reply targets and relevant earlier original speech as context.
Do not recursively summarize previous generated bubbles. Historical reading
must not acquire knowledge of later outcomes that the speaker did not have.
Default human messages to their original text; character impersonation of
the Developer is outside this first implementation.

Selecting a bubble toggles its original text in place; provide equivalent
keyboard operation and an explicit full-original view preference. Preserve
scroll position, focus, drafts, source links and message IDs. Replies always
address the original message, regardless of the current display mode.
Make the presentation/original distinction understandable with concise UI
labels, without exposing implementation metadata in the normal reading flow.

Keep discussion overviews, status, unanswered questions and workflow cards
from p4 ex2/ex3 available. Their meaning and placement must not depend on
the generated wording. The optional playback view should consume the same
presentation records instead of generating a second interpretation.

Missing character definitions, pending generation or failed rendering must
leave the original readable and the discussion usable. Late results update
the relevant bubble without closing an original the reader has opened.

## Ownership and data flow

### Independent character project

Import the existing settings repository into project management, preserving
history and existing assets. Choose its final registered name during the
initial inventory and record the mapping. Its owned material includes:

- Stable character IDs, display names, nicknames, lore and speaking style.
- Portraits and other reusable presentation assets with provenance.
- A machine-readable manifest and documented consumer format.
- Workflows to add or update characters, normalize their data and validate
  the resulting manifest and asset references.

Keep character identity separate from runtime project identity. The consumer
maps WFE participants to character IDs; another entertainment interface can
reuse the characters with its own mapping. Migrate old agent-name mappings
explicitly rather than assuming names such as `agforge` still identify the
current participant. A participant without a mapping keeps a plain display.

aghangout consumes a pinned export or build artifact from this project.
Remove hand-maintained lore and asset mappings duplicated in application
source. Changing or adding a character should not require editing UI code.
Choose the smallest useful export format; a new asset registry service is
not required merely to share the definitions.

### Presentation renderer and storage

aghangout owns the rendering contract and presentation behavior. WFE supplies
original source snapshots and a separate UI-facing presentation path where
needed. Choose the worker hosting arrangement after inspecting the existing
executor and API boundaries; rendering must not consume the single practical
conversation slot or block delivery of new original speech.

The renderer receives only the necessary original sources, their identities,
reply relationships, character definitions and requested language. It emits
structured display results, not discussion messages or practical commands.
It needs no discussion-posting, invitation or workflow-mutation tools.

Store generated dialogue in an ignored location in the checkout holding the
current archive, for example `pj-ponderer/.local/character-dialogue/`.
Resolve that location through the configured archive destination, and choose
one canonical writer/location rather than independently writing into run
clones and archive clones. Document how it survives normal archive updates
and deployment, and how regeneration works after its loss.

Each result records at least:

- Discussion and original message IDs, speaker identity and source hashes.
- The earlier context/source horizon used for rendering.
- Character ID and settings revision, output language and renderer revision.
- Generated text, completion time and generation state/error when relevant.

Cache identity includes the actual inputs, including context and character
revision. Use durable jobs and atomic publication so retries, concurrent
readers and worker restarts do not duplicate results or expose partial text.
Changed source or settings can produce a new result without silently passing
off the old result as current. Keep older output only as useful local data;
there is no requirement for a Git history of generated dialogue.

Generation runs asynchronously for new speech, with bounded work for history
opened by a reader. Provide a discoverable retry/regenerate operation. Avoid
regenerating all historical discussions on a deployment or character update.
Measure latency and usage with a small real trial before choosing the default
rendering profile and scheduling policy.

### Practical-work boundary

An ignored directory alone does not provide this boundary. Keep display text
out of ordinary conversation APIs, transcripts, archive publication,
discussion summaries, retrieval indexes and practical-agent context assembly.
Presentation reads use a distinct path. Do not add generated dialogue to
ponderer's ordinary working context merely because its storage lives nearby.

Inspect filesystem exposure as well as API payloads: practical run workspaces
must not receive the presentation cache or character lore as conversation
material. This is separation of normal data flows, not a claim that an
unrestricted administrator cannot read the files.

## Execution sequence

### Step 1 — inventory and concrete contracts

Inspect repository/workspace state, project registration, archive checkout
lifecycle, aghangout release packaging and available rendering execution
paths. Recheck relevant service records using nctl. Identify active work
before modifying shared checkouts.

Record the final ownership, character-project name, participant mapping,
storage location, job host and API shapes. Select an existing discussion
containing long speech, disagreement, uncertainty, questions and English
source text for a complete reading trial. Keep a local baseline of its
original records and relevant current releases.

Deliverable: a short implementation note and fixtures sufficient to exercise
the contracts below; no approval gate between this and subsequent steps.

### Step 2 — import and normalize character management

Register the character repository as a managed project and add its creation,
editing and normalization workflows. Preserve assets/history, document the
consumer format and provide tools with usable `--help`. Migrate current
definitions and mappings, including an explicit fallback for characters
without portraits or lore. Replace aghangout's manual definition copies with
the exported definitions.

Verify that a representative character addition/update reaches a preview
without UI source changes, and malformed definitions or missing asset paths
produce actionable diagnostics. Do not require new artwork for all speakers
before the feature can ship.

### Step 3 — implement generation, cache and isolated reads

Implement source snapshotting, the renderer, persisted job state, ignored
storage and the presentation read/retry path. Keep original APIs unchanged
in meaning. Exercise real rendering on the selected discussion early, then
adjust context selection, brevity and voice from observed failures.

Verify source-to-result identity, setting/source invalidation, duplicate
requests, interruption recovery and failures. Demonstrate that generated
text and lore are absent from practical inputs and archive publication.

### Step 4 — integrate the reading experience

Add portraits, short bubbles, in-place original toggles and a full-original
preference to the discussion transcript. Preserve reply targets, citations,
workflow cards, unread position, drafts and responsive layout. Share the
presentation source with playback where applicable.

Exercise pending-to-ready updates, original-open state, unknown characters,
long original text, keyboard use and narrow screens. Avoid generating model
calls from repeated component renders or ordinary polling.

### Step 5 — evaluate one complete discussion

Render and read the selected discussion end to end in Japanese, then compare
every bubble with its source. Assess brevity, distinct character voices,
natural continuity and preservation of meaning. Record concrete defects and
the changes that fix them rather than adding speculative prompt rules.

Include a newly arriving message to verify asynchronous behavior. Measure
generation latency and usage, recording only cost figures actually supplied
by the provider. Check fallback and restart behavior with the same reader
flow. Expand to additional examples only when a coverage gap or failure
justifies it.

### Step 6 — deploy, verify and report

Run the relevant existing checks for changed components and focused boundary
tests. Commit/push the intended changes, deploy the affected worker/API and
aghangout release, and register changed service topology with Nautobot if
applicable. Preserve code/data recovery instructions and local backups where
state changes require them; backward-compatible code paths are unnecessary.

Verify the deployed discussion from another LAN PC: short dialogue, original
toggle, reply destination, workflow cards, language, new-message rendering
and generation failure fallback. Recheck service state with nctl and report
any pre-existing or newly introduced drift accurately.

Update developer documentation and local environment notes in their proper
locations. Write step reports and a final report in this episode, including
release revisions, the reading trial, boundary evidence, measured latency,
remaining limitations and directly usable deployed review links. Keep
machine-specific evidence and addresses in ignored operational notes.

## Completion evidence

The episode is complete when all of these are demonstrated:

1. The independent managed character project owns reusable definitions and
   provides working addition/update and data-normalization workflows.
2. A complete discussion can be read as concise, coherent character dialogue
   while preserving proposals, objections, questions and uncertainty.
3. Every generated bubble resolves to the correct original; switching views
   and replying preserve identity and reader state.
4. Display text is stored outside Git beside the configured archive and is
   excluded from practical prompts, normal conversation reads and archives.
5. Missing characters, generation failure and worker restarts leave the
   discussion usable without duplicate rendering or lost original speech.
6. The deployed experience works from another LAN PC, with reports and links
   that let the Developer inspect it with minimal effort.
