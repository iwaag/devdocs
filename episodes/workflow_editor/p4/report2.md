# workflow_editor p4 — Stage 2: Gitea and global resource management

Stage 2 of [plan.md](plan.md) is implemented:

- pj-agdev `2417bd1` (`experiments/workflow_editor`)
- agdevworld `8022fef`

All evidence is synthetic. It comes from a disposable Gitea container and
temporary registries. The person's Gitea was only read: its version, and the
developer token's login. No repository was created on it in this stage.

## Implementation

### Gitea client and credentials

`server/gitea.ts` holds the Gitea client.

**Host setting.** A `gitea.json` beside the registry holds `{url, owner,
tokenFile}`. `--gitea` or `WFE_GITEA` can point elsewhere. The operating
setting is `pj-agdev/.local/agdev/gitea.json`, which points at the existing
developer token outside every repository.

**Token handling.** The token reaches Git as an `http.<gitea>/.extraHeader`
through `GIT_CONFIG_*` variables. It is never written into:

- a URL or a remote;
- a Git config file;
- a project file or a run record.

**URLs in projects.** Submodule URLs are relative (`../<name>.git`, or
`../../<owner>/<name>.git`). `.gitmodules` therefore names no host.

**Gitea's API lags behind pushes.** Observed on 1.27.1:

- the `empty` flag, the branch list and the raw-file endpoint stay stale for
  roughly a second after a push;
- emptiness is therefore read with `git ls-remote`;
- file contents (`project.yaml`, `.gitmodules`) are read with a shallow fetch
  into `<area>/.local/gitea-cache/`.

The shallow fetch also names the exact commit that was read.

### Creation on Gitea, with recovery

`createProject` takes a Gitea client and runs these steps in order:

1. Create the root repository, and in submodule mode the devdocs repository,
   on Gitea.
2. Push devdocs' initial commit.
3. Initialise the root with `origin` set to the clean URL.
4. Add devdocs as a submodule with a relative URL, using the header
   environment.
5. Make the initial commit and push the root.
6. Register the repositories, the project and the workspace.

**Journal.** Every step is recorded in `.local/wfe-create.json`, including
the Gitea ids of the repositories the creation made. `resume` continues
after the finished steps.

**Same-named repositories.** `ensureRepo` decides what to do with an existing
repository of the same name:

| Existing repository | Result |
| --- | --- |
| Made by this creation (id in the journal) | kept |
| Same name, empty, and `reuse` given | reused |
| Anything else | a collision; nothing in it is changed |

**Without Gitea.** The local path remains, but only for tests and fixtures:

- a directory-mode project with no remote;
- the submodule mode with a local bare source.

It is reached only with `--no-gitea` or when no setting exists. Through the
agdevworld route it is refused.

### Registry: projects, repositories, workspaces

`registry.json` gains `projects` and `repositories` lists.

| Entry | Holds |
| --- | --- |
| Project | id and the key of its root repository |
| Repository | keyed by `gitea:<id>`; owner/name as registered, category, description |
| Workspace | its project id |

Project names, intents, submodule membership, gitlinks and run state are not
copied into the registry.

Every write holds a cross-process lock (`server/lock.ts`). The lock is a
SQLite `BEGIN IMMEDIATE` on a host-local lock file, so the OS releases it
when the holder exits or crashes. There is no stale-lock takeover. Writes
happen only when something changes.

### agdev operations: `server/agdev.ts`, CLI and HTTP

| Operation | CLI | HTTP |
| --- | --- | --- |
| Create a project on Gitea | `wfe create` (with the setting) | `POST api/agdev/projects` |
| Register an existing project root (ag.project.v2 only; its devdocs repository too) | `wfe project register <owner>/<name>` | `POST api/agdev/projects/register` |
| Obtain a workspace (clone with submodules into the area, register) | `wfe project obtain <id>` | `POST api/agdev/projects/<id>/workspaces` |
| Register or create a shared repository | `wfe repo register` / `create` | `POST api/agdev/repositories` |
| Add a registered repository to a project | `wfe repo add <repo> <path>` | `POST api/workspaces/<ws>/shared` |
| Global listing | `wfe dashboard` | `GET api/agdev` |

Adding a shared repository refuses a path that is already a submodule or
already exists.

**Publication status.** Each repository row now carries the commits that no
remote-tracking ref contains, with the upstream as last fetched. The Project
Editor shows them as "N unpublished" or "published". Work completion and
publication are separate; the publication operation and its retry are
Stage 3, step 5.

### The dashboard and the Project Editor

The editor's home page is now the **agdev dashboard**:

- **Status bar:** Gitea, execution host, registry and read time.
- **Projects:**
  - purpose and devdocs mode, with the source and time of that reading;
  - workspaces with their ongoing runs and waits, and links to runs and the
    Project Editor;
  - "Obtain a workspace from Gitea".
- **Shared repositories:**
  - listed once each, with category, description and Gitea link;
  - every project that uses one, each with its source and time;
  - "no project uses it yet" kept apart from "unknown".
- **Entrances:** create a project, register an existing project, register or
  create a shared repository, and register a local workspace by path. The
  service refuses the last one through agdevworld.

The dashboard keeps unavailability apart from an empty listing:

- Gitea unreachable: the status bar says so, and repositories show "not read",
  not missing or empty.
- A project whose definition could not be read says that, with the source it
  tried.
- Usage that could not be read is "unknown for …", not "unused".

The Project Editor remains one project's resources. It adds:

