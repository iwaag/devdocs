# workflow_editor p4 — Stage 4: continuing RTS and editor development (in progress)

Stage 4 has two parts:

- **Prepared:** inventory, project roots, Gitea, fresh workspaces, and a
  deployment separate from the source being edited. This is everything
  except the real runs.
- **Outstanding:** one real improvement run per project, requested,
  answered and accepted by the person through agdevworld (step 5). Also the
  launchd jobs, which the Omni Agent was not allowed to load.

Synthetic tests do not complete this stage, and none are claimed for it.

Commits:

- pj-agdev `cae13c6`: launchd templates; the experiment copy marked frozen.
- agautolab: the executor's launcher script.
- Gitea `developer/pj-workflow-editor` `9c29117`: the deploy script.

## 1. Inventory

### RTS: `pj-agdev/.local/workflow-editor-p2/pj-rts-vs-bot`, the p3 area

The p3 area was read only, never changed or reset.

| Kept | How |
| --- | --- |
| Game source `game/` with its root history (5 commits) | the new root repository's history |
| Study notes `study/study-rts` with their history | shared repository `developer/study-rts` (category study) |
| Workflow `build-game` | `devdocs/workflows/build-game.yaml` |
| Knowledge of the p3 MVP run: braindump, plan, spec, implement and playtest reports, screenshots | `devdocs/notes/p3-mvp/` |

Not migrated:

- the p3 run record (`run.json`, `ag.workflow-run.v1`, old layout) and its
  history;
- the local bare sources in `sources/`;
- the p3 registry.

### Editor: `pj-agdev/experiments/workflow_editor`

The editor is a self-contained build unit: its own `package.json`, tests,
checks and docs. Its development history (21 commits) is the
`experiments/workflow_editor` path history of pj-agdev.

Not migrated: rehearsal and fixture areas
(`.local/workflow-editor*`). They are ignored local data.

## 2. Project roots

- **RTS: `developer/pj-rts-vs-bot`.**
  - The old root's history, plus one migration commit:
    - `project.yaml` moves to `ag.project.v2` with `devdocs: directory`;
    - the devdocs submodule is replaced by a devdocs directory (README,
      workflows, `notes/p3-mvp/`);
    - the study submodule URL becomes `../study-rts.git`.
  - New workflow `improve-game`: confirm-scope (talk) → implement → verify
    (headless tests + browser) → human-check (talk). Bindings: root
    editable, `study/study-rts` readonly.
- **Editor: `developer/pj-workflow-editor`, a repository of its own.**
  - Its history was split from pj-agdev with `git subtree split`, in a
    separate clone; pj-agdev's refs were not touched.
  - One registration commit adds `project.yaml` (id `workflow-editor`,
    `devdocs: directory`), `devdocs/README.md`, and the workflow
    `improve-editor`: confirm-scope → implement → validate (`npm run check`
    and the browser checks it touches) → human-check.
  - The copy in pj-agdev is left in place and marked **frozen** at the split
    (`91be3bb`). Removing it, or making it a submodule, is a decision left to
    the person.

Definition approvals were not recorded. They are the person's to give, and
they are not a gate.

The migration is one script, `.local/agdev/migration/migrate.sh` (ignored):

- the Gitea token reaches Git through `GIT_CONFIG_*` environment variables,
  not argv;
- the repositories are created empty, and the histories are pushed to them;
- registration uses `wfe repo register` and `wfe project register`.

## 3–4. Gitea and fresh workspaces

The dashboard (`wfe dashboard`, and `localhost:8093/wfe/`) lists:

- the two projects, each with `devdocs: directory`;
- their workspaces;
- `developer/study-rts`, used by `rts-vs-bot (study/study-rts)`.

Workspaces obtained from Gitea with `wfe project obtain`:

| Workspace | Path | Use |
| --- | --- | --- |
| `rts-vs-bot` | `.local/agdev/pj-rts-vs-bot` | autolab's, for the RTS run |
| `workflow-editor` | `.local/agdev/pj-workflow-editor` | autolab's, for the editor run |
| `workflow-editor-2` | `.local/agdev/pj-workflow-editor-2` | the Omni Agent's, for setup commits only |

Checks in the fresh workspaces:

- **RTS.**
  - `wfe status`: structure complete; both workflows valid.
  - `study/study-rts` is checked out at `d2734db`.
  - `node game/test/headless.js`: all 13 checks pass.
  - No file contains the Gitea token.
- **Editor.** `npm ci && npm run check`: 87 tests pass, type check and
  build clean, so the source is usable from Gitea alone.

## 6. The service in use is separate from the source being edited

`scripts/deploy.sh <area> [<commit>]` is in the editor project:

1. It archives the commit into `.local/agdev/service/releases/<sha>/`.
2. It runs `npm ci && npm run check` there. A failure changes nothing in use.
3. It points `service/current` at the release.
4. It rewrites `.local/agdev/bin/wfe`, the launcher that autolab and its
   agents use; it runs `current`, pinned to the area's registry.
5. It restarts `com.agdev.wfe-service`.

The first deployment is `9c29117`:

- the check passed in the release (87 tests);
- the release, started by hand, serves both projects through
  `http://localhost:8093/wfe/`;
- autolab's executor tests pass against it
  (`WFE_EXPERIMENT=.../service/current`, 3 passed).

launchd:

- **Templates.** `pj-agdev/devenv/launchd/com.agdev.wfe-service.plist.in`
  and `com.agdev.agautolab-wfexec.plist.in`.
- **Executor settings.** `agautolab/.local/wfexec.toml` sets `wfe` to the
  area's launcher.
- **Installed.** The service plist is in `~/Library/LaunchAgents/`.
- **Not loaded.** `launchctl bootstrap` was refused to the Omni Agent as
  unauthorized persistence. The person loads both jobs (below).
- **Not running now.** The Omni Agent stopped its session process after
  validation, so port 8098 is free for the job. The executor is not running
  either.

## Outstanding: what the person does

1. **Load the two jobs:**

   ```sh
   sed 's#__PROJECTS_ROOT__#'"$HOME"'/projects#g' ~/projects/pj-agdev/devenv/launchd/com.agdev.agautolab-wfexec.plist.in > ~/Library/LaunchAgents/com.agdev.agautolab-wfexec.plist
   launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.agdev.wfe-service.plist
   launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.agdev.agautolab-wfexec.plist
   ```

   The dashboard's status bar then shows "execution host: available".
2. **Request one improvement per project.** Open
   `http://localhost:8093/wfe/` (or the operation room's "agdev dashboard ↗"):
   - Open the project editor of workspace `rts-vs-bot`, then "Request a
     run", workflow "Improve the game".
   - Do the same for workspace `workflow-editor` (not `workflow-editor-2`),
     workflow "Improve the editor".

   Each request is your own words. Answer the confirm-scope question in the
   run view, check the result, and Accept (or Reject) with evidence.
3. **Redeploy after the editor run is accepted.** Then the Omni Agent
   redeploys the editor from the accepted commit (`scripts/deploy.sh`) and
   confirms that ongoing runs display again.

## Next entry point

After the two runs:

- Verify the records and the Gitea results from fresh workspaces.
- Redeploy the editor.
- Write the final `report.md`, keeping the foundation (Stages 1–3, done)
  separate from real use.
