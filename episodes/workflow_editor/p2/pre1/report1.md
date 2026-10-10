# workflow_editor p2/pre1 step 1 — baseline and capability audit

## What was done

- Ran the p1 checks unchanged: `npm run check` passes (tsc, 31 tests, build).
- Added a measurement harness that never touches another registry or
  workspace: `checks/measure.ts` builds its own repositories and registry in a
  temporary directory, starts its own service from the production build on
  port 8195 (`--serve-dist`), and drives the UI with playwright-core.
  `checks/bench/counters.ts` is preloaded into that service (`node --import`)
  and counts Git subprocesses, file reads and SSE events, and reports process
  CPU over IPC. The service code is unchanged.
- The p1 service and Vite dev server that were already running on
  8095/5175 (p1 fixture registry) were left alone. No p1 browser check was run,
  because those reseed the shared p1 fixture under that running service.

Startup reproduced on isolated data: `npm run build`, then
`node server/main.ts --serve-dist --port 8195 --registry <bench registry>`;
the UI opened a workflow, followed sequential external saves, and a UI save
wrote the YAML that the next external edit then changed (probes 1–2 below).

## Baseline measurements (p1 code, 1000 ms polling)

Data built by the harness:

| Case | Submodules | Workflow files | Notes |
| --- | --- | --- | --- |
| Representative | 3 (devdocs + 2, one uninitialized) | 1 | the intended p2 task |
| Large synthetic | 20 (devdocs + 19, one uninitialized) | 150 | exposes repeated full refreshes |

20 sequential edits of the open workflow (alternating in-place writes and
atomic-rename saves), 10 of `project.yaml` in the project view; 0.3–1.5 s
random pause after each observed update, to sample the polling phase.
Latency = file write/rename completed → value visible in the DOM (20 ms page
polling).

| View | p50 | p90 | max | > 2 s |
| --- | --- | --- | --- | --- |
| Workflow, representative | 671 ms | 1047 ms | 1219 ms | 0/20 |
| Project, representative | 927 ms | 1319 ms | 1319 ms | 0/10 |
| Workflow, large | 1030 ms | 1298 ms | 1318 ms | 0/20 |
| Project, large | 964 ms | 1384 ms | 1384 ms | 0/10 |

Load (service process):

| Situation | Representative | Large |
| --- | --- | --- |
| No viewer connected, 10 s | 0 Git, 0 reads, 1 ms CPU | same |
| Viewer open, idle, per second | 3 file reads, 6 realpath, 0 Git, ~2 ms CPU | 152 file reads, 6 realpath, 0 Git, ~18 ms CPU |
| Per external edit, workflow view | 23 Git, 9 reads, ~24 ms CPU, 1 GET | 125 Git, 442 reads, ~197 ms CPU, 1 GET |
| Per external edit, project view | 43 Git, 9 reads, ~35 ms CPU, 1 GET | 247 Git, 381 reads, ~316 ms CPU, 1 GET |

Observations:

- No viewer means no watch work, as intended.
- One SSE event per changed file per tick; the UI makes one GET per edit.
- Every refresh re-inspects every repository (about 6 Git commands per
  submodule) even when only a definition file changed. The project view does
  it twice per refresh: `projectResponse()` runs `repositories()`, then
  `workflowSummaries()` runs it again.
- Every refresh re-reads and re-validates every workflow file; the idle poll
  reads every workflow file each second.
- The ~2 s target is met for single sequential edits in both cases. The
  costs are in Git and full re-reads, not in the polling interval.

## Update cases probed (representative data)

| # | Case | Baseline |
| --- | --- | --- |
| 1 | UI save writes the definition file | pass |
| 2 | External edit after a UI save appears | pass (801 ms) |
| 3 | Burst: open workflow edited + a workflow created | **lost**: content never refreshed |
| 4 | Burst: open workflow + `project.yaml` | pass (event order happened to favour it) |
| 5 | Burst: two workflow files saved together | **lost** |
| 6 | Open workflow deleted | **invisible** (no banner; file not recreated) |
| 7 | Deleted open workflow restored | **lost** (view never reloaded) |
| 8 | Open workflow renamed | **invisible** |
| 9–11 | Project view: workflow created / renamed / deleted | pass (~1 s) |
| 12–13 | Malformed save, then valid save | pass (error ~1.1 s, recovery ~1.0 s) |
| 14 | Service stopped: outage visible | **no indication** |
| 15 | Edit made while the service was down | **lost**: not shown 10 s after restart |
| 16 | Submodule working tree becomes dirty | **not shown** |
| 17 | Root `git add` | **not shown** |
| 18 | Root `git commit` | **not shown** |
| 19 | Submodule initialized | **not shown** |
| 20 | Fast workspace switch (A→B→A) | pass |
| 21 | Registry changed while a view is open | **not shown** |

Registry: a missing registry file returns HTTP 500 `ENOENT: …`, and the UI
shows that error instead of an empty setup state. A malformed one returns a
bare JSON parse message without naming the file.

Causes:

- **3, 5, 7: coalescing replaces a content refresh with a context-only
  refresh.** The workflow view keeps one pending timer. An event for the open
  file schedules `load()`. A later event in the same batch (another file, or
  the `workflows` list event that follows any create/delete) clears that timer
  and schedules `load({contextOnly: true})`. Case 4 passed only because the
  `project` event is emitted before the workflow event.
