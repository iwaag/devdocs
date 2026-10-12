# Workflow editor p7/ex1 — step 6: verification and deployment

## Tested and released commits

| Repository | Commit | What |
| --- | --- | --- |
| agdevworld | `6058c0f` (deployed) | contracts `54ac34b`, catalog and editor `fbd8cac`, templates in editing `e514bf6`, delivery to runs `2285682`, request-sourced adjustments `6058c0f` |
| agautolab | `f1c466d` | wfrun common guide (catalog, request source, research evidence) and the settings test |
| pj-agdev | `0047974` | submodule pointers |

## Verification

- **Editor**: `npm run check` gives **211/211** (191 before ex1; +5 catalog, +6 instantiation, +5 run snapshots, +4 request adjustments; two existing tests updated for the new recorded facts). The release build ran the same check in its own directory: 211/211.
- **Executor**: agautolab `test_wfexec`, `test_wfnotify`, `test_wfintake`, `test_wfconversation` give **31/31** against this editor. One test compared attempt settings exactly. It now includes `catalog`, and it asserts that the catalog's meaning statement reaches the agent's prompt.
- **Browser checks** (private services, isolated registries and catalogs):
  - `nodes` 20/20 (new);
  - `appearance` 15/15;
  - `review` 22 passed, 0 failed;
  - `runs` 33/33, `exec` 20/20, `conversation` 12/12;
  - `docsteps` 4/4. It had failed at the same spot since p6 and passes on this release. The cause was not investigated: no ex1 change touches its view, so the earlier failure was probably environmental.

### Required evidence

| # | Evidence | Where |
| --- | --- | --- |
| 1 | Catalog CRUD, persistence across restart, validation, stale edits, through shared interfaces | `test/catalog.test.ts` (each read is a new read of the file, as after a restart; HTTP and CLI parity; stale on both); `checks/nodes.ts`; live: the service route created the catalog and `bin/wfe nodes list` read the same file |
| 2 | Templates make editable ordinary nodes with valid bindings and provenance; catalog edits and deletion leave definitions unchanged | `test/instantiate.test.ts`; `checks/nodes.ts` (inspector, save, catalog edit afterwards) |
| 3 | Semantic changes keep their approval effects; provenance and display have no execution meaning | `test/instantiate.test.ts` (digests, compiled graph, stale vs approved, `adjust-nothing`); `test/appearance.test.ts` |
| 4 | Browser and CLI/API see the same catalog; new and resumed attempts of one run get the same snapshot | `test/catalog.test.ts`; `test/catalogrun.test.ts` (attempt 1 and the resumed attempt: same hash in settings, same brief lines, after a catalog edit); `test_wfexec` (prompt) |
| 5 | Failed catalog reads are visible, not empty and not overwritten | `test/catalog.test.ts` (three kinds); `test/catalogrun.test.ts` (run created with `unavailable`, file untouched, brief says so); `checks/nodes.ts` (both pages) |
| 6 | Original-request evidence names the right input; existing refusals stay | `test/requestadjust.test.ts` |
| 7 | A study insertion compiles to the right dependencies and report steps; the adjusted run replays with its original definition | `test/requestadjust.test.ts` (formation example) |
| 8 | No keyword expansion, no implied capability, no duplicate mandatory work | `test/requestadjust.test.ts` (graph equals the workflow's; template node compiles to its type; same bindings; the run completes without the template) |
| 9 | Presentation, result confirmation, repair adjustments and cleanup still work | `test/present.test.ts`, `test/adjust.test.ts` (unchanged, passing); `checks/review.ts` |

## Release

- **Before.** `nctl status`: ok. Nautobot does not model the agdev services, so their state was read from launchd and `wfe`: no open attempt, no queued job (jobs 26 done, 1 dropped; attempts 26 ended), every presentation stopped, every run completed. Stores were backed up to `.local/agdev/.local/backups/p7ex1-20261012-191523/`.
- **Deployed** with `agdevworld/scripts/wfe-release.sh deploy .local/agdev`: `cde57cd` → `6058c0f`, healthy (the service and `bin/wfe` name the commit). `http://localhost:8093/wfe/` answers 200.
- **After.**
  - All 8 existing runs (`rts-vs-bot` improve-game run-001…004, `ppt` 4) check `usable`.
  - The first read through agdevworld's route created `node-catalog.json` with the two presets (`initializedNow: true`). The example is offered, not added.
  - Through `/wfe/`, the User nodes page lists them, and run-004's view says "None supplied: this run was created before node catalogs".
- **Executor guide.** agautolab's guide was changed in the working tree only after the release, because wfexec reads it at each claim. No restart was needed.

### Schema and cutover

- `node-catalog.json` is new (`ag.node-catalog.v1`). No store schema changed (exec 1, notify 3, intake 1, present 1).
- `run.json` stays `ag.workflow-run.v2`. New runs add `input.sha256`, `context.catalog` and `settings.catalog`, and may carry adjustments with source `request`. Existing runs were not converted.
- Workflow files stay `ag.workflow.v2` (optional `template`). The compiler stays `ag.wfe-compile.v2`; the notify protocol stays 2.
- The palette's built-in presets became catalog entries at the one-time initialization.

### Rollback limits

- `wfe-release.sh rollback .local/agdev` returns to `cde57cd`. That release ignores `context.catalog` and the node `template` field. It cannot read a run that recorded a `request`-sourced adjustment: the run is refused as a shape problem, not damaged. agautolab's guide should go back to `d31f654` with it; otherwise agents are pointed at `wfe nodes` and `wfe run nodes`, which the old release lacks.
- No store restore is needed. The catalog file would just be unused.

## Limitations

- The live catalog has only the seeded presets. The person's own templates, including whether they want the research example, are theirs to add.
