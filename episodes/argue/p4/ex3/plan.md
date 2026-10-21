# Argue p4 ex3 — workflows alongside the discussion

## Objective

Implement the Developer's request following ex2: show workflows referenced,
created or edited during a discussion in the same screen as the conversation,
with an intuitive connection to the turn in which the action occurred.

Provide workflow cards attached to relevant speech or recorded activities
and a shared graph panel. A reader can see what the agent consulted, what
it created, and what changed without leaving the discussion for the workflow
editor. Preserve the workflow version that existed at that turn.

This plan covers implementation, migration where justified, deployment and
verification. Creating this document does not start those operations.

## Context and conventions

Ex2 made aghangout's discussion entry overview-first, with directly readable
speech. Preserve that reading path while adding workflow evidence.

The inspected implementation already provides useful foundations:

- Discussion messages can name their run, attempt and node.
- Discussion links connect messages to runs, while activities name source
  messages and conductor decisions can name a graph revision.
- WFE keeps workflow definitions and run revisions, including frozen
  execution graphs and recorded node state.
- WFE's `src/canvas.ts` renders graphs with a read-only mode, node selection,
  focus, pan/zoom and execution-state overlays. It still has editor-specific
  dependencies; inspect these before choosing a reusable boundary.

These facts do not mean that every arbitrary file read is recorded as a
workflow reference, or that the exact historical execution state is always
recoverable. Identify those gaps explicitly.

Follow `devpolicy/styles.md`, `devpolicy/terms.md`, `devdocs/README_DEV.md`,
`pj-agdev/.local/devenv.md` and `localrule.md`:

- Generated documents are English; Developer communication is Japanese.
  Reader-facing UI labels follow the Japanese discussion reading path.
- Proceed through authorized implementation, tests, deployment and intended
  commits/pushes without approval checkpoints. Review records are evidence.
- Backward compatibility is unnecessary. Preserve historical authorship,
  IDs, source artifacts and unrelated active work during changes.
- Before local service work, inspect Nautobot/nctl state following
  `pj-clusterintent/nctl/README.md`, refreshing observations as needed.
- Keep machine details, credentials, backups and raw operational evidence
  in ignored files. Finish with deployed links usable from another LAN PC.
- Give agents discoverable tools; add guidance grounded in this feature's
  contract and demonstrated behavior, not a fixed discussion procedure.
- Record Omni work for in-system agents with the required one-line episode
  handoff note.

## User experience

### Cards attached to the conversation

Place a compact workflow card immediately below the associated message.
Show a readable workflow name, the relationship (`参照`, `作成`, `編集`,
or `実行`), its version, a short explanation and a small graph preview.
For an edit, show a concise factual change such as `結果レビューを追加`
only when supported by the recorded change; otherwise show counts and names.

Selecting a card opens the graph in a right-hand panel on wide screens and
a bottom sheet or full-width panel on narrow screens. Keep the conversation
position and unsent draft intact. Multiple cards can share one panel; do
not create a full editor instance for each message.

The panel identifies the source turn and can scroll back to its message.
Selecting another card switches the graph and highlights the selected card.
Selecting a referenced node reveals its details and, where explicitly
recorded, the related messages. Do not infer that every message in the same
run discussed every node.

### The version at that turn

Open the exact referenced version by default, labeled `このターン時点`.
Provide a separate `現在` action when a current target can be resolved.
Never silently substitute current main, the latest draft or the latest run
revision when the historical definition is unavailable.

Distinguish:

- a reusable workflow definition from a particular execution of it;
- a committed/published definition from a captured draft;
- definition structure from execution state;
- historical state from current state.

A historical definition may be available when historical node states are
not. In that case show the definition and an explicit state-unavailable
label. Any current execution overlay is explicitly labeled as current.

### References, creation, edits and execution

- **Reference:** show the version actually identified by the reference
  record, with selected nodes highlighted when specified.
- **Creation:** show the new graph and its provenance. A proposal or draft
  is not labeled as published or executed.
- **Edit:** show before and after versions, highlight added/changed nodes
  and connections, and make removed elements visible in a before view or
  removal list. Provide a compact textual change list beside the graph.
