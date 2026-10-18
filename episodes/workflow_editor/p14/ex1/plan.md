# Workflow editor p14 ex1 — restore a concise Simple context view

## Objective

Make Context as easy to scan as the existing Runs and Workflows lists. The
Developer's screenshot shows that p14 exposed too much at once: explanatory
copy, repeated scope labels, manager/editor metadata, observation and check
times, run references, sizes, full bodies and editing actions.

Improve both presentation and cagent's generated prose. Keep ownership visible
and preserve evidence without requiring the reader to inspect all of it before
choosing an item. This plan records the agreed direction; creating it does not
start implementation.

## Working conventions

- Follow `localrule.md`: generated files are English; communicate with the
  Developer in Japanese. Once implementation is requested, perform implementation,
  trials, deployment, commits and pushes without additional approval checkpoints.
- Deliver a verified result on the normal home-LAN route with direct inspection
  links and minimal actions for the Developer. Perform agent-operable setup and
  verification rather than handing over commands to run.
- Backward compatibility is unnecessary. Reuse and simplify the current context
  components rather than maintaining a second UI or store.
- Preserve existing human instructions, block identities, evidence, histories,
  snapshots, unrelated edits and explicitly held runs.
- Inspect service state through Nautobot/nctl before runtime work, following
  `pj-clusterintent/nctl/README.md`. Keep machine addresses, paths, raw runtime
  evidence and screenshots in ignored local storage.

## Presentation contract

### A list first, readable content on demand

Use the existing Workflows list as the visual reference: prominent title,
comfortable row spacing, short secondary information and one obvious action.
Do not solve density by shrinking text or simply enlarging the current cards.

The ordinary Simple view of a block contains:

- its title, at the same visual level as a workflow title;
- a concise manager label for agent-maintained information;
- its inherited scope where that scope is not already clear from the grouping;
- a short meaningful exception state when applicable;
- an Open action, or Edit for an already visible short instruction.

Short human instructions can remain fully readable inline. Long instructions and
environment information open into a reading surface. Select the inline behavior
by content kind and actual rendered length; a few explicit newline characters
do not make several long sentences short. Never hide part of an instruction
without an obvious way to read the full text.

An illustrative arrangement, not literal runtime content:

```text
Context                                            + Add

Resources for Forge                                 Open
cagent · Some availability is unverified

Developer instructions                              Edit
Write generated files in English unless requested otherwise.
All projects

Cluster overview                                    Open
cagent · All projects
```

Do not add an LLM call to generate list previews. Prefer a useful title and
existing structured status over another summary field that must be maintained.
The list is a navigation surface, not a substitute for the delivered context.

### Open, read, then edit or inspect evidence

Open a consistent in-place reading surface, preferably a side panel where the
existing layout supports it. On narrow screens use the available width. Preserve
the originating editor, selected run/node, scroll position and unsaved drafts.

Order the content as follows:

1. Title, manager and applicable scope.
2. Full body in normal readable typography, with a bounded line length and
   paragraphs or lists. Reuse an existing safe prose renderer where appropriate;
   do not render ordinary prose as a dense preformatted diagnostic block.
3. Relevant actions: View resource details, Ask cagent and Edit.
4. An expandable Evidence & history section with the supporting record.

Edit opens the existing draft and save behavior. Put Remove in the editing
surface. Use one Add entry that offers instruction and environment-information
types during creation instead of two permanent buttons per block list.

Keep meaningful operations available in Simple mode. Opening content, editing,
reading evidence and following detail links must not require switching the whole
application to Detail. Detail can expose technical diagnostics such as hashes,
source keys and delivered byte counts.

### Reduce repetition and metadata

- State local storage ownership once per section using concise copy such as
  “Only in this environment.” Put the fuller storage/application explanation in
  contextual help or the edit surface.
- Avoid repeating scope in the heading, subtitle and badge. Make a wider scope
  explicit when editing an inherited block.
- Keep management ownership visible. Put the last editor, publication time,
  producing run, observation timestamps and check history in Evidence & history.
- Remove line and byte counts from the normal list. Size diagnostics remain
  available in Detail or while editing, where they can inform a change.
- Group inherited content compactly when there are many entries. Preserve a
  direct way to read all inherited blocks, and do not hide a short useful human
  instruction solely because it is inherited.
- Apply the same disclosure approach to repository guide content, explanatory
  labels and context previews so simplifying agent cards does not leave another
  dense wall of text above or below them.

### Honest status without a telemetry row

The normal list shows concise exceptions such as a failed refresh, partial
verification or unsaved changes. Exact observation times and coverage belong in
the opened content/evidence view. Preserve the difference between unknown,
partial and failed states; hidden timestamps must not imply current availability.

Use existing evidence semantics. Do not invent a freshness threshold or label a
service healthy because its context publication is recent. Keep decision-relevant
uncertainty in the full overview and delivered context even when the list uses
a shorter status phrase. Put the detailed explanation beside its evidence.

## Generated-content contract

The screenshot demonstrates a second issue: a four-line body can contain an
entire observation report. Refine cagent's context guide based on this concrete
failure, retaining the p14 partial-observation guidance.

A system or project overview should communicate:

1. Which resources are relevant and what they are useful for.
2. Any uncertainty or constraint needed before acting.
3. Where to read details or consult cagent, using the existing structured links.

Move probe chronology, methods, per-host coverage, inventory lists and repeated
provenance explanations into the existing detail documents. Keep distinctions
such as installed versus available, or previously observed versus currently
verified, in the overview when they affect its meaning. Never achieve brevity by
turning a qualified observation into an unconditional availability claim.

Use short sentences and a small number of meaningful bullets or paragraphs.
Evaluate brevity by reading effort, rendered width and delivered content size,
not line count alone. Prefer editorial guidance over a new hard length gate or
automatic truncation.