- **6, 8:** after a delete, the next event is always the list event, so
  the `missing` problem never reaches the view.
- **15:** on reconnect the watcher starts with no snapshot, so it emits nothing
  for changes made while it was down, and the UI does not re-read on
  reconnect. **14:** the views ignore EventSource errors.
- **16–19:** only `project.yaml`, `.gitmodules` and workflow files are
  watched. Git state refreshes only as a side effect of a definition change or
  a page reload.
- **21:** the workspace list is read only when a view opens.

## Operation matrix: existing entry points

| Operation | Person (p1) | IDE agent (p1) | Gap |
| --- | --- | --- | --- |
| Create project | none (the project view can only create a missing `project.yaml`) | none; `scripts/seed.ts` builds a disposable fixture | both missing |
| Register workspace | hand-edit the ignored `registry.json` | the same | no UI, no command |
| List workspaces / status | project view | `GET /api/workspaces`, `GET …/project` (raw JSON) | no CLI |
| Edit intent, goals, graph, descriptions | project view, workflow editor | file editing; `PUT` with a full JSON model | none (file editing is the route) |
| Create workflow | project view "New workflow" | file editing; `POST …/workflows` | none (CLI convenience missing) |
| Add submodule | project view form (`Workspace.addSubmodule`) | `git submodule add`; `POST …/submodules` | CLI route missing |
| Validation | workflow editor and project view (`shared/validate.ts`) | `GET …/workflows/<file>` JSON | no CLI |
| Approve | workflow editor (`Workspace.approve`) | `POST …/approve` with JSON | no CLI |
| Auto-arrange | workflow editor button; **the algorithm lives only in the browser** (`src/layout.ts`) | **none** | browser-only; move the layout to `shared/` |
| Rename/delete files, repair YAML | VS Code | file editing | none |
| Diffs, commits, history | VS Code / Git | Git | none |

## Decisions for steps 2–5

**CLI.** `wfe` will be a thin command-line client in the experiment
(`cli/wfe.ts`), run by Node directly, as the service is. It calls the same
shared and server modules that the service uses (`Workspace`, the registry
module, `shared/validate.ts`, `shared/canonical.ts`, a shared layout module).
It does not call the service over HTTP, so **no command requires the running
service**. `wfe serve` starts the service, and `wfe status` reports whether
one is answering. Writes go to the files, and an open browser follows them
through the watcher. Commands:

| Command | Reads / changes |
| --- | --- |
| `wfe create <dir> --id --name --intent --goal…` | creates a project and registers it |
| `wfe register [<dir>]` | records an existing workspace's Git root |
| `wfe list` | registered workspaces and what is observed |
| `wfe status [<workspace>]` | project, repositories (Git state), workflows with issue counts and approvals |
| `wfe validate [<workflow>]` | validation issues |
| `wfe workflow new <id> [--name]` | creates a workflow file with the editor's template |
| `wfe approve <workflow> intent\|definition --approver <name>` | approval record (same code as the UI) |
| `wfe arrange <workflow>` | rewrites `layout` with the UI's auto-arrange algorithm |
| `wfe add-repo <path> <location>` | `git submodule add`, the same operation as the UI |
| `wfe serve` | starts the service for this registry |
| `wfe setup <area>` | creates or refreshes an authoring area and its `AGENTS.md` |

A workspace is selected by `--workspace <id>`. Without it, the CLI uses the
registered workspace that contains the current directory. Workflows can be
named by file name or by ID. `--json` gives structured output.
Exit codes: 0 success, 1 the operation was refused or found errors, 2 usage
error.

**Layout of an authoring area** (default `pj-agdev/.local/workflow-editor-p2/`):

```
AGENTS.md          generated from a tracked template
wfe                generated launcher pinning this area's registry
registry.json      this area's workspaces only
sources/           local bare repositories (e.g. <id>-devdocs.git)
pj-<id>/           projects created here
```

A project root gets `project.yaml`, `.gitignore` (`.local/`), an empty
`.local/`, and `devdocs` as a submodule whose URL is relative to the project
root (`../sources/<id>-devdocs.git`). The root has no remote, so Git resolves
the relative URL against the work tree, and the area can be moved as a whole.
Two commits are made: the devdocs source's initial commit (`README.md`,
`workflows/`) and the root's initial commit. Both use the person's
`git config user.name/email`. If either is missing, creation stops before it
writes anything. Local file transport is enabled per command only.

Browser creation writes only inside the service's authoring area. Browser
registration writes only the registry. The CLI is the person's or agent's own
process and takes any destination. That difference will be recorded in the
matrix.

**Update reliability** (step 4): keep a per-view refresh set instead of one
timer, re-read on every SSE (re)connect, and show the outage. Separate the
cheap definition poll from a slower Git-state poll and add a Refresh action.
Stop the full Git re-inspection on definition-only changes, and fix the double
inspection in the project view.

## Files

- `pj-agdev/experiments/workflow_editor/checks/measure.ts`, `checks/bench/counters.ts`
- Raw baseline record: `pj-agdev/.local/workflow-editor-measure/baseline.json` (ignored)

The input named "accepted p2 execution proposal" is not a file in the
repositories. This phase follows `plan.md`, which states the scope.
