# Workflow editor p7/ex1 — step 3: templates in workflow editing

Commit: agdevworld `e514bf6`.

## Changes

- **Add node.** The palette shows the basic types, then "Yours:" with one button per catalog template (tinted with its color), then *edit…* (to User nodes). An unavailable catalog shows a chip and a link, not an empty list.
- **Adding a template** uses the shared `instantiate()` on the draft:
  - the id is the template id, made unique;
  - type, name, description and appearance are copied;
  - `template: {id, revision, expectation}` is recorded.
  
  Bindings are chosen in the inspector among the workflow's declared bindings. The node is then edited like any other.
- **Inspector**: *Made from your template* shows the id, the revision and the expectation. It says when the catalog has moved on ("Your catalog has revision 4 now; this node keeps what it was made from") or when the template was deleted, and it states that provenance is outside approval and the executed graph. The run view's node detail shows the same provenance from the run's own definition revision.
- **YAML** (`server/yamlDoc.ts`): `template` is part of the node shape. A new one is written as a block mapping, and an untouched file still round-trips byte for byte. A malformed hand-written `template` is a shape problem. A malformed template id is the warning `template-id`. An unknown template is not an error.
- **Shared instantiation for agents and tools:**
  - `wfe nodes instantiate <template> --workflow <wf> | --file <path> [--run <run>] [--id] [--bind]… [--after]… [--before]…`. `--after` with `--before` inserts the node and removes the direct edge. `--run` takes the template from that run's snapshot (step 4).
  - `POST /api/workspaces/<ws>/workflows/<file>/instantiate`.
  
  Both write through the editor's round trip and save without committing.
- **Adjustments**: `template` is compared for non-pending nodes, so started work's provenance cannot be rewritten (`adjust-started`). Provenance alone is `adjust-nothing`, because the compiled graph does not change.
- **Node copy.** The editor has no node copy or duplicate operation. Model clones (drafts, revisions, snapshots) are deep copies and keep `template`, and run snapshots keep the YAML byte for byte.

## Verification

- `test/instantiate.test.ts` (6 tests):
  - instantiation copies content, makes ids unique, refuses a taken id, undeclared bindings and a delegate template, never sets `workflow` or `confirms`, and inserts between nodes;
  - YAML round trip;
  - intent and definition digests and the compiled graph equal those of the same node written by hand without provenance;
  - a description edit makes the approval stale, a provenance-only edit does not;
  - editing and deleting the catalog leaves the workflow file and an existing run's bundle and snapshot unchanged, and `wfe validate` still passes;
  - the CLI and HTTP produce byte-identical files, and HTTP refuses an undeclared binding with 422;
  - adjustments: provenance only is `adjust-nothing`; a completed node's provenance is `adjust-started`.
- `npm run check`: tsc, **202/202** tests (191 before ex1, +5 catalog, +6 instantiate), build.
- `checks/nodes.ts` (browser, private service on :8197), **20/20**:
  - dashboard → User nodes; seeded templates as card previews; the example offered, then added and edited (revision 2 on disk);
  - a concurrent edit makes the browser's save stale with the reason, and *Reload* shows it;
  - a new template with a live preview;
  - in the workflow editor, the templates are offered beside the basic types, and adding one gives an ordinary study node with its look;
  - the inspector shows template, revision and expectation; binding and description edits save as an ordinary node with provenance;
  - after a catalog edit the inspector says so and the file is unchanged;
  - an unreadable catalog is reported on both pages and left as it was;
  - no console errors.
- Screenshots: `agdevworld/.local/wfe-checks/screenshots/p7ex1-user-nodes.png`, `p7ex1-add-from-template.png` (ignored).

## Limitations

- In the browser a template node is added unconnected, like the basic types. Placement (`--after` / `--before`) is in the CLI and API, which agents use.
- `checks/appearance.ts` and `checks/nodes.ts` both build their own temporary areas. The live area's catalog is created at deployment (step 6).