- "Add existing repository": search the registry, choose a destination path,
  and the repository is added as a staged submodule of this project only.
- Publication chips on repository rows.

Directory-mode devdocs appears there as a project resource of kind
`directory`, not as a Repository.

### Route from agdevworld

`compose.yaml` publishes a second nginx listener on `127.0.0.1:8093` only.
Its `/wfe/` location proxies to the editor service on the host loopback (port
8098 for the operating instance) and adds `x-wfe-route`. The `:8090` site is
unchanged; on 8090, `/wfe/` falls through to the SPA's `index.html`.

Why the route is not on 8090: nginx sees the same Docker gateway address for
local and LAN clients, so a `/wfe/` on the LAN-published 8090 cannot be
restricted by source. The route reaches only as far as the loopback service
itself.

Through the route the service refuses anything that takes a host path or a
repository location, with 403:

- path registration;
- submodule by URL;
- local projects.

The UI uses relative API paths, and Vite builds with `base: './'`, so it works
at `/wfe/`. The operation room's toolbar links "agdev dashboard ↗" to
`http://localhost:8093/wfe/`.

## Validation

| Command | Result |
| --- | --- |
| `npm run check` | 78 tests pass, 0 fail; tsc and build clean |
| `node --test test/gitea.test.ts` (disposable Gitea) | 8 pass |
| `npm run build && node checks/agdev.ts` | 13 browser checks pass, 0 page errors |
| agdevworld `npx tsc --noEmit`; `docker compose up --build -d web` | clean; container rebuilt |
| `curl` through `localhost:8093/wfe/` | see below |

What `test/gitea.test.ts` covers:

- **Directory mode.** Created on Gitea, pushed and registered. No token and no
  machine path in any tracked file or the local Git config. `origin` is the
  clean URL.
- **Submodule mode.**
  - devdocs is pushed before the root that records it, and the URL is
    `../pj-beta-devdocs.git`.
  - A fresh workspace obtained from Gitea reads the run.
  - `root:<commit>` reconstructs the committed stage through the gitlink.
- **A fresh directory-mode workspace** has no structure errors.
- **Partial failure.**
  - Stopped after the Gitea step, then stopped again before the push, then
    resumed.
  - The same Gitea ids are kept; nothing is created twice.
  - An unfinished creation needs `resume`.
- **Collisions and reuse.**
  - A same-named repository with content is refused and left untouched.
  - An empty one is refused unless `reuse` is given; with `reuse` it is
    adopted.
- **Shared repositories.**
  - One repository added to two projects is listed once, used by both, each
    use with its source and time.
  - Project alpha adopts an update; beta's gitlink is unchanged.
  - An occupied path is refused.
- **Registration and the dashboard.**
  - Registering an existing root registers its devdocs repository too.
  - A project without a workspace is listed, with its definition read from
    `gitea …@<commit>`.
  - An unused repository shows a valid empty usage.
  - With Gitea down: the state is `unreachable`, repositories are `unknown`,
    the definition says it was not read, and usage is unknown rather than
    unused.
  - A non-project root is refused.
- **HTTP.**
  - Through the route header, path registration and submodule-by-URL are 403.
  - Creating a shared repository and adding it work.
  - An occupied path is 409.
  - A foreign Origin is 403.

`checks/agdev.ts` drives the dashboard in a browser:

- an empty registry says so;
- a project is created through the form;
- a shared repository is created; it shows as unused;
- a second workspace is obtained from Gitea;
- "Add existing repository" proposes `study/rts`, stages a gitlink, and
  refuses the occupied path;
- the other workspace is unchanged;
- usage is shown with its source;
- Gitea is stopped: repositories show "not read" and projects stay listed.

Screenshots: `pj-agdev/.local/workflow-editor-p4/screenshots/p4-dashboard*.png`
and `p4-project-add-existing.png`.

The route, checked with `curl` against the operating service:

- `GET localhost:8093/wfe/` returns the editor's page.
- `GET localhost:8093/wfe/api/agdev` returns the dashboard JSON, with
  `gitea: ok` and the registry not created yet.
- `POST localhost:8093/wfe/api/workspaces {path}` returns 403 with the
  "not offered through agdevworld" message.
- `GET localhost:8093/wfe/api/events` streams `hello`.
- `localhost:8090/wfe/…` returns the agdevworld SPA, not the editor.
- `lsof` shows 8093 listening on 127.0.0.1 only.

## Remaining limitations

- The operating service on `:8098` was started by hand from the checkout,
  for this validation. It is not yet a launchd job, and it is not yet
  separated from the source being edited (Stage 4, step 6).
- The registry is empty: no real project is registered yet. Stage 4 does
  that.
- Execution-host availability shows `not-configured` until Stage 3 connects
  autolab.
- The p1 fixtures (`scripts/seed.ts`, `checks/step*.ts`, `e2e.ts`) still use
  local bare repositories. They are fixtures, not operation.

## Next stage entry point

Stage 3: execution by autolab alone.

- **Locking.** `server/lock.ts` is ready for per-run locking across the CLI,
  the service and the executor. The run reducer and its version check
  (`expectSeq`) already exist.
- **What autolab provides.** From the Stage 1 inspection:
  - `agag.harness` builds the `claude -p` command line, and
    `agag.execution.LiveExecution` keeps pid/start/end records.
  - The listener already serves one entry at a time from a SQLite queue.
  - There is no stop feature: a timeout kills only the direct child.
- **Plan for Stage 3.** autolab will invoke the same `wfe run` operations
  through the CLI. No state rule is duplicated in Python.
