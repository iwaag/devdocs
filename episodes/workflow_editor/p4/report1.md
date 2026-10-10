# workflow_editor p4 — Stage 1: storage contract

Stage 1 of [plan.md](plan.md) is implemented in
`pj-agdev/experiments/workflow_editor` (commit `5ef7511`). All evidence below
is synthetic: temporary projects created by tests and by the browser check.
No person's workspace was touched.

## What changed

### Two devdocs storage modes

`project.yaml` is now `ag.project.v2`. It declares the storage mode in a new
field: `devdocs: directory | submodule`.

- `wfe create` and the service's creation default to `directory`.
- In `directory` mode, creation writes `devdocs/README.md` and
  `devdocs/workflows/` into the root repository. The result is one commit and
  no `.gitmodules`.
- `--devdocs submodule` keeps the earlier behaviour: a separate repository
  added as a submodule. In Stage 1 it still uses a local source; Stage 2
  replaces that with Gitea.

`server/devdocs.ts` is the single resolver. It reads the declared mode and the
Git structure, and names the repository that owns devdocs and the path inside
it: the root at `devdocs/...`, or the devdocs repository at its own root.

The resolver reports three problems and never converts anything:

- `devdocs-mode-mismatch`: the declaration disagrees with `.gitmodules`.
- `devdocs-mode-undeclared`: the mode is absent or not a known value.
- `runs-old-layout`: p3 run folders are still present.

The project view shows the mode read-only.

History references name a repository and a commit:

- `<commit>`: a commit of the owning repository.
- `root:<commit>`: a project root commit. In submodule mode the reader follows
  its recorded devdocs gitlink, and the display names both commits.

Historical reads never fall back to working-tree files.

In directory mode, devdocs is a project resource: a repository row of kind
`directory`, not an independent repository. A binding to `devdocs` is valid in
both modes.

### Run layout and record

| Item | p4 value |
| --- | --- |
| Runs live in | `devdocs/runs/<workflow-id>/<run-id>/` |
| Run record schema | `ag.workflow-run.v2` |
| Project identity | `run.json` carries `project` |
| Record naming another project | `location` problem |

Old formats are reported, never read as an empty listing:

- A v1 `run.json` shows as a `schema` problem.
- The p3 layout is not listed; a structure warning says it is there.

Updated to the new layout:

- path creation, listing and the watcher;
- artifact references and history reads;
- `wfe run` help and `wfe status`;
- the IDE authoring guide (`templates/AGENTS.md`), `START.md` and the setup
  prompts;
- `docs/contract.md`, `docs/runs.md` and `README.md`.

### Delegate nodes: definition and display only

Execution of delegate nodes is refused at three points:

1. **Preflight.** Run creation refuses a workflow with a delegate node before
   writing anything, with status 422 and code `delegate-unsupported`.
2. **Reducer.** The shared reducer refuses a `run.create` whose graph has a
   delegate node, so no other entrance can create such a run.
3. **Operations.** The `node.delegate` operation, the HTTP route and
   `wfe run delegate` are removed. Parent/child fields are gone from the
   record.

Display and validation:

- Validation keeps delegate definitions valid and adds the
  `delegate-not-executable` warning.
- The canvas card says "execution not supported".
- The inspector explains that runs are refused by the browser, the API and
  the CLI alike.

### Access as path scopes

`shared/access.ts` resolves access as path scopes. The most specific binding
containing a path governs it, and `.` contains everything. The run's own
folder is `report`: writable for its records even under `root: readonly`.
Other runs and workflow definitions are not included in that exception.

`wfe run access <run> [<path>]...` answers what the definition declares for a
path. These remain self-checks, and the docs say so.

## Validation

| Command | Result |
| --- | --- |
| `npm run check` (tsc, `node --test`, vite build) | 70 tests pass, 0 fail; type check and build clean |
| `node --test test/runs.test.ts` | 19 pass |
| `npm run build && node checks/runs.ts --repeat 10` | all browser checks pass, 0 page errors |

New or rewritten tests in `test/runs.test.ts`:

- **Creation in both modes.**
  - Checks the layout, `project`, `ag.workflow-run.v2` and the definition copy.
  - Checks authorship, and that no machine paths appear in the record.
- **History in both modes.**
  - Two commits each reconstruct the fixed workflow and the recorded stage
    (seq 2 and seq 6).
  - `root:<commit>` reaches the same stage, and in submodule mode names the
    followed gitlink.
  - Reports read at a commit, at HEAD and from the working tree differ as they
    should.
  - A file that exists only in the working tree is 404 at a commit.
- **Delegate refusal.**
  - Refused in preflight with nothing written, and by the reducer.
  - Refused in the CLI: create exits 1, and `run delegate` is now an unknown
    subcommand.
  - Refused over HTTP: `node.delegate` returns 400.
  - The definition stays valid, with the warning.
- **Visible problems, not empty listings.** Mode mismatch and undeclared mode
  are reported and block run creation. Malformed, tampered, old-schema and
  foreign-project records, and the old layout, are all shown as problems.
- **Path-scope access.** Covers a readonly root, an editable subpath, the
  run's own records, another run, a workflow file, and an undeclared path.
- **Unchanged parts.** The CLI, HTTP, watcher and operation-parity tests pass
  on the new layout.

The browser check `checks/runs.ts` now runs on a directory-mode project. It
adds the delegate card's "execution not supported" text and the refused run.
History is read from a root commit, and the banner names "project root".

| Measurement | Value |
| --- | --- |
| Progress saves shown | 10 of 10 within 2 s, 1 s poll interval |
| Malformed `run.json` shown | 0.9 s |
| Report file visible | 1.3 s |
| Change made during an outage, after restart | 1.0 s |

The record is in `pj-agdev/.local/workflow-editor-p4/checks-runs.json`
(ignored).

## Limitations and notes

- The submodule mode still creates its devdocs source as a local bare
  repository. That is the Stage 2 entry point: Gitea replaces it.
- p1 fixtures (`scripts/seed.ts`, the `checks/step*.ts` / `e2e.ts`) still seed
  a submodule-mode `demo` project with local bare sources. They run under the
  new schema because the example `project.yaml` declares `devdocs: submodule`.
  They are not part of p4's operation.
- The person's p3 authoring area `pj-agdev/.local/workflow-editor-p2/` keeps
  its p3 runs. Under the new contract those runs are reported as the old
  layout and as an old project schema; they are not converted. Stage 4
  registers the projects again.
- Stage 1 does not yet have run locking across processes. That is Stage 3:
  the plan puts it with execution.

## Next stage entry point

Stage 2 covers Gitea, the global resource registry, the agdev dashboard, and
the route from agdevworld. Inspection already done:

- **Gitea.** It runs at `agstudio.local:3000`.
  - autolab's `project_init.py` creates `autodev/*` repositories with the
    autolab-agent token.
  - agdevworld's relay (`agentroom/contexts.py`) has a urllib client that
    creates repositories under the `developer` token. It passes credentials
    to Git as a header through `GIT_CONFIG_*`, never in the URL.
- **agdevworld.** The relay is reached cross-origin at `localhost:8094` with
  open CORS. nginx on `:8090` serves only static files, and there is no
  same-origin proxy yet.
