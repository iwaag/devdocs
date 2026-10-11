# workflow_editor p4/ex1 — Step 1: inventory and ownership decision

Step 1 of [plan.md](plan.md). Read-only: nothing was changed, stopped or
unregistered in this step.

## Runtime state (2026-10-11, about 13:25 JST)

| Item | State |
| --- | --- |
| `com.agdev.wfe-service` | loaded by the person (12:41), running; node on `127.0.0.1:8098` |
| `com.agdev.agautolab-wfexec` | loaded, running; heartbeat `autolab; idle` |
| Installed plists | identical to the templates in `pj-agdev/devenv/launchd/` with the projects root filled in |
| `service/current` | `releases/9c29117…` (the only release) |
| Area launcher `bin/wfe` | runs `service/current/cli/wfe.ts` with `WFE_REGISTRY` = the area's registry |
| `agautolab/.local/wfexec.toml` | `wfe` = the area launcher |
| agdevworld route | nginx `:8093` on 127.0.0.1 only, `/wfe/` → host `:8098`; `/` serves the agdevworld SPA |
| Exec store (`.local/exec/exec.sqlite`) | jobs 0, attempts 0, checkpoints 0; logs folder empty |
| Runs | none, in any workspace (`devdocs/runs/` absent in all three) |

So no editor job, attempt or run exists: there is nothing to stop or drain
before cutover, and no real RTS run has happened since p4 stage 4.

## Catalog (registry.json)

| Kind | Entry | Fate in this correction |
| --- | --- | --- |
| Project | `rts-vs-bot` → root `gitea:131` | keep |
| Project | `workflow-editor` → root `gitea:132` | remove |
| Workspace | `rts-vs-bot` → `.local/agdev/pj-rts-vs-bot` | keep |
| Workspace | `workflow-editor` → `.local/agdev/pj-workflow-editor` | remove |
| Workspace | `workflow-editor-2` → `.local/agdev/pj-workflow-editor-2` | remove |
| Repository | `gitea:130` `developer/study-rts` (study) | keep |
| Repository | `gitea:131` `developer/pj-rts-vs-bot` (root) | keep |
| Repository | `gitea:132` `developer/pj-workflow-editor` (root) | remove, and exclude by identity |

RTS workspace: `main` = `origin/main` = `7e662e1`, clean;
`study/study-rts` at `d2734db` (= its Gitea `main`); workflows `build-game`
and `improve-game`.

## Editor source locations

| Location | Commit | Working tree |
| --- | --- | --- |
| Gitea `developer/pj-workflow-editor` `main` | `9c29117` (only branch) | — |
| Workspace `workflow-editor` | `9c29117`, = origin | clean (ignored: `dist/`, `node_modules/`) |
| Workspace `workflow-editor-2` | `9c29117`, = origin | clean |
| Release `service/releases/9c29117…` | `9c29117` (git archive) | no file differs from the workspace except build output |
| Frozen copy `pj-agdev/experiments/workflow_editor` | pj-agdev `cae13c6` (code as of `91be3bb`) | clean |

`diff -r` (excluding `.git`, `node_modules`, `dist`, `.local`):

- workspace 1 = workspace 2 = release.
- frozen copy → workspace: `cli/wfe.ts` (two `wfe status` lines: a
  directory-mode devdocs row and an "N unpublished" count), `README.md`
  (frozen notice vs. own-project notice), plus `project.yaml`,
  `devdocs/` (README, `workflows/improve-editor.yaml`) and
  `scripts/deploy.sh`, which exist only in the editor project.

No change exists only in one place outside Git: no uncommitted work, no
other branch, no release built from a commit that is not on Gitea.

## Decision: what is imported

Import source: **`9c29117`**, the deployed release, which already contains
everything in the frozen copy.

| Imported | Not imported |
| --- | --- |
| `src/ shared/ server/ cli/ test/ checks/ docs/ templates/ examples/ scripts/seed.ts`, `index.html`, `package.json`, `package-lock.json`, `tsconfig.json`, `vite.config.ts`, `.gitignore` | `project.yaml`, `devdocs/` (self-development definition: `improve-editor`), `scripts/deploy.sh` (replaced by an agdevworld-owned deploy), `node_modules/`, `dist/`, operating data |

## Path assumptions found (to change in Step 2)

- `cli/wfe.ts` and `server/main.ts` default the registry to
  `<code>/../../.local/workflow-editor/registry.json` — a p1 fixture area
  relative to the code. Moved code with this default would silently open a
  different (new, empty) registry.
- `cli/setup.ts` generates launchers that point at the code directory, and
  stamps "experiments/workflow_editor/templates".
- `checks/*.ts` and `scripts/seed.ts` put fixtures and screenshots in
  `pj-agdev/.local/workflow-editor*`, three levels above `checks/`.
- `README.md` documents the experiment and the self-development route.
- agautolab `tests/test_wfexec.py` defaults to the frozen copy through
  `WFE_EXPERIMENT`.
- The launchd comment and `pj-agdev/devenv/launchd/README.md` name
  `scripts/deploy.sh` of the editor project.
- agdevworld's `Dockerfile` does `COPY . .`; a package inside agdevworld must
  be kept out of the frontend image (`.dockerignore`).

## Next

Step 2: import into `agdevworld/workflow_editor`, fix these paths, and add
agdevworld-owned build/check/deploy commands. The catalog is not touched
until Step 4.
