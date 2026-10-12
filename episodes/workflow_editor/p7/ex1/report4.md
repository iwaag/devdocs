# Workflow editor p7/ex1 — step 4: the person's expectations reach run agents

Commits: agdevworld `fbd8cac` (snapshot at creation, input hash) and `2285682` (delivery and inspection). The agautolab guide text is prepared here and lands with the deployment in step 6. wfexec reads the guide from its working tree at every claim, so committing it before the editor release would hand agents commands the installed release lacks.

## Changes

- **Snapshot at creation.** `createRun` takes the operating area, and every entrance passes it: `wfe request` and the browser request (`ExecStore.request`), Zulip intake, and `wfe run create`.
  - It writes `<run>/context/node-catalog.json` (`ag.node-catalog-snapshot.v1`: generation, time, templates with content) and records `context.catalog`:
    - `{state: supplied, file, sha256, generation, templates}`;
    - an empty catalog is an explicit empty snapshot (`templates: 0`, file written);
    - an unreadable catalog is `{state: unavailable, problem}`, the run is still created, and the person's file is left alone.
  - The input file's `sha256` is recorded (for step 5).
- **Reading** checks the snapshot file against its hash, like the bundle. A changed or missing snapshot makes the record unusable (`bundle`). It is never re-read from the live catalog.
- **Attempts.** `claim` records `settings.catalog` (`{state, sha256, templates}` or `{state: unavailable}`) beside the contract and guides. The brief (`catalogBrief`) gives:
  - the plan's statement, verbatim;
  - one line per template: id, name, type, revision, expectation;
  - one line naming `wfe nodes instantiate <template> --run <run> --file <draft>` and `wfe run adjust`, with their help commands.
  
  Empty and unavailable catalogs each get one sentence saying so. A run without a catalog (created before ex1) gets nothing, so its brief does not claim a catalog.
- **Inspection.**
  - `wfe run nodes <run> [--show <template>]` prints the statement and each template in full, or why there is none.
  - `wfe run show` has a *Node catalog* line.
  - The run view has a *Node catalog (n templates, as given)* section: snapshot time, which attempts recorded receiving the same hash, and each template's name, id, type, revision and expectation.
  - `GET runs/<wf>/<run>` returns `catalog`.
- **Docs.** `docs/runs.md` covers the folder layout, the snapshot and attempts paragraph, and the operations table. `docs/catalog.md` came from step 1.
- **Common guide** (agautolab `agent/guides/wfrun/guide.md`, prepared, applied in step 6):
  - the head names `wfe run nodes` and `wfe nodes help`;
  - a new section, *The steps the person expects to see*, states as facts: the catalog was given at creation; it is an expectation, not a rule; whether and where is the agent's design choice; no duplicate of work the workflow already represents; integration is named in `plan.md`; the original request as an adjustment source, while scope changes still need agreement;
  - adjustments now list the original request among their sources;
  - under Documentation, requested research leaves traceable evidence (sources, contributions, choice and why, uncertainty), `plan.md` says where, a study node's report holds its own research, integrated research is in that node's report, and summaries link instead of restating.
  
  Both new paragraphs carry the `improve-game/run-004` reference.

## Verification

- `test/catalogrun.test.ts` (5 tests):
  - A request freezes the catalog. Attempt 1's brief has the statement, the template line and the tool line, and its settings record the snapshot.
  - After the person edits a template and deletes another, the resumed attempt's brief and settings are the same snapshot, and the file is byte-identical. A new run gets the edited catalog with a different hash.
  - A tampered snapshot makes the record unusable.
  - An empty catalog gets an explicit empty snapshot. An unreadable catalog gives `unavailable` with the parse problem, no file, and the person's file untouched. Each brief says which case it is.
  - A run created without an area has no catalog: nothing in its brief, `catalog: none`, and `wfe run nodes` says so.
  - `wfe run nodes` shows the snapshot's text, not the live one. `wfe nodes instantiate --run --file` takes the snapshot's description.
- `test/settings.test.ts` updated: attempt settings now include `catalog`.
- `npm run check`: **207/207**, tsc and build clean.

## Notes

- The brief lists expectations only. Descriptions are a `wfe run nodes` call away. That keeps the brief short as the catalog grows, and the description is what an instantiated node carries anyway.
- Workflow-authoring tools outside runs (an IDE agent editing `devdocs/workflows`) use `wfe nodes list|show|instantiate --workflow`, the same operations as the browser.
