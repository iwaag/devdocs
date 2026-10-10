# workflow_editor p2/pre1 step 2 — create/register and the CLI

## What was built

In `pj-agdev/experiments/workflow_editor/`:

| Piece | Where |
| --- | --- |
| Project creation (shared by browser and CLI) | `server/create.ts` (`createProject`) |
| Registration, registry states, structure diagnostics | `server/registry.ts` (`registerWorkspace`, `loadRegistry`, `workspaceAt`), `Workspace.structure()` |
| Service routes | `POST /api/projects`, `POST /api/workspaces`; `GET /api/workspaces` now also returns the registry path/existence and the authoring area |
| Browser setup actions | `src/homeView.ts`: the home page `#/` lists projects and always shows "Create project" and "Register workspace" |
| Auto-arrange shared with the CLI | `src/layout.ts` moved to `shared/layout.ts`, with a new `autoArrange()` used by both the button and `wfe arrange` |
| CLI | `cli/wfe.ts` (`npm run wfe`, `bin: wfe`) |
| Tests and checks | `test/setup.test.ts` (13 tests), `checks/setup.ts` (15 browser checks), `checks/bench/service.ts` (private service for checks) |

### Creation

`createProject` does the following:

1. Checks the inputs: id pattern, name, and the person's Git identity. A
   missing `user.name`/`user.email` stops it before it writes anything.
2. Refuses a non-empty destination that is not an unfinished creation.
3. Writes the marker `.local/wfe-create.json`.
4. Creates the local devdocs source `<area>/sources/<id>-devdocs.git` with an
   initial commit (`README.md`, `workflows/.gitkeep`). An explicitly given
   source is used unchanged instead.
5. Runs `git init` for the root. Writes `project.yaml` (block-style goals,
   block intent when multiline) and `.gitignore` (`.local/`).
6. Runs `git submodule add ../sources/<id>-devdocs.git devdocs`. The URL is
   relative to the root. The root has no remote, so Git resolves it against
   the work tree, and the area can be moved as a whole (checked by hand).
7. Commits only the files it created, as "Create project <id>".
8. Registers the workspace and removes the marker.

Each step reports `done`, `kept` (it already existed) or `failed`, and the
commits are listed. A failure leaves the marker. The result names the
finished steps and says nothing was rolled back. Running it again without
resume is refused. With resume it skips what exists and never rewrites or
deletes. Files the person added meanwhile are kept and not committed for them
(tested).

### Registration

`registerWorkspace` records `git rev-parse --show-toplevel` (realpath) of the
given directory. If that root is already registered under any id, the result
is `already-registered` and nothing changes. A new entry is appended to the
raw registry JSON, so other entries and unknown fields stay. An id clash or a
non-Git directory is refused. Diagnostics cover `project.yaml`
(missing/unreadable/validation), `.gitignore` without `.local/`, devdocs that
is missing or uninitialized, and a missing `devdocs/workflows/`, each with a
fix. The project view shows the same diagnostics.

A **missing** registry is `{workspaces: [], exists: false}`: the home page
shows "No project is registered yet" and both actions. A **malformed** one
(bad JSON, wrong shape, an entry without id/path, duplicate ids) raises an
error that names the file. The home page shows it with the actions still
visible, and nothing writes over it.

### Boundaries

The browser's `POST /api/projects` resolves the destination with the
existing `inside(area, …)` check. A destination such as `../escape` gets
HTTP 400, and nothing is created (tested). Browser registration writes only
the registry. The CLI is the caller's own process and takes any destination.
That difference is deliberate and documented in the README and in
`wfe help create`. The existing host, origin and content-type guards are
unchanged.

## The CLI

`wfe help` is a one-screen index. It also prints the registry in use and the
path of `docs/contract.md`. `wfe <command> --help` says what the command
reads or changes, its inputs, its result and its exit codes. Commands:
`create`, `register`, `list`, `status`, `add-repo`, `workflow new`,
`validate`, `approve`, `arrange`, `serve`.

- **No command needs the running service.** They call the same modules
  directly. `wfe status` reports whether a service answers on the port. It
  also detects one serving a different registry (the p1 service on 8095 does
  that) and says how to start the right one. `wfe serve` builds the UI and
  starts the service for the CLI's registry and area.
- The workspace is the registered one that contains the current directory,
  or `--workspace <id>`. Workflows are named by file name or id.
- `--json` gives structured output. Exit codes are 0 for success, 1 for
  refused or errors found, 2 for usage.
- `approve` requires `--approver`. The registry's default approver is not
  used, and passing validation never implies approval. It calls
  `Workspace.approve`, the code behind the UI buttons, and the digests match
  (tested).
- `arrange` applies `autoArrange` and saves through `Workspace.saveWorkflow`.
  The positions equal the button's, and approvals stay approved (tested).
- `validate` uses `Workspace.workflowResponse`, so the issues are the
  editor's.

## Operation matrix — actual entry points

| Operation | Person | IDE agent |
| --- | --- | --- |
| Create project | Home → "Create project" | `wfe create <dir> --name … [--intent …] [--goal …]` |
| Register workspace | Home → "Register workspace" | `wfe register [<dir>]` |
| List / inspect workspaces and Git state | Home; project view (repositories, workspaces, diagnostics) | `wfe list`, `wfe status` |
| Edit intent and goals | Project view, or VS Code on `project.yaml` | Edit `project.yaml` |
| Create a workflow | Project view → "New workflow" | `wfe workflow new <id>`, or write `devdocs/workflows/<id>.yaml` |
| Edit graph and descriptions | Workflow editor, or VS Code | Edit the workflow YAML (`docs/contract.md`) |
| Add submodule | Project view → "Add submodule", or `git submodule add` | `wfe add-repo <path> <location>`, or Git |
| Validation | Workflow editor status line and inspector; project view counts | `wfe validate [<workflow>]` |
| Approve intent / definition | Workflow editor → "Approve intent/definition" with the declared approver | `wfe approve <wf> intent\|definition --approver <name>` |
| Auto-arrange | Workflow editor → "Auto-arrange", then Save | `wfe arrange <wf>` |
| Rename/delete definitions, repair malformed YAML | VS Code | File editing |
| Diffs, commits, history | VS Code / Git | Git |
| Start the editor | `wfe serve` | `wfe serve` |

Both parties may use the CLI. Remaining asymmetries:

- Auto-arrange in the browser changes the draft, which is then saved. The CLI
  writes the file directly, because the CLI has no drafts. The result is
  identical.
- The browser creates projects only inside the authoring area. The CLI also
  works outside it. The person can use the CLI too.
- The person also has the editor's live view and its unsaved-draft guard. The
  agent reads the files and `wfe status`. That is a difference of medium, not
  of capability.

No operation is browser-only any more. Auto-arrange was the only one.

## Verification

- `npm run check`: tsc clean, 44 tests (31 p1 + 13 new), build.
- `checks/setup.ts`: 15/15 on its own temporary area and service (port 8196).
- The p1 browser checks step2, step3, step4 and e2e pass unchanged. They ran
  against a private seed and service: `WFE_FIXTURE`/`WFE_URL` are new in
  `checks/lib.ts`, and e2e now reseeds `FIXTURE`. So the p1 fixture and
  the service already running on 8095/5175 were not touched.
- A CLI trial from an empty scratch area: `create` → `status` → refused
  re-create. It produced the expected files, two commits and a real devdocs
  submodule.

The p1 root page used to redirect to the first available workspace. It now
shows the projects list. The workflow and project views link back to it
("Projects").
