# workflow_editor p4/ex1 — Step 4: cutover and cleanup

Step 4 of [plan.md](plan.md): the editor's registrations are retired, the
boundary release is active, and RTS is preserved. Both services were
restarted twice and the result read again each time.

Commits: agdevworld `2678c9f` (test fix), `6fe2b61` (CLI executor state),
`2a4e70c` and the following commit (launcher port); pj-agdev `77b4d7d`
(pointer).

## 1. Inventory and recovery copy

Taken just before the change, in
`pj-agdev/.local/agdev/.local/backups/p4ex1-20261011-134428/`:

- `registry.json` and `gitea.json`;
- `exec.sqlite` (`sqlite3 .backup`, consistent);
- the `service/current` target and the `bin/wfe` launcher.

| Item | State |
| --- | --- |
| Exec store | jobs 0, attempts 0, checkpoints 0; heartbeat `idle` |
| Runs | none in any of the three workspaces |
| `pj-workflow-editor`, `pj-workflow-editor-2` | both `9c29117`, no local changes; everything is in the import (report1) |
| `pj-rts-vs-bot` | `7e662e1` = Gitea `main`, clean; `study/study-rts` `d2734db` = Gitea `main` |

Nothing had to be stopped, drained or recorded as pending: there was no
editor job, no RTS work, and no live attempt under which the CLI could have
been replaced.

## 2. Boundary configuration and release

`pj-agdev/.local/agdev/system.json` (ignored; system configuration):

| Kind | Entry |
| --- | --- |
| Protected | `../../agdevworld` (the editor source), `../../agautolab`, `../../experiments/workflow_editor`, `../gitea` (credentials) |
| Retired | Gitea id 132, `developer/pj-workflow-editor` |

The first deployment of the boundary code (`0b9d97a`) **failed its own check
inside the candidate release**, and nothing in use changed. The cause was a
test regex that expected the protected package path to contain
"workflow_editor". In a release the path is `releases/<commit>`; the refusal
itself was right. After the test fix (`2678c9f`) the release built (93/93)
and activated healthy.

The failed candidate's folder
`service/releases/0b9d97a….tmp` is still there: the safety check refused
its removal, because the shell's working directory was inside it at the time.
It is inert (no `RELEASE.json`, never current), and the person may delete
it.

With the boundary active and before unregistration, both editor workspaces
already showed on the dashboard as "unavailable: refused … retired". HTTP
`GET …/workspaces/workflow-editor/project` returned 403.

## 3. Catalog cleanup

```text
$ bin/wfe project unregister workflow-editor
Unregistered project workflow-editor.
  workspace workflow-editor (…/pj-workflow-editor; the folder is kept)
  workspace workflow-editor-2 (…/pj-workflow-editor-2; the folder is kept)
  repository gitea:132 developer/pj-workflow-editor (on Gitea it is kept)
```

The two folders were moved to `.local/agdev/.local/retired/` as unregistered
archives, with a README. This places them under the protected control data.

The registry now holds:

- project `rts-vs-bot` → `gitea:131`;
- workspace `rts-vs-bot`;
- repositories `gitea:130` `developer/study-rts` (study) and `gitea:131`
  (root).

Gitea `developer/pj-workflow-editor` and its history are not deleted.

## 4. Same code, same catalog

After the service and the executor were restarted
(`launchctl kickstart -k` on both):

| Reader | Code | Catalog |
| --- | --- | --- |
| Browser through `localhost:8093/wfe/` | (served by the service below) | RTS project with workspace `rts-vs-bot`; `developer/study-rts` used by `rts-vs-bot (study/study-rts)`; no other rows; "execution host: available — autolab; idle" |
| Service `GET /wfe/api/version` | agdevworld `6fe2b61`, release dir `service/releases/6fe2b61…` | registry = `.local/agdev/registry.json` |
| `bin/wfe version` / `wfe dashboard` / `wfe list` | `6fe2b61` | the same catalog |
| autolab executor | `wfexec.toml` → `bin/wfe` → `6fe2b61` | heartbeat from the new process (pid changed) |

