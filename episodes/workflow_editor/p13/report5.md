# p13 Step 5 — verify, deploy and demonstrate

## Tests

- WFE `npm test` 395/395 in the release build (incl. `test/context.test.ts`:
  scope isolation, source selection, guide deduplication, removal and a
  cleared override's listing, concurrent edits, snapshot consistency while
  an override changes during an open attempt, missing snapshots, unreadable
  sources, delegation scope, session comparison; `test/setup.test.ts`: the
  installed authoring entry).
- autolab `tests/test_wfexec.py` + `test_wfconversation.py` 22/22: context
  after the contract once, recorded identities and delivery, fresh/resume/
  reconstruction prompts, a changed source replacing a `run` session.
- Browser `checks/context.ts` 23/23 in Chromium and WebKit (Step 3 list plus:
  a source absent before and after is not marked).
- Publication: the trial runs' branches (pushed checkpoints, integration)
  contain none of the override wording; run.json holds snapshot ids,
  fingerprints and hashes only. Overrides and snapshots live in the area's
  ignored `.local/contexts/`.

## Deployment

- Cluster state checked first: `nctl drift --host agstudio` converged.
  No attempt open, nothing queued. Stores and area settings backed up
  (`.local/backups/p13-predeploy-*`, host notes in the ignored devenv).
- WFE releases through `scripts/wfe-release.sh deploy`: `6bb8069`, then
  fixes `c3a9de1` (a cleared workflow override stays listed) and `40b3903`
  (a removed source shown once in the run panel). Each: release check 395/395,
  service and launcher healthy, authoring entry refreshed
  (`<area>/authoring/AGENTS.md`, `CLAUDE.md`).
- Executor: autolab `e66e1dd` (context in the prompt, delivery record) and
  `4c18502` (wfrun contract mentions the context section);
  `com.agdev.agautolab-wfexec` restarted.
- Managed definitions: `project.yaml guides: [agent/guide.md]` added and
  `agent/guide.md` removed from every workflow's `execution.guides` in
  agsfx, archsage, cagent, forge, jevlab, playprey and ppt (all valid,
  pushed). agops (`recover` only) and rts-vs-bot (cross-project guides of one
  workflow) keep workflow-scoped guides. `wfe context show --for execution`
  on cagent/change: one project guide, one workflow guide.
- pj-agdev pins agdevworld `40b3903`, agautolab `4c18502`.

## Demonstration (deployed service, agops `context-check`)

A disposable workflow (write `note.md`, then confirm with the operator;
session policy `run`, profile sonnet). Overrides set through `wfe context`:

| Scope | Override |
| --- | --- |
| system | "Unless the request says otherwise, write every produced text (files, notes, reports) in English." |
| project agops | "agops trial notes start with the line `Project: agops`." |
| workflow context-check | "A context-check note ends with the line `Checked by context-check`." |

| Run | Request | Actual `note.md` |
| --- | --- | --- |
| run-001 | English: two sentences on why a home lab keeps backups | English; first line `Project: agops`; last line `Checked by context-check` |
| run-002 | Japanese, and "このメモは日本語で書いてください" | Japanese (the explicit request refined the general English preference); same first and last lines |

Change between attempts (run-002, waiting on its confirmation):

1. The workflow override was changed to end with
   `Checked by context-check (v2)`; then the operator asked for a rewrite.
2. Attempt 2 got a new snapshot; its session was replaced with the recorded
   reason `configuration changed: context workflow/override changed`.
3. While attempt 2 was running the workflow override was cleared. Attempt 2
   kept its snapshot: the rewritten note ends with `(v2)`.
4. `wfe context run` then showed attempt 1 (revision 1), attempt 2
   (revision 2), and the next attempt without the override, with "Differs:
   workflow/override removed" and the session that would replace it.
5. run-001's next attempt (after the clear) recorded the override as absent;
   both runs completed after the operator's confirmation. All attempts
   recorded `attempt.context delivered`.

After a `com.agdev.wfe-service` restart the overrides, their revisions and
every snapshot were read back unchanged through the LAN editor route (HTTP
200); the deployed Context view and the run panel rendered with no page
error, and with no horizontal page scroll at 390 px.

Cleanup: the system and project trial overrides were cleared (revisions
kept); the workflow override was already cleared in the trial. No user
preference existed before. Evidence (ignored): `pj-agdev/.local/
workflow-editor-p13/step5/` (requests, notes, `wfe context run` outputs,
deployed screenshots) and deploy logs beside it.
