# p5 step 1 — contract and compiler

Plan section: [plan.md](plan.md) §2, §3, §10.1.

## What was done

agdevworld `1d3a718` (`workflow_editor/`).

| Area | Change |
| --- | --- |
| Schema | `ag.workflow.v2`. `documentation: {policy: default \| manual}` is required. |
| Missing or wrong policy | Validation error `documentation-policy`. Nothing is defaulted silently, so a run cannot pick up rules its author never declared. |
| v1 files | Reported as a `schema` error naming the fix (schema v2 + `documentation.policy`). They still load in the editor so the fix can be made there. |
| New workflows | Created with `policy: default` (`emptyWorkflow`, `newWorkflowText`). |
| YAML | The documentation mapping round-trips like other fields. A file without it is written back unchanged until the policy is set. |
| Approval | The policy is in the definition projection, now kind `ag.workflow.v2/definition`. Changing the policy makes definition approval stale. Intent approval is unaffected (intent projection unchanged). Layout stays outside both. |
| Compiler | `shared/compile.ts`, version `ag.wfe-compile.v1`. Details below. |
| Docs | `docs/contract.md`: schema v2, the "Documentation policy" section, validation table, digest projection. README layout table. |

### The compiler

One compiler serves the editor preview and run creation.

- **Default policy.**
  - Adds `_plan` (a talk) before every authored entry node.
  - Adds `_report.<node>` after every authored node, and rewrites each `A → B` into `A → _report.A → B`.
  - Adds `_final-report` after the reports of all authored terminal nodes.
  - Generated steps get no reports of their own.
- **Manual policy.** The effective graph is the authored graph, with duplicate edges collapsed.
- **Generated step identities.** Generated ids start with `_`, which `ID_PATTERN` forbids for authored ids, so the two cannot collide. Each generated step carries a `StepSpec`:
  - `kind`
  - `for`: the authored node it reports on
  - `output`: `plan.md`, `report-<node>.md` or `report.md`
  - `purpose`
  - `instructions`
- **What is frozen.** The result (`EffectiveGraph`: compiler version, policy, nodes, edges) is the object a run will freeze. The instructions are part of it, so a later guide or compiler change cannot rewrite what an existing run was asked to do.
- **Refusals.** The compiler refuses graphs that are not executable:
  - no nodes, under either policy
  - broken or self edges
  - cycles
  - malformed ids
  - a missing or unknown policy

  Delegates still compile for the preview. Run creation keeps refusing them.

### Existing fixtures

Every test and check fixture moved to v2. The run and exec tests describe authored-only execution, so their fixtures declare `policy: manual`, which keeps their meaning. The `onboarding` example declares `default`, so the editor preview has a default-policy sample.

## Verification

- `npm run check` (tsc, 105 tests, vite build): passed.
  - The 93 existing tests are unchanged in substance.
  - The 12 new tests are in `test/compile.test.ts`. They cover:
    - linear expansion (exact node set and edges)
    - branch and join gating
    - several entries and terminals
    - no report-of-report (2n + 2 steps)
    - no id collision with look-alike authored ids
    - manual = authored
    - refusals
    - determinism
    - policy validation and the v1 hint
    - definition-only staleness
    - YAML round trip
    - split counts

## Limitations / not yet done

- **Snapshot and evidence not wired.** Freezing the effective graph in `run.json` and the document-evidence fields are designed but not connected. Both land in step 2 with the reducer: the compiler's output type is the snapshot format, and the evidence is checked by the file layer, not the reducer.
- **Active workflows still on v1.** The RTS workflow in its workspace is still v1. The installed release (`6fe2b61`) is unaffected until deployment; the definition is updated deliberately in step 5.
- **Browser checks not rerun.** The `checks/*.ts` browser checks were only type-checked here.

## Next entry point

Step 2:

- add `definition.effective` to run creation;
- make the reducer use the frozen graph;
- add document evidence and plan agreement to the reducer and the file layer;
- update CLI/API, briefs, and autolab's guide.