- **Execution:** show the linked run and its graph/state with the time basis
  stated. Starting a run is not itself a workflow-definition edit.

Compare parsed definitions by stable node IDs and edge endpoints. Distinguish
behavioral changes from layout/style-only changes; describe a new ID as an
addition/removal unless a rename relationship is explicitly recorded. Include
changed node properties, not just counts. Keep authored nodes and generated
execution steps distinguishable in the view and comparison.

The initial feature is a read-only embedded graph. The existing full editor
remains available as a secondary action. Inline authoring, conflict handling
and saving edits from aghangout are outside this episode.

### Preserve conversation readability

Distinguish workflows that are the subject of the discussion from the
conductor's own progression workflow. Show subject-workflow cards beside the
relevant speech. Group routine progression changes into a compact expandable
entry instead of adding a large card for every ponder step.

An activity with no spoken message remains visible as a recorded activity,
with its source messages and turn identity. Do not manufacture a message or
attach it to the nearest message by timestamp. Multiple operations in a turn
can be grouped visually while retaining their individual records and order.

Large graphs open at a readable scale focused on the relevant nodes, with
fit/pan/zoom controls. Include a textual node/change view for accessibility
and graph-load failures. Graph loading must not block reading or composing.

## Evidence and data contract

WFE owns the durable association; aghangout presents it. Extend or reuse the
existing discussion link/activity model after inspecting its semantics.
Avoid a second workflow store or an independent source of execution truth.

Each association needs enough information to establish:

- discussion ID and explicit source message(s), activity or turn;
- actor and originating run/attempt where available;
- relationship: referenced, created, edited or executed;
- role in the discussion: subject work or conductor progression;
- project/workflow identity and definition-versus-run identity;
- immutable version locator and content digest; for edits, before and after;
- relevant node IDs, optional explanation and source operation identifier;
- ordering/time and idempotency identity;
- whether the evidence came from a tool operation or an explicit authored
  reference claim.

Prefer existing run revision snapshots and immutable repository revisions.
If an unpublished draft can be referenced, capture its bytes with a digest
using existing artifact mechanisms, mark it as a draft and define retention.
A mutable path alone cannot prove what was seen at that turn.

Record creation, revision and execution associations from the existing
operations when a discussion context is known. Publish the association only
after the underlying operation succeeds, with retry/recovery behavior that
does not duplicate it or lose a successful operation's association.

Provide a discoverable explicit reference/attachment operation for workflows
consulted as discussion material. A successful tool read can establish that
the artifact was retrieved; it cannot prove comprehension. An authored
attachment is a declared reference. Do not claim comprehensive tracking of
arbitrary shell reads or infer references from workflow names in prose.

Explicit association with a message may happen after the operation. Until
then, retain the known turn/activity context. Validate identity and source
relationships using existing WFE recording rules, including cross-project
work and delegation. No association write should post speech, start a run or
reopen a discussion by itself.

Export associations and immutable locators through the discussion archive.
Define how a fresh reader resolves them after source branches change or a
project retires. Preserve available snapshots; show honest unavailable states
when evidence was never captured. Historical backfill uses only proven
message/activity/revision links, with provenance for any added record.

## Step 1 — inspect and settle contracts

1. Inspect current published WFE, aghangout and ponderer revisions, project
   instructions, graph rendering, run snapshots, revision events, discussion
   associations and API access rules.
2. Map which reference/create/edit/execute cases are already provable and
   which require new recording. Include definitions with no run, drafts,
   delegated work, operations before speech and multiple revisions per turn.
3. Choose the smallest extension to associations, artifact resolution and
   archive exports. Specify idempotency and interrupted-operation traces.
4. Choose a reusable read-only graph rendering boundary, including layout,
   style assets, state overlay and node selection. Avoid copying the editor's
   graph renderer into aghangout or embedding its entire navigation shell.
   A dedicated embed route is acceptable if it supports exact versions,
   selection and contained navigation.
5. Produce screen fixtures for cards, side panel, narrow-screen panel,
   change view, multiple workflows and unavailable evidence.

Deliverable: `report1.md` with contracts, recording matrix, renderer choice,
screen fixtures, affected components and migration/recovery approach.

