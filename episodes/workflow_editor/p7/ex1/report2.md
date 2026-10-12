# Workflow editor p7/ex1 — step 2: the user-node catalog and its editor

Commit: agdevworld `fbd8cac`.

## Changes

- **Shared model** (`shared/catalog.ts`): the `ag.node-catalog.v1` shape, content validation, seeds and examples, the run snapshot shape, and the pure `instantiate()` (used in step 3).
  - Types are `study`, `do`, `talk`. A `delegate` template is refused with a message saying why.
  - The seeds are the p7 presets with their existing descriptions and appearance, plus an expectation each.
  - The example is *Preliminary research*: a study whose description asks for sources, findings, the choice and its alternatives, and uncertainties. Its expectation is a study node before the work it informs.
- **Persistence** (`server/catalog.ts`): `<area>/node-catalog.json`.
  - `readCatalog()` initializes a missing file exclusively, with the seeds and `initialized: {by, at, seeded}`. A malformed file, another schema or invalid content reads as `unavailable` with the problem.
  - Writes go through `change()`: under the catalog's cross-process lock, read, apply, validate the result, atomic replace, `generation + 1`.
  - Refusals:
    - `template-stale`: update or delete based on an old revision;
    - `template-exists` / `template-retired`: an existing id, or a deleted id (ids are never reused);
    - `template-invalid`, `template-id`, `template-by`: bad content, a bad id, no declared author;
    - `catalog-unavailable`: any write to an unreadable file.
  - A save with unchanged content keeps the revision.
- **Interfaces** (all on the same operations):
  - HTTP: `GET/POST /api/nodes`, `PUT/DELETE /api/nodes/<id>`, `POST /api/nodes/examples/<id>`.
  - CLI: `wfe nodes list|show|create|update|delete|example|instantiate`. `wfe nodes --help` explains each field, revisions, stale edits and instantiation. The author defaults to the registry's approver or `--by`.
  - The area is the explicit registry's directory, as for every other area file. There is no implicit catalog.
- **Browser**: the dashboard's header has *User nodes* (`#/nodes`).
  - The page has a short explanation, the person's name (remembered per browser), and the templates as card previews with id, revision, who/when and the expectation.
  - The editor has name, type, description, expectation, icon and color, with a live preview and per-field explanations. The id is asked only for a new template.
  - Save is refused when stale, with a *Reload it* action. Delete asks first.
  - Examples are listed separately as *Add "…"*.
  - An unavailable catalog shows the problem and offers no editing.
- **Presets moved.** `NODE_PRESETS` is gone from `shared/model.ts`. The workflow editor's palette offers catalog templates (step 3). `checks/appearance.ts` now uses the seeded templates.
- **Run creation** already freezes the snapshot and the input hash. The step 4 plumbing was needed for `--run` instantiation; step 4 reports it.

## Verification

- `test/catalog.test.ts` (5 tests):
  - Initialization happens once; deleted seeds stay deleted on later reads.
  - Revision rules: a no-op save keeps the revision, an edit bumps it, stale update and delete are refused, retired ids are refused, and so are delegate, missing expectation, unknown icon and a bad id. `generation` counts only real writes.
  - Three kinds of unreadable file (bad JSON, other schema, invalid template) are `unavailable`, refuse writes and are left byte for byte.
  - The browser route and `wfe nodes` see each other's writes. A stale edit is refused on both. The example is offered, not added. An anonymous write is refused.
  - The example becomes the person's template.
- `checks/appearance.ts` 15/15 with templates in the palette.
- Browser check `checks/nodes.ts` (step 3 report): the catalog page flows, including a stale edit and an unreadable file.
- During the browser check, editing a field rebuilt the whole form, so a click into the next field landed on a replaced input. Fixed: typing now updates only the preview, problems and buttons.

## Limitations

- The page does not follow catalog changes live. *Refresh*, the stale check and *Reload it* cover concurrent edits.
- The name field is a declaration (like approvers), not authentication.
