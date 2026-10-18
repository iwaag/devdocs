# workflow_editor p14 ex1 — a concise Simple context view: final report

## Result

Context now reads like the rest of the Simple interface. Each section is a
list in the shape of the Workflows list — a title, one short line and one
action — and anything longer opens in a side panel beside the page, where
it can be read, edited, and its evidence inspected without leaving the
editor or the run. cagent's blocks are overviews with the observation
report moved behind them, and a consumer run still chose the right
resource and saw the risk that mattered.

- **Rows.** Title at the workflow-title level; the manager; the scope only
  where the grouping does not say it; at most two exceptions (*Unsaved
  changes*, *Last refresh failed*, *Partly verified*, *Not verified*, *No
  evidence recorded*, …). A short instruction (two rendered rows or fewer,
  wide characters counted twice) reads in full in its row with *Edit*;
  everything else is *Open*. No times, editors, runs, sizes or Remove in
  the list. One *Add* per list with a kind choice.
- **Panel.** Title, kind · manager · scope; the body as prose; *View
  resource details*, *Ask <manager>*, *Edit*; *Evidence & history* (last
  editor, observations with status and coverage, check, refresh,
  revisions). Editing (with Remove, size, conflict recovery and the notes
  on someone else's block or a wider scope) happens there. Esc/Close keeps
  drafts and returns focus; Detail adds hashes, sizes and references.
- **Sections.** The repository-owned purpose/goals/guides behind one row
  (the host's own controls and save, opened in the panel); guide content
  and the next-run preview as rows; "only in this environment" said once
  with help; inherited scopes in one list that folds behind *Show all*,
  keeping short instructions. Run card and overview use the same rows and
  panel with shorter copy; actual and next attempt stay separate.
- **cagent.** The guide distinguishes overview from detail (relevant
  resources, the uncertainty that matters, where to read more) with the
  screenshot as its failure reference, and — from run-006 — keeps what was
  last seen working versus failing. Forge's block went from a probe report
  to two routes, their probe-time difference and the unknowns; evidence,
  times and details unchanged.

## Verified

- Fixture checks in Chromium and WebKit, 1400 px and 390 px, all pass:
  `contextBlocks` 40/44, `contextEditors` 70/74, `contextRows` (new)
  17/19, `context` 7. Unit tests 401/401 (also in the release build).
- Disclosure changes no revision or fingerprint; cagent's publications
  change the next attempt and leave earlier snapshots intact.
- Installed release `ebfe8a5` on the LAN route, read-only, both engines
  and widths: rows, panel, details opening, evidence, the consumer run,
  no horizontal scroll, no page errors.
- Consumer `freework/run-003`: given both cagent blocks, picked SwarmUI
  (the only route taking a reference sketch), named its failed API as the
  blocking uncertainty, its 4-day-old proof, sources and `cagent/consult`;
  it did not need the details. Details read locally and from agautolab1.

## Revisions

agdevworld `cfcc2e9`, `ebfe8a5` (installed, pushed); pj-agdev `3ec5e68`
(pushed); pj-cagent `114208b`, `d98e6bd` (pushed); cagent runs
context/run-006, run-007; forge freework/run-003. Step reports:
[report1](report1.md) (rows and panel), [report2](report2.md) (entrances),
[report3](report3.md) (cagent and the consumer), [report4](report4.md)
(journeys, deployment, before/after).

## Limits and notes

- The inline threshold is a width estimate (96 columns, wide characters
  doubled), not a measurement of the rendered row; on a phone a two-row
  instruction can wrap to four lines, still in full.
- cagent's forge overview keeps one status code ("returned 200"); left as
  its judgement. The first rewrite (run-006) dropped a decision-relevant
  difference; the guide now names that trial, and run-007 restored it.
  Further guidance should follow further evidence.
- The panel is non-modal: the page behind it stays usable and scrollable.
  Only one item is open at a time; opening another keeps the first one's
  draft in its row.
- Step 1's first table claimed a WebKit 390 px pass that had actually run
  Chromium at desktop width (a zsh word-splitting slip); corrected in
  report1, and every configuration was re-run in Step 2 and Step 4.
- Unchanged from p14: no automatic cagent refresh, no snapshot retention.

## Rollback

Code: `scripts/wfe-release.sh activate <area>
ed688f6a3b9758d97ade18c0498b4be74831b7ee` (no data change; backup
`p14ex1-predeploy-20261018-233708` in the area). Content: the store's
history holds system `cagent-cluster` rev 4 and forge `cagent-resources`
rev 6; the guide changes are pj-cagent `114208b`, `d98e6bd`.
