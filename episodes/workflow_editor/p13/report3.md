# p13 Step 3 — the matrix and attempt inspection UI

## What was built (agdevworld `78e9795`, `workflow_editor/src/contextView.ts`)

- **Entrances**: a *Context* button on the dashboard (`#/context`, choose a
  project), the project view (`#/context/<ws>`) and the workflow view
  (`#/context/<ws>/<workflow>`).
- **Scope matrix**: columns System / Project / Workflow / Run, rows Default
  (repository-owned) / Override (local, not in Git). Each default shows its
  delivery (included, reference, not set, unreadable), applicability
  (execution only / authoring only / both), origin and every declaration;
  project intent links to the project view, workflow intent to the workflow
  editor. The Run column states that runs have no pre-authored default and
  that run input is in each run's view. A workflow selector moves between
  workflows of the project.
- **Override editor**: edit in place, an *Unsaved changes* marker and a
  navigation veto, Save / Discard / Clear, the author name (stored per
  browser), the content hash the save replaces, revision history with exact
  content. A conflicting save shows what is saved now with *Load it* or
  *Save mine over it*. After a save the page says it applies to the next
  attempt of affected runs, that nothing was started or resumed, and points
  current-run steering at the instruction route.
- **Resolved preview**: the bundle the claim's resolver produces for the next
  run of the workflow, or for authoring; source chips with delivery; the
  exact text; *Copy text* / *Copy command* (`wfe context show … --rendered`)
  with the LAN-HTTP clipboard fallback. Authoring is labelled "available to
  copy, not injected".
- **Run view → Context** (Detail side pane, loaded when opened): *Used by
  attempt …* with snapshot id, delivery ("prompt delivered …; delivery, not
  proof it was read"), each source and the exact text; or "Unavailable: …"
  with the recorded identities (older attempts, or a snapshot on another
  host). *Next attempt with current settings*, labelled not yet applied,
  with added/changed/removed sources marked, the session consequence, and
  the steering route (Conversation link, `wfe run instruct`). Nothing is sent
  from there.

## Checks

`checks/context.ts` (private fixture over the LAN host name and plain HTTP):
22 checks pass in Chromium and in WebKit — entrances, matrix contents and
deduplicated guide, the unsaved marker, save message, preview update,
revision author, a conflicting edit and loading the current revision, copy,
the authoring label, the run's used-vs-next with an added override marked,
no horizontal page scroll at 390 px, no unexpected failed request or console
error. Screenshots: `agdevworld/.local/wfe-fixture/screenshots/p13-*.png`.
WFE `npm test` 394/394, `tsc` clean.
