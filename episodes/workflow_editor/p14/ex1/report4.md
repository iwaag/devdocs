# p14 ex1 Step 4 — journeys, deployment, the installed route

## Journeys (fixture, before deployment)

All four context checks in Chromium and WebKit at 1400 px and 390 px
(Step 2 table): `contextBlocks` 40/44, `contextEditors` 70/74,
`contextRows` 17/19, `context` 7 — all pass. Together they cover finding a
block, opening it, reading evidence, following details, editing and
saving, recovering a conflict and returning focus to the row; drafts
surviving a live refresh, closing and reopening the panel, mode switches
and the navigation guard; keyboard-only open and close; long, Japanese,
unbroken and multiline content; many blocks and empty scopes.

Disclosure versus delivery: opening and closing blocks changes no
revision and no next-attempt fingerprint (`contextBlocks`); cagent's
deliberate publications changed forge `freework/run-002`'s next attempt
and left its snapshots intact (Step 3). Unit tests 401/401.

## Deployment

- nctl before runtime work: Nautobot, worker, dumps and submodules ok;
  drift 47 converged, 1 drifting (the pre-existing agobserver drift).
- No open attempt; the store copied to
  `.local/backups/p14ex1-predeploy-20261018-233708` in the area.
- `wfe-release.sh deploy <area> ebfe8a5`: 401/401 in the release build,
  `current` `ed688f6` → `ebfe8a5`, service restarted, service and
  `bin/wfe` both report `ebfe8a5`. The store format did not change, so the
  code rollback is `activate … ed688f6…` alone; context content rolls back
  through the store's history (system rev 4, forge rev 6).

## The installed route (read-only, `http://agstudio.local:8093/wfe/`)

A read-only Playwright script (`deployed.ts` in the ignored
`workflow-editor-p14ex1/`) passed in Chromium and WebKit at 1400 px and
390 px:

- `#/home/context` opens no empty block; the Developer's Japanese
  instruction reads in its row with *Edit*; cagent's row is title,
  *cagent*, *Partly verified*, *Open*; no audit record, size or Remove in
  the list;
- cagent's panel: manager and scope, the overview, *View resource details*
  opening the cluster detail on the LAN route, *Evidence & history* with
  run-007 and the observations;
- forge: the repository part as one row, two inherited rows naming their
  scope, the forge overview keeping the ComfyUI/SwarmUI difference, the
  generation detail opening;
- the consumer run `freework/run-003`: *Used for this attempt* names the
  cagent blocks as delivered; a project block opens beside the run;
- no horizontal scroll at 390 px, no page errors.

## Before and after (same viewport, whole sections)

`before/` (installed `ed688f6`) and `after/` (installed `ebfe8a5`), both
Chromium, WebKit and 390 px:

- forge section, 1400 px: before, purpose textarea, four goal inputs, the
  guide list, the save row, the guide's box, and three cards with
  manager/editor/run/observation/check/size lines and grey bodies — about
  1,400 px tall above Runs and Workflows; after, five rows (purpose and
  goals, guide, Resources for Forge, Developer instructions with its text,
  cagent cluster summary) — about 570 px — with the same title size,
  icon tile, spacing, divider and right-hand action as the Workflows list
  next to them.
- Home section (808 → 256 px): before, the two cards plus an empty *New instruction*
  form; after, two rows.
- The panel at 1400 px leaves the list visible on the left; at 390 px it
  takes the width below the header.

## Commits and pushes

agdevworld `cfcc2e9`, `ebfe8a5` (pushed); pj-agdev pin `3ec5e68`
(pushed); pj-cagent `114208b`, `d98e6bd` (pushed to Gitea); devdocs
reports and README_DEV. Local notes: `pj-agdev/.local/devenv.md`
"workflow_editor p14 ex1 on this host".
