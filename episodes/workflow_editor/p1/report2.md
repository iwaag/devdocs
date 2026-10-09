# workflow_editor p1 — step 2 report: project read/write surface

## Outcome

The local service and the project editor screen are working against the real
Git fixture. Workspace registration and observation, project metadata editing,
repository inspection, local submodule addition, and workflow discovery and
creation all pass their tests (9 service tests, 21 browser checks). Total
`npm test`: 20/20.

## What was built

**Service** (`server/`, Node HTTP on `127.0.0.1:8095`):

- `registry.ts`: reads the ignored `registry.json`. Each workspace is a local
  registration (id, label, host, path). It is observed separately: whether the
  directory exists, is a Git repository, and has a readable `project.yaml`.
  No other hosts are contacted.
- `workspace.ts`: Git inspection per submodule, from `.gitmodules` (path, URL), `git
  ls-tree HEAD` (recorded gitlink), `git ls-files --stage` (staged gitlink if
  different), and inside the submodule: `rev-parse HEAD`, `symbolic-ref`
  (branch, or `null` for detached), `status --porcelain` (dirty count). It also covers
  initialization (own `.git` and own top level), project save, workflow discovery
  (`devdocs/workflows/*.y?ml`, identified by content `id`), creation
  (`<id>.yaml`, exclusive create, unique ID), and `git submodule add`.
- `files.ts`: every path is resolved inside the workspace root, lexically and
  through `realpath` (symlinks out are refused). Saves are temp-file + fsync +
  rename. An unchanged save does not touch the file.
- `api.ts`: JSON routes, `Host` must be `127.0.0.1`/`localhost`. Writes need
  `content-type: application/json` and, from a browser, an allowed `Origin`
  (the Vite dev origin or the service's own). Clients without `Origin` are
  local processes and may write.
- `watch.ts`: polling watcher with SSE, used by the project view to refresh after
  external changes (exercised fully in step 4).

**UI** (`src/`, Vite on `127.0.0.1:5175`, proxying `/api`): the project editor
follows `project_editor.jpg`. It has a dark top bar with the project name and a
workspace selector. The left column holds project id/name/intent/goals with explicit
save, and sub-repositories grouped Fixed / Study / Wedo / Other, with branch or
"detached HEAD", recorded hash, staged gitlink, "HEAD ≠ recorded",
"N uncommitted" and "not initialized". The right column holds workspaces and
workflows, with approval and validation chips, "Open Editor" and creation.
Access badges (Read-only / Editable) show only for the workflow selected in
"Access in workflow". Bindings that resolve to no repository are listed as
unresolved.

## Verified against actual Git state

Service tests (`test/workspace.test.ts`, each on a fresh seed in a temp dir):

- Recorded commit equals `git ls-tree HEAD <path>`, HEAD equals `git
  rev-parse HEAD` in the submodule, for every initialized submodule.
- Uninitialized: `assets/shared` in A has a recorded commit, no HEAD,
  `initialized: false`; in B it is initialized.
- Detached HEAD (`study/tools`) is `branch: null`; `study/agentic-patterns` is
  on `main`.
- Dirty checkout: a new file in `wedo/runtime` gives `dirty: 1`. A local commit
  in `study/tools` gives `matchesRecorded: false`.
- A malformed `project.yaml` is reported and is not overwritten by a save (409).
- `git submodule add ../study-evals.git study/evals` stages `A study/evals`
  and `M .gitmodules`. The repository shows staged, initialized, on `main`.
  Earlier dirty state is untouched.
- A failed add (`../no-such-repo.git`) returns Git's stderr and the partial state
  (.gitmodules entry, path, index, module dir). Git status is byte-identical
  before and after: nothing is reset.
- Workflow files are found by `id`, including a file whose name differs from its
  id. Duplicate and malformed IDs are refused.
- `..`, absolute paths and a symlink to the other workspace are refused.

Browser check (`checks/step2.ts`, 21 checks, all pass; screenshots in
`pj-agdev/.local/workflow-editor/screenshots/step2-*.png`): the
labels for uninitialized, detached, branch and dirty states, and editing and saving
name and goals (comments preserved in the file). Access badges for `onboarding`
(study/tools Read-only, wedo/runtime Editable, assets/shared not bound) and
unresolved bindings for `draft-gaps`. A failed add showing stderr and partial
state, then a successful add of `study/evals` (staged; `draft-gaps` errors drop
from 4 to 3 because its missing repository now resolves). Workflow creation
opening the editor route, and switching to workspace B and seeing its own
state. Also the unavailable registered workspace, and no unexpected console errors.

## Decisions and deviations

- Local file transport for `git submodule add` is enabled per command
  (`-c protocol.file.allow=always`) only when the location is a local path or
  `file://` URL. Git disables it by default for submodules since 2.38; no Git
  config file is changed.
- The project root is listed among the fixed repositories (as `.`), since
  workflows can bind it.
- The workspace list shows host "this machine" from the registry and the
  observed state only; it does not invent sync status for other hosts.
- Two first-run failures were in the check script, not the product. Playwright's
  `has-text` is a case-insensitive substring ("Saved" matched "Unsaved"), and
  `waitForURL` waits for a load event that hash routes never fire. The checks
  now wait on exact conditions.
