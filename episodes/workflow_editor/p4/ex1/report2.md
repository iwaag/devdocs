# workflow_editor p4/ex1 — Step 2: system integration

Step 2 of [plan.md](plan.md): the editor is imported into agdevworld, built
and installed by agdevworld's own script, and the running service and `wfe`
launcher now run that code. The operating catalog is unchanged in this step
(both editor workspaces and the editor project are still registered; Step 4
removes them).

Commits:

- agdevworld `35ab47a`: `workflow_editor/`, `scripts/wfe-release.sh`,
  `package.json` entry points, `.dockerignore`, README_DEV.
- agautolab `167ae13`: the executor test runs against the new location.
- pj-agdev `1a1606c`: submodule pointers; launchd template comment and notes;
  frozen-copy notice.

## Import

`agdevworld/workflow_editor/` = `git archive 9c29117` of the retired
`developer/pj-workflow-editor`, minus `project.yaml`, `devdocs/`
(`improve-editor`) and `scripts/deploy.sh`. It is tracked directly in the
agdevworld repository (no submodule) and keeps its own `package.json` and
lockfile.

Changes made while importing:

| Area | Change |
| --- | --- |
| Registry selection | `cli/wfe.ts` and `server/main.ts` no longer default to a registry relative to the code (`<code>/../../.local/workflow-editor/registry.json`). The registry is required: `--registry` or `WFE_REGISTRY` (the area launcher sets it). Without one, `wfe` exits 2 with "no registry chosen" and the service refuses to start. |
| Fixture paths | The p1 seed, the browser checks and their screenshots use the package's own ignored `.local/` (`.local/fixture/`, `.local/p4/`) instead of `pj-agdev/.local/workflow-editor*`. `npm run service` / `npm start` pass the fixture registry explicitly. |
| Names | `EXPERIMENT` → `PACKAGE`; local `experiment` variables renamed; the `wfe setup` stamp names `workflow_editor/templates/`. |
| Version | New `server/version.ts`: `GET api/version` and `wfe version` report the package directory (symlinks resolved) and, in an installed release, `RELEASE.json` (agdevworld commit, tree of `workflow_editor/`, build time). |
| Navigation | The dashboard header starts with "← agdevworld": `/` (the operation room) through the `/wfe/` route, `http://localhost:8093/` otherwise. The operation room's "agdev dashboard ↗" is unchanged. |
| README | Ownership, explicit registry, fixture paths and the release route rewritten; the self-development and `deploy.sh` text removed. |

## Build, release and recovery commands (agdevworld)

- `npm run editor:install`, `npm run editor:check`; `npm run check` = the
  frontend build + the package check. The Docker image excludes
  `workflow_editor/`.
- `scripts/wfe-release.sh build|activate|deploy|rollback|status <area>`:
  - **build** archives `workflow_editor/` at an agdevworld commit into
    `<area>/service/releases/<commit>/`, runs `npm ci && npm run check` there,
    writes `RELEASE.json`. A failure leaves nothing in use changed.
  - **activate** refuses while an execution attempt is open, switches
    `service/current`, rewrites `bin/wfe`, kickstarts `com.agdev.wfe-service`,
    then requires both `GET api/version` and `bin/wfe version` to name the
    commit within 30 s. Otherwise it switches back to the previous release and
    restarts. The service and the launcher both run `service/current`, so they
    always move together.
  - **rollback** returns to `service/previous`; **status** reads it all back.
  - Nothing is fetched from `developer/pj-workflow-editor`.

## Validation

| Check | Result |
| --- | --- |
| `npm run check` in `agdevworld/workflow_editor` | 87 tests pass, 0 fail (disposable-Gitea tests included); tsc and build clean |
| agdevworld `npm run check` (frontend build + package check) | pass |
| `node checks/runs.ts`, `checks/agdev.ts`, `checks/exec.ts` from the new location | all pass (33, 13 and 18 checks) |
| agautolab `uv run pytest -q tests/test_wfexec.py` (default `WFE_PACKAGE` = `agdevworld/workflow_editor`) | 3 passed |
| **Clean checkout**: `git clone` of agdevworld from GitHub into a scratch directory, `npm ci`, `npm run editor:install`, `npm run check` | pass, 87/87. Neither editor workspace, the frozen copy nor the Gitea editor repository is involved. |
| `wfe list` with no registry | exit 2, "no registry chosen" |

Release script, on a synthetic area:

| Case | Result |
| --- | --- |
| Activate with an attempt `running` in the area's exec store | refused, `current` unchanged |
| Activate with no service answering (health fails) | switched, failed health in 2 s, restored `current` to the previous release, launcher rewritten to it, exit 1 |
| Activate with a service started on the new `current` | healthy; `service/previous` recorded; `status` shows the service and the launcher on the same commit |

## Live activation

Before activating: 0 open attempts, 0 queued or claimed jobs.

`scripts/wfe-release.sh deploy pj-agdev/.local/agdev`:

```text
built and checked: releases/35ab47ae42d4f3aacb2775ee8507fab7743c6501
current: releases/35ab47ae… (was: 9c29117238afb8bb405c4b9a98930ac7388ec179)
restarted com.agdev.wfe-service
healthy: the service and bin/wfe run agdevworld 35ab47ae…
```

Afterwards:

- `localhost:8093/wfe/api/version`: package
  `.local/agdev/service/releases/35ab47ae…`, release agdevworld `35ab47a`,
  tree `6562bb4`, registry = the area's `registry.json`.
- `bin/wfe version`: the same release.
- The dashboard through the route lists the unchanged catalog (`rts-vs-bot`,
  `workflow-editor`) with the executor `available — autolab; idle`. autolab's
  executor kept its heartbeat through the new launcher; its plist and
  `wfexec.toml` did not need to change.
- `service/previous` = `9c29117…`, the rollback target (accepted by its
  dashboard answering, since it predates `RELEASE.json`).
- The installed `com.agdev.wfe-service.plist` was regenerated from the
  template (comment only; the program arguments are identical, so no reload).

## Not yet

- No protection of system paths and no exclusion of the retired repository:
  Step 3.
- The catalog still has the editor project and both workspaces: Step 4.
