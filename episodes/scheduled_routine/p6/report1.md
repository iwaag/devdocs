# Step 1 — localtest workspace contract

Implemented and committed the repository-backed local-test convention for the
`study` pattern.

## Contract

`autolab project init-localtest <paper-id>` uses the existing
`autolab project init-repo` path and then initializes the clone with:

- `.gitignore`, including `.local/` for host facts, credentials, and large
  artifacts;
- `README.md`, naming the paper and its arXiv source;
- `localtest.yaml`, the persisted resume record; and
- `report.md`, a portable plan/result/evidence/handoff skeleton.

The supported `localtest.yaml` states are `prepared`, `waiting_external`,
`running`, `verified`, `failed`, `adoption_pending`, and `complete`. This is
small committed state in the experiment repository; no database or service
was added.

The standard demonstrated name is:

| project | paper ID | workspace folder | Gitea repository |
|---|---|---|---|
| `studyarxiv` | `2608.23283` | `localtest-2608.23283/` | `autodev/studyarxiv-localtest-2608.23283` |
| `studyarxiv` | `hep-th/9901001` | `localtest-hep-th-9901001/` | `autodev/studyarxiv-localtest-hep-th-9901001` |

Only the slash in an old-style arXiv ID changes to a hyphen; modern IDs keep
their dot.

## Evidence

- Added the convention and resume-state contract to Autolab's pattern guide.
- Added `init-localtest`, which derives the folder, calls the ordinary
  repository initializer, seeds the records only when absent, and commits and
  pushes them idempotently.
- Tests: `uv run pytest -q tests/test_cli.py tests/test_project_init.py` —
  `41 passed`.
- Autolab commit `6d31db1` was pushed to GitHub; the `pj-agdev` pin was
  committed and pushed as `08e57ec`.

Did this implementation work for Autolab — handoff candidate.
