# workflow_editor p1 — report

## Summary

The p1 MVP is in `pj-agdev/experiments/workflow_editor/`. It is a local service
plus a browser editor for Git-backed project and workflow definitions.
The YAML files stay the authority and can be edited in any text editor or
agent IDE in turn with the UI. The full acceptance scenario passed against
real local repositories: seed, edit in the UI and externally, validation and
recovery, approvals and stale states, and a Git publish reproduced in a
second workspace.

**Assumption, as agreed: writers take turns.** UI edits and external file
edits are sequential. There is no revision-based write, merge, conflict UI or
concurrent-write guarantee. The external-change guard only keeps an unsaved UI
draft from being silently replaced.

Step reports: [report1](report1.md) contract and fixtures,
[report2](report2.md) project surface, [report3](report3.md) workflow editing,
[report4](report4.md) external editing, [report5](report5.md) approvals and
the end-to-end trial.

## What passed

| Area | Evidence |
| --- | --- |
| Contract, YAML subset, digests, round trip | 11 tests (`test/contract.test.ts`) |
| Git inspection, project edits, submodule add, workflow discovery/creation, path bounds | 9 tests on real fixtures (`test/workspace.test.ts`) |
| Save/reopen, atomic save, failed save, malformed never overwritten, HTTP origin/host/type guards | 6 tests (`test/persistence.test.ts`) |
| Approval records, stale rules, refusals | 5 tests (`test/approval.test.ts`) |
| Project screen | 21 browser checks (`checks/step2.ts`) |
| Workflow editing, modes, save and reopen | 42 browser checks (`checks/step3.ts`) |
| External edits, draft guard, malformed recovery, failed saves | 33 browser checks (`checks/step4.ts`) |
| Acceptance scenario 1–7 | 48 browser checks (`checks/e2e.ts`) |

Also: `tsc` clean and `vite build` passes. The single-process production mode
(`npm start`) serves the UI and API from one origin.

## How to start the editor

From `pj-agdev/experiments/workflow_editor/`:

```sh
npm ci
npm run seed -- --reset   # fixture under pj-agdev/.local/workflow-editor/
npm run service           # service on http://127.0.0.1:8095
npm run dev               # UI on http://127.0.0.1:5175 (proxies /api)
# or, single process with the production build:
npm start                 # http://127.0.0.1:8095
```

The registry of workspaces is the ignored
`pj-agdev/.local/workflow-editor/registry.json` (`--registry` or `WFE_REGISTRY`
to point elsewhere). To edit a real project, register its workspace root there.
The file contract is in `docs/contract.md` in the experiment.

## Design in brief

- **Files:** `project.yaml` (`ag.project.v1`: id, name, intent, goals) at the
  project root. Workflows are `devdocs/workflows/*.yaml` (`ag.workflow.v1`:
  intent, repository bindings with readonly/editable access, typed nodes, edges,
  approvals, optional layout). Repositories come from `.gitmodules` and Git, not
  from `project.yaml`.
- **Editing preserves the file:** the edited model is applied onto the parsed
  YAML document (`yaml` 2.9). Comments, key order, styles and unknown fields
  survive, and an unchanged document is written back byte for byte. Anchors,
  aliases, tags, merge keys, several documents and wrongly shaped fields are
  rejected without rewriting.
- **Approvals:** SHA-256 over canonical JSON of an intent projection and a
  definition projection. Layout, approvals, workflow id/name, comments and
  formatting are excluded.
- **Service:** Node HTTP on 127.0.0.1, run from TypeScript without a build. Git
  runs with argument arrays, and file access is bounded to registered roots
  (including symlinks). Saves are temp file + rename. A 1 s polling watcher pushes
  changes over SSE while a browser is connected. Only local Host names are
  answered, and browser writes come only from the editor's origins.
- **UI:** TypeScript + Vite, with DOM cards and SVG edges. Compact and mini modes
  render the same positions at two densities. There are no Run/Share actions and
  no activity states.

## Deviations from the plan

- **`readonly` restricts nothing yet in the editor.** p1 has no operation on
  repository contents to restrict. The editor writes only definition files and
  runs `git submodule add`. Access is declared and displayed (card badges, the
  project view's per-workflow access column), as the contract states.
- Definition approval also covers node `name` (the card label).
- The `yaml` library writes one space before a trailing comment and no padding
  inside flow collections. Both are documented normalizations.
- New UI features beyond the plan's list, each to make the plan's items
  usable: "Auto-arrange" (a layout-only action), free-slot placement of new
  nodes, and turning programmatic scrolls of the canvas into pans.
- Defects found and fixed during the steps: an early-typing race on a new
  workflow, overlapping placement of new nodes, and a stale "Saved" label after
  reload. Several first-run failures were in the check scripts themselves; the
  step reports name them.

## Remaining limitations

- Writers take turns. A write landing between the service's read and its
  rename is not detected. Two browser tabs editing the same file are two
  writers.
- Change latency is up to the polling interval (1 s). The watcher covers
  `project.yaml`, `.gitmodules` and up to 200 workflow files. It does not
  watch submodule working trees (dirty state refreshes on reload or on the
  next definition change).
- The approver is declared, not authenticated, and the YAML can be edited
  directly. Approvals attest the definition only, not execution permission or
  repository contents.
- Workspaces are local registrations only. A workspace on another machine is
  listed only if registered here, and its availability is whatever this
  machine can observe.
- No undo. Graph cycles are rejected rather than modelled. No conditional
  edges, loops or retries.
- Submodule removal, commits and pushes are outside the editor.
- Layout is in compact-card units. Mini mode scales positions by 0.66, which
  fits the current card sizes but is a fixed ratio.
- The browser checks are scripts, not a test runner suite. They need the service and the dev
  server running, and they reseed the shared fixture.

## Deferred beyond p1 (unchanged)

Workflow execution, agent chat inside the editor, Gitea integration,
agdevworld integration, migration of existing projects, remote workspace
discovery/deployment, conditional branches, loops, retries, failure
transitions, author authentication, OS-level readonly enforcement and robust
concurrent editing.
