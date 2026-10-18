# p14 Step 5 — deploy, verify, hand off

## Release checks

- `npm run check` in the release build: tsc, 401/401 tests, build (both
  releases). One timing test (`sqlite` lock bound) failed once in a
  loaded dev run and passed in isolation and in the release build.
- Browser checks (fixtures, Chromium and WebKit, 1400 px and 390 px):
  `contextBlocks` 29/29 (32 narrow), `contextEditors` 63/63 (67 narrow),
  `context` 25/25.
- autolab `tests/test_wfexec.py` 21/21.

## Data

- Backups before the first change, before the deploy and before the
  redeploy (the area's ignored backups folder).
- Conversion verified on a copy of the live store first, then live on the
  first read: the Developer's instruction (in Japanese, author iwaag)
  became the system block `developer` with all four revisions; the two
  cleared agops overrides became removed blocks with history; the p13
  files moved to `contexts/archive/p13/`.
- Historical snapshots: the five p13 snapshots (`ag.wfe-context.v1`)
  read unchanged; agops `context-check/run-002` still shows its attempt as
  prepared and delivered on the installed release.

## Deployment

nctl status/drift checked before runtime work (47 converged, the
pre-existing agobserver drift). Deployed with `wfe-release.sh deploy`:
`de014de` (before Step 4's trials), then `ed688f6` (freshness shows the
newest observation; longer bodies stay open — found while inspecting the
deployed forge page). The service and `bin/wfe` report the release; the
normal LAN route serves it. The executor and managed definitions needed no
restart (guides and context are read per claim).

## Verified on the installed release (LAN route, read-only script; Chromium and WebKit, desktop and 390 px: all pass)

- Home: separate Developer and cagent cards, kinds, managers, last
  editors, freshness;
- forge: cagent's resource card (24 checkpoints), inherited system cards
  naming their scope, the detail link opening on the LAN route;
- the consumer run: *Used for this attempt* names the cagent blocks and
  says delivered;
- a p13 run: its historical snapshot is readable.

Writes on the live service: cagent's own publications (Step 4), one
cross-host trial block written from agautolab1 with `curl` through the LAN
service and removed again (history kept), and a `WFE_SERVICE` CLI read.

## Left in place / removed

Kept: `developer` (system), `cagent-cluster` (system), `cagent-resources`
(forge). Removed: the cross-host trial block. The cagent context runs and
the forge consumer run are records.

## Commits

Committed locally: agdevworld (`d7dbcf9` … `ed688f6`), agautolab
`60ed2fb`, pj-agdev pin `55a87b9`, devdocs (reports, README_DEV).
**Pushing these was refused by the session's permission classifier**;
they are ahead of origin. pj-cagent (`8e478fe`, `8d33bb9`, `7f6a721`)
was pushed to its Gitea.