WFE continues to deliver the saved overview and existing evidence contract.
UI disclosure alone must not change fingerprints or prompt contents. Deliberate
changes published by cagent follow normal next-attempt and snapshot semantics.
Do not redesign p14's storage, transport or observation model for this episode.

## Starting points

Paths are relative to the projects directory unless stated otherwise.

| Area | Starting points |
| --- | --- |
| Current behavior and evidence | `devdocs/episodes/workflow_editor/p14/report.md`, `report3.md`, `report4.md`, `report5.md` |
| Context components | `pj-agdev/agdevworld/workflow_editor/src/contextParts.ts`, `contextView.ts`, `runContext.ts` |
| Hosts and visual reference | Home/project/workflow/run views and existing Runs/Workflows list styles in `workflow_editor/src/` |
| Delivery contract | `workflow_editor/shared/context.ts`, `server/context.ts`, `docs/context.md` |
| Browser checks | Existing context-block, context-editor and context-overview checks in `workflow_editor/checks/` |
| Generated prose | Managed cagent project's `agent/context.md` and `context` workflow, discovered through the current registry |
| Release | agdevworld `README_DEV.md` and `scripts/wfe-release.sh` |

## Step 1 — establish the compact reading and editing surfaces

1. Inspect current component ownership, draft lifecycle, display-mode behavior
   and application panel conventions. Read applicable repository instructions.
2. Refactor the block presentation into a compact row and an on-demand reading
   surface, sharing one block state and one editable draft.
3. Match the existing workflow-list typography, spacing and action placement.
   Give opened prose a comfortable line length and ordinary body text size.
4. Move supporting metadata into Evidence & history, technical sizes/hashes into
   diagnostics, and Remove into editing. Consolidate Add actions.
5. Preserve conflict recovery, actual writer identity, manager identity, keyboard
   access, focus return, navigation protection and draft retention.

Deliverable: an existing block can be found, read, edited and inspected in Simple
mode without displaying its full audit record in the list.

## Step 2 — integrate all context entrances and remove duplicate copy

1. Apply the shared presentation to Home, project, workflow, run and overview
   surfaces. Keep actual-attempt and next-attempt context clearly distinguished.
2. Simplify section descriptions, guide presentations and inherited-scope groups.
   Avoid duplicated labels and independently editable copies of the same value.
3. Show short human guidance inline and long content on demand. Verify long
   unbroken lines, multiline text, Japanese text, many blocks and empty scopes.
4. Ensure meaningful exception states remain visible without constructing another
   long metadata row. Keep detailed evidence one deliberate action away.
5. Compare the result alongside the existing Runs/Workflows lists at the same
   viewport. Judge hierarchy and reading effort, not just element counts.

Deliverable: Context belongs visually to the existing Simple interface across
all entrances, while remaining fully operable there.

## Step 3 — improve cagent's overview/detail composition

1. Review the current system and forge publications and the evidence supporting
   them. Identify report-like prose and duplication of structured detail links.
2. Update cagent's guide with the overview/detail distinction and the specific
   screenshot failure reference from this episode. Keep tool syntax in help and
   avoid adding a rigid publication procedure.
3. Have cagent refresh the system and forge overviews using the existing tools.
   Preserve relevant evidence and unknowns; do not manufacture a new observation
   timestamp or require a new scan merely to shorten prose.
4. Check that detail documents retain the supporting information moved out of
   the overview and that their links work from the UI and a consumer environment.
5. Run a bounded consumer trial to verify that the concise context still supports
   useful resource selection and discovery of missing details. Inspect delivered
   context and detail access, not only the final answer.

Deliverable: short, useful cagent-authored overviews backed by readable details,
without weakening the evidence contract or replacing cagent with Omni prose.

## Step 4 — verify complete journeys and deploy

1. Run focused browser journeys in Chromium and WebKit at desktop and narrow
   widths: find a block, open it, read evidence, follow details, edit and save,
   recover a conflict and return to the original editor.
2. Verify drafts and focus survive refresh, panel close/reopen, mode switches and
   navigation as appropriate. Test keyboard operation and long content.
3. Verify UI-only disclosure changes do not alter saved bodies, delivery or
   fingerprints. Verify cagent's intentional updates affect future attempts and
   leave previous snapshots intact. Run required repository/release checks.
4. Inspect current service state through Nautobot/nctl and deploy through the
   normal WFE release process. Preserve local block data and retain rollback
   information for code and intentionally revised context.
5. Inspect the installed LAN route at the same viewport as the original
   comparison. Capture representative before/after evidence for the whole
   section and its reading surface, not only isolated component crops.
6. Commit/push the changes and relevant submodule pins, update context docs and
   appropriate local environment notes, and write `report.md` with verification,
   limitations and revisions. Provide direct LAN links in the Japanese handoff.

Deliverable: a deployed Simple interface the Developer can inspect immediately,
with useful refreshed content and no setup delegated back to the Developer.

## Completion criteria

- Normal Simple context rows visually match the hierarchy and spacing of the
  existing Workflows list; titles and primary actions are easy to find.
- Audit metadata, technical size counts, removal controls and long full bodies
  no longer appear on every row by default.
- Short human instructions remain readily readable. Agent ownership, inherited
  scope and meaningful uncertainty remain understandable without a metadata wall.
- Full content, editing, details and evidence remain accessible in Simple mode;
  opening them preserves the current task and unsaved work.
- Generated overviews describe useful resources, material uncertainty and routes
  to details; observation-report prose lives in the detail documents.
- Existing concurrency, history, snapshots and delivery semantics remain intact.
- Deployed desktop and narrow-screen journeys are verified, with direct links
  and a small number of inspection actions for the Developer.
