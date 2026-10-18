# workflow_editor p14 — agent-maintained environment context: final report

## Result

cagent now publishes a short cluster summary for all projects and
project-specific resource advice into the WFE area's local context, and a
newly started forge run used it — with its age, its gaps and the route to
details — without asking anyone.

- **Blocks.** Local context is a set of independently managed blocks per
  scope (system, project, workflow): title, kind (*instruction* or
  *environment information*), manager, last editor, body, evidence,
  details, consultation workflow, per-block revisions. One file per block,
  atomic, optimistic concurrency on the content revision; a human edit of
  cagent's block keeps cagent as manager and records the human as last
  editor; cagent's next write from an unseen revision conflicts.
- **Delivery vs bookkeeping.** A block delivers its rendered section; the
  section's hash is the fingerprint input. Facts, evidence times/status,
  a failed refresh, details and consultation change delivery; a check,
  revision numbers and detail contents do not. No age string in prompts.
- **Evidence.** Publication, observation and verification times are kept
  apart; observation is the source's own; partial and failed refreshes
  keep the last content and are delivered as such.
- **Access.** `wfe context` (locally, or `--service`/`WFE_SERVICE` through
  the area's service from another host) and the HTTP block API; details by
  logical reference (`context:<scope>/<block>/<name>`); the consultation
  route is a workflow (`cagent/consult`).
- **GUI.** Separate cards in Simple with kind, manager, last editor,
  scope, freshness, failed refresh and size; Detail adds evidence, hashes
  and history; per-card edit, conflict recovery; the run card names the
  blocks that differ for the next attempt.
- **cagent.** Workflow `context` and guide `agent/context.md` in
  pj-cagent; five trial runs (first publish, unchanged check, refresh
  after an observation, a partial refresh, publishing what a probe
  established).

## Verified

- Tests 401/401; browser checks in Chromium and WebKit at desktop and
  390 px (`contextBlocks` 29/32, `contextEditors` 63/67, `context` 25);
  autolab executor 21/21.
- Live: conversion of the Developer's p13 instruction with history; a
  cross-host write from agautolab1 through the LAN service; cagent's
  publications; a check leaving the fingerprint unchanged
  (`4d687024…` before and after), an observation and a partial refresh
  changing it; the forge consumer's snapshot holding both cagent blocks,
  its log reading the detail, its answer naming agpc/ComfyUI (24
  checkpoints at 12:30Z), the unknowns and `cagent/consult`; installed
  UI checked read-only in both engines and widths.

## Revisions

agdevworld `d7dbcf9`, `92678d4`, `cdc123b`, `de014de`, `ed688f6`
(installed); agautolab `60ed2fb`; pj-agdev pin `55a87b9`; pj-cagent
`8e478fe`, `8d33bb9`, `7f6a721` (pushed). Step reports:
[report1](report1.md) (store, resolver), [report2](report2.md) (agent
access), [report3](report3.md) (GUI), [report4](report4.md) (cagent and
the consumer), [report5](report5.md) (deployment and verification).

## Limits and follow-ups

- **Pushes pending.** The classifier refused pushing agdevworld,
  agautolab, pj-agdev and devdocs; the commits are local.
- **No automatic refresh yet.** Refresh is an explicit `wf-cagent.context`
  request; triggers on scan/change results, new projects and intent
  changes (coalesced, no publication loops) are the next step.
- cagent's first instinct on a mixed probe was all-or-nothing (run-004);
  the guide now says a partial refresh publishes what it established
  (run-005 confirmed). Further guidance should follow evidence.
- Manager/last editor are records, not authentication; any writer may
  edit any block (by design in this phase).
- The forge consumer's final report and cagent's `published.md` are in
  their repositories and name hosts; both repositories already do so
  (pj-cagent is private). Block bodies and details stay in the area's
  store.
- Home and editors re-read blocks on their usual refresh (Home every
  20 s); a publication does not push a live event.
- Snapshots still have no retention policy (from p13).

## Rollback

Activate the previous release (`wfe-release.sh activate <area>
f998049…`) **and** restore `contexts/` from the `p14-predeploy-*` backup:
p13 code cannot read blocks. Blocks written since would be lost unless
copied first; the p13 files are also kept in `contexts/archive/p13/`.
cagent's `context` workflow can stay (unused) or be reverted in pj-cagent.
