# workflow_editor p1 — step 1 report: contract and fixtures

## Outcome

The file contract, the shared model, validation, canonicalization, the YAML
round-trip layer and a repeatable fixture seed are in place under
`pj-agdev/experiments/workflow_editor/`. All 11 contract tests pass and the
type check is clean.

## Decisions made in this step

- **Service:** Node's own HTTP server, run directly from TypeScript (Node 26 strips
  types natively), so the service needs no build step and no framework. Git runs
  through `execFile` with argument arrays.
- **YAML parser:** `yaml` 2.9 (eemeli/yaml). Its Document API keeps comments, key
  order and styles, so edits are applied onto the parsed document instead of
  regenerating the file. No text patching.
- **Tests:** `node --test` on the TypeScript sources; no test framework.
- **Browser checks:** `playwright-core` 1.62.1 with its cached Chromium, as a dev
  dependency of the experiment only.
- **Digests** use Web Crypto SHA-256, which the browser and Node both have, so
  the UI and the service compute the same digest.

## Contract (summary; full text in `docs/contract.md`)

- `project.yaml` (`ag.project.v1`): schema, id, name, intent, goals. Nothing else:
  repositories come from `.gitmodules` and Git.
- `devdocs/workflows/*.yaml` (`ag.workflow.v1`): schema, id, name, intent,
  `repositories` (key → path + readonly/editable), `nodes` (id → type, name,
  description, binding keys, delegate `workflow`), `edges` (`{from, to}`),
  `approvals` (intent / definition: digest, approver, at), `layout.nodes`
  (id → x, y). Node `name` is optional; it is the short card label.
- IDs match `^[a-z0-9][a-z0-9_.-]{0,63}$`; delegates resolve targets by workflow ID.
- Edges mean "target waits for source"; several outgoing edges branch, several
  incoming edges join (wait for all); graphs must be acyclic.
- **YAML subset:** one document; plain text keys; no anchors, aliases, explicit
  tags, merge keys or duplicate keys. Wrongly shaped fields are rejected too.
  Rejected files are shown read-only and never rewritten.
- **Preservation:** unknown fields, comments, key order and styles of untouched
  entries survive; an unchanged document round-trips byte for byte.
- **Digests:** `sha256:` over canonical JSON (sorted keys, no whitespace) of an
  intent projection and a definition projection. Text is normalized (line
  endings, trailing spaces, surrounding blank lines). Workflow id/name,
  approvals, layout, comments and formatting are excluded.
- **Save semantics:** explicit save, temp file + rename, no commit; external
  changes reload automatically only without an unsaved draft; malformed files
  block saving over them.

## Fixture

`npm run seed -- --reset` builds `pj-agdev/.local/workflow-editor/` (ignored):

- `sources/`: bare repositories `pj-demo`, `devdocs`, `study-agentic-patterns`,
  `study-tools`, `study-evals` (not yet a submodule; for step 2), `wedo-runtime`,
  `wedo-pipeline`, `shared-assets`.
- `pj-demo` records submodules `devdocs`, `study/agentic-patterns`, `study/tools`,
  `wedo/runtime`, `wedo/pipeline`, `assets/shared` with relative URLs
  (`../<name>.git`).
- `workspaces/a` and `workspaces/b` are clones of `pj-demo`. A leaves
  `assets/shared` uninitialized and puts `study/agentic-patterns` on branch
  `main`; other submodules are at the recorded commit with a detached HEAD.
- `registry.json` registers A, B and one workspace whose directory does not
  exist (to show registration without availability).
- Commits use a fixed identity and date: a reseed reproduces the same root
  commit (checked). The seed refuses to delete a directory without its marker
  file.

Synthetic workflows (in `examples/project/workflows/`, copied into devdocs):

- `onboarding`: all four node types, a parallel branch (`survey` → `provision`
  and `review`), an all-predecessor join (`integrate`), a delegate to
  `repo-setup` with a parallel `talk` beside it, readonly and editable bindings
  including `devdocs`. No layout, so it is laid out automatically.
- `repo-setup`: the delegate target, with stored positions.
- `draft-gaps`: a saved draft with a missing repository, an undeclared binding,
  an unknown delegate target and an edge to a missing node.

`examples/invalid/` holds malformed, anchored, cyclic, mutually recursive and
wrongly shaped files for the tests.

## Tests (11, all passing)

Example validity and node-type/branch/join coverage; each broken reference in
`draft-gaps`; rejection of malformed, anchors, tags, merge keys, several
documents, complex keys and duplicate keys; cycles, self and recursive
delegation; duplicate IDs, bad paths, types and access; canonical JSON; digests
unaffected by reordering, layout, approvals, name, whitespace and YAML style;
stale-state rules (description, edge, access, delegate target → definition
stale; intent → both; layout → neither); byte-for-byte round trip; edits that
keep comments, unknown fields and flow style; new files reading back as the
same model.

## Deviations and notes

- The `yaml` library normalizes the spacing before a trailing comment to one
  space when it re-serializes a document. The examples use one space, so they
  round-trip exactly. This is documented as a known normalization rather than
  worked around.
- A self-edge is reported as `edge-self`, not also as a cycle.
- Definition approval also covers node `name` (the card label), because it is
  part of what a reviewer reads.