Two mismatches were found and fixed during this step:

- **Executor state.** `wfe dashboard` said "Execution host: not-configured"
  while the browser said "available": the CLI never read the area's exec
  store. Fixed in `6fe2b61`; both now say "available — autolab; idle".
- **Service port.** `wfe status` looked for the service on the default port
  8095. The launcher written by `wfe-release.sh` now also pins `WFE_PORT`
  (8098, from the service URL), so it says "Editor: running at
  http://127.0.0.1:8098/".

The `migration/` folder of the area (stage-4 migration script) and the Gitea
read cache for the retired repository remain as ignored data. Nothing reads
them.

## 5. Stale URLs, direct requests and queued jobs

On the operating system, after cleanup:

| Probe | Result |
| --- | --- |
| `GET /wfe/api/workspaces/workflow-editor{,-2}/project`, `…/runs` | 404 "not registered" |
| `POST …/workspaces/workflow-editor{,-2}/requests` | 404 |
| `POST /wfe/api/agdev/projects/workflow-editor/workspaces` (obtain) | 404 |
| `POST /wfe/api/agdev/projects/register` and `…/repositories`, `developer/pj-workflow-editor` | 403, retired |
| Browser `localhost:8093/wfe/#/ws/workflow-editor` | `workspace "workflow-editor" is not registered`; no page errors |
| `wfe project register developer/pj-workflow-editor`; `wfe repo register … --category other` | exit 1, retired |
| `wfe register` of the archived editor workspace | exit 1, inside the area's control data |
| `wfe register` of `agdevworld`, of `agdevworld/workflow_editor` (its Git root is agdevworld), and of a symlink to agdevworld in a scratch directory | exit 1, protected |
| `wfe register` of `pj-agdev` and of `service/current` | exit 1: contains the running release |
| a fresh clone of `developer/pj-workflow-editor` outside the area, `wfe register` | exit 1: its origin is retired |
| `wfe add-repo tools/editor ../pj-workflow-editor.git` in the RTS workspace | exit 1, retired; RTS working tree unchanged |
| A job injected into the live queue (`workspace = workflow-editor`, `improve-editor/run-001`) | The running executor dropped it on its next tick: "workspace "workflow-editor" is not registered". No attempt was opened. |

Throughout, the registry stayed at one project, one workspace and two
repositories.

## 6. Second restart

Both jobs were kickstarted again, and the same reads were repeated:

- the service is on `6fe2b61`;
- the executor is `available — autolab; idle`;
- the projects are `[rts-vs-bot: [rts-vs-bot]]` and the repositories
  `[gitea:130 developer/study-rts used by rts-vs-bot]`, with no unlinked
  workspaces;
- the editor IDs return 404 and `rts-vs-bot` returns 200.

## RTS preservation

- Gitea `developer/pj-rts-vs-bot` `main` = `7e662e1` and
  `developer/study-rts` `main` = `d2734db`, as before.
- The workspace is clean, at the same commit.
- `wfe status --workspace rts-vs-bot`: structure complete; `build-game` and
  `improve-game` valid; no runs.

Screenshots (ignored), in `pj-agdev/.local/workflow-editor-p4/screenshots/`:

- `p4ex1-dashboard.png`;
- `p4ex1-operation-room.png`, reached through the dashboard's "←
  agdevworld", whose "agdev dashboard ↗" returns to `/wfe/`;
- `p4ex1-stale-editor-url.png`.

## Releases now on disk

| Release | Status |
| --- | --- |
| `6fe2b61` | current |
| `2678c9f` | previous, the rollback target |
| `35ab47a` | kept |
| `9c29117` | the old editor-project release; no longer referenced by anything in use |
| `0b9d97a.tmp` | the failed candidate (above) |
