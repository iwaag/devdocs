# improve_wfnode p1 step 1: definition contract and storage

Date: 2026-11-03. Plan: [plan.md](plan.md). Code: agdevworld `workflow_editor/`, this step's commit is listed under *Revision*.

## What changed

- `shared/model.ts`: `WorkflowNode.example`, `.expectations`, `.specializes`; `Workflow.groups` (`name`, `kind`,
  `cardinality`, `description`, ordered `members`, optional display color). A field of the wrong shape is a shape
  issue, which keeps the file read-only in the editor rather than dropping it.
- `shared/expectations.ts` (new): strengths, the six documented keys and their meanings.
- `shared/validate.ts`: group errors (dangling, duplicate and overlapping members, generated steps as members, kind,
  cardinality, name), expectation errors (key, duplicate, strength), a warning for an unknown key, malformed provenance.
- `shared/canonical.ts`: the additions join the definition digest only when declared, so an existing definition keeps its digest.
  Member order counts. Expectation order, layout and group color do not. Omitted strength digests as `preferred`.
- `server/yamlDoc.ts`: the fields round-trip. Comments and unknown fields inside nodes and groups survive an edit.
- `server/runs.ts`: known node fields and revision differences. `AuthoredDiff.groups` reports added, removed, changed
  and reordered groups; `example`, `expectations` and `specializes` are reported as node changes.
- `docs/contract.md`: *Groups*, *Examples*, *Expectations* and the validation table.

## Design choices the plan left open

- Provenance is a node field: `specializes: {examples: [...], revision: rN}`. It is part of the digest, because it says what the work is.
- The group's display color is `appearance: {color}`, the only styling, outside the digest.
- Duplicate expectation keys are errors: one entry states the strength and note for a key.

## Checks that ran

- New `test/nodeExpectations.test.ts`: 7 of 7 passed (reading, legacy meaning, round trip, empty list, digest, errors, shape issues).
- `compile`, `contract`, `drafts`, `adjust` suites: 48 passed, 0 failed. `tsc` clean.

## Not yet covered

- `compile.ts` does not yet carry the fields into a run's frozen graph (step 2), so this step alone gives them no run meaning.
- Draft, trial and snapshot preservation rest on those paths copying file bytes. It is exercised end to end in step 2 and step 5, not here.
- Nothing is deployed yet.

## Revision

agdevworld: the commit "node expectations, example status and explicit list groups in the definition contract".