## Step 2 — implement associations and immutable reads

1. Implement persistent associations and their CLI/API reads/writes under
   existing WFE ownership. Add explicit reference capture and integrate
   recording into relevant create/revise/execute paths.
2. Support immutable graph resolution for committed definitions, run
   revisions and captured drafts. Never execute workflow content to display
   it, and retain existing path/access validation.
3. Provide the semantic before/after comparison and explicit availability
   of historical/current execution state. Preserve version and time labels.
4. Export associations and make them inspectable from archived discussions.
   Backfill only relationships justified by existing records; do not rewrite
   old speech or treat time proximity as evidence.
5. Update tool help and concise ponderer/participant guidance so agents can
   attach workflows they discuss. Record their actual use in later trials.
6. Test operation success/failure, duplicate delivery, crash recovery,
   cross-project and delegated associations, before-message recording,
   missing versions, retired sources and metadata writes with no dispatch.

Deliverable: `report2.md` with implemented contracts, evidence-preservation
checks and focused test results.

## Step 3 — implement embedded graph interaction

1. Reuse the chosen WFE renderer in aghangout's read-only panel, with
   lazy-loaded graph data and compact card previews.
2. Add message/activity cards and bidirectional selection. Preserve source
   message navigation, conversation scroll, filter return state and drafts.
3. Implement historical/current switching, before/after display, node and
   edge changes, node details and the textual fallback.
4. Group conductor progression separately from subject workflows. Make
   activity-only records understandable without turning them into speech.
5. Verify long node labels, large graphs, missing nodes, multiple cards,
   loading/errors, keyboard focus, Escape/close behavior, 200% zoom and
   narrow touch layouts. Do not communicate differences by color alone.
6. Run relevant WFE and aghangout checks, with browser cases proving that a
   past turn stays on its pinned version after later workflow changes and
   that graph interaction does not trigger edits or discussion dispatch.

Deliverable: `report3.md` with interaction screenshots, browser evidence and
test results.

## Step 4 — deploy and exercise a bounded discussion

1. Inspect current Nautobot/nctl observations and active work; back up
   affected stores/configuration and record coherent code/data recovery in
   ignored notes.
2. Publish and deploy affected WFE, aghangout and guide changes through
   existing release mechanisms. Update intended repository pins and current
   developer documentation.
3. Run a clearly categorized check discussion in which an agent references
   an existing workflow, creates or copies a draft, revises it, and links a
   safe bounded execution. Exercise a delegated project workflow as well as
   conductor progression, keeping their roles distinct. Give the task brief
   once and record any subsequent assistance.
4. Verify that the UI shows each operation at its evidenced turn, that the
   edit view identifies the actual change, and that a later edit does not
   alter the historical graph. Include an operation with no spoken message.
5. From another LAN PC, verify cards, panel, source navigation, before/after,
   reload persistence and narrow layout. Confirm that ordinary conversation
   reading and posting still work.
6. Give the Developer direct links to representative turns, screenshots and
   the full-editor secondary action. Report missing reference capture or
   historical evidence honestly instead of claiming universal tracking.

Deliverable: `report4.md` with deployed revisions, trial evidence, LAN checks
and recovery notes; `report.md` with outcomes and remaining limitations.

## Completion criteria

- A reader can inspect a referenced, created or edited workflow in the same
  screen as the discussion and identify its associated turn without opening
  the full editor.
- The initial graph is the evidenced historical version. Current definition
  and current execution state are explicit alternatives, never silent
  replacements for historical data.
- Creation, reference, editing and execution have distinct meanings backed
  by recorded evidence. Missing associations and unavailable versions are
  visible limitations, not inferred facts.
- Changes to nodes, connections and properties can be inspected, including
  removals; differences are understandable without relying on color.
- Subject workflows and conductor progression remain distinguishable, and
  routine internal changes do not overwhelm the reading path delivered in ex2.
- Associations survive retries and archives without duplicate operations or
  changes to original speech. Reads and graph interaction start no work.
- Relevant tests pass and the deployed experience is verified from another
  LAN PC. Intended commits/pushes, pins, help and documentation are updated,
  with direct review links and required Omni handoff notes.
