# workflow_editor p4/ex1 — Step 3: operation boundary

Step 3 of [plan.md](plan.md): protected-target checks in the shared
service/CLI operations, with tests on isolated fixtures. Code only; the
operating area gets its `system.json` and this code at cutover (Step 4).

Commits: agdevworld `0b9d97a`; pj-agdev `9706de9` (pointer).

## What is protected, and where it is configured

`server/boundary.ts` loads the boundary for a registry:

| Protected | Source |
| --- | --- |
| the package directory this process runs (source checkout or installed release) | implicit |
| the area's `registry.json`, `gitea.json`, `system.json`, `service/`, `bin/`, `.local/` (exec store, Gitea cache) | implicit |
| further system paths | `system.json` → `protectedPaths` |
| retired repositories: Gitea id + owner/name | `system.json` → `retiredRepositories` |

`system.json` sits beside the registry, outside every managed project; no
project file or workflow access declaration reaches it. A malformed file
refuses every checked operation (it is never read as empty).

**Overlap.** Paths are compared after resolving symbolic links; for a path
that does not exist yet, the nearest existing ancestor is resolved. A
candidate is refused when it equals, lies inside, or contains a protected
path.

**Retired identity.** By Gitea id where the operation knows it (registration
of repositories and project roots, obtain, shared addition). Otherwise by the
configured owner/name: a clone's `origin`, a submodule location.

The p1 fixtures moved from the package's `.local/` to agdevworld's ignored
`.local/wfe-fixture` and `.local/wfe-checks`, because the package directory
itself is now protected.

## Where it is checked

| Entrance | Check |
| --- | --- |
| `registerWorkspace` (`wfe register`, dashboard register-by-path, end of creation and of obtain) | protected path; retired `origin` |
| `createProject` (CLI and browser) | protected destination, before anything is written |
| `obtainWorkspace` | retired root (by id); protected clone destination |
| `registerRepository`, `registerProject` (`wfe repo/project register`, Gitea creation) | retired id or owner/name; retired root key |
| `addSharedRepository` (`wfe repo add`, "Add existing repository") | retired id |
| submodule by location (`wfe add-repo`, local `POST …/submodules`) | retired location |
| workspace resolution in the API and the CLI | protected path; retired `origin` |
| `ExecStore.workspace` → `request()` and `claim()` | protected path; retired `origin`. A refused job is **dropped** with the reason; no attempt is opened. |
| dashboard | a refused registration is listed as unavailable with the refusal, not as a workspace |

The API answers 403; the CLI exits 1 with the message.

These checks protect the supported operations. They are not an OS sandbox:
an agent with shell access under the same account can still reach the files.
The README says so.

**Cutover operation.** `wfe project unregister <id>` removes the project, its
workspaces and its root repository entry. It changes nothing else: workspace
folders, Gitea repositories and shared repositories stay. It is refused while
a workspace of the project has a queued or claimed job or an open attempt.

## Tests

`test/boundary.test.ts` (6 tests, temporary areas, no Gitea):

1. **Registration refuses** each of these, and registers nothing:
   - the protected path itself;
   - a symlink with an innocent name pointing at it;
   - a Git repository inside it;
   - an ancestor repository containing a protected subfolder;
   - the running package (no `system.json` needed);
   - a repository under the area's `service/releases/`.

   An ordinary repository beside them still registers.
2. **Creation** in a protected destination is refused, and nothing is
   written.
3. **Retired repository**, refused in every role:
   - by Gitea id (under a later name) and by owner/name (under a new id);
   - as a project root, after a hand-injected repository entry;
   - as a clone's origin;
   - as a submodule, through the CLI (exit 1) and HTTP (403). No
     `.gitmodules` is written.
4. **Injected registrations.** Two are hand-written into `registry.json`: the
   protected path, and a symlink to it.
   - `request()` is refused for both.
   - A job put straight into the queue is dropped at `claim()` with a
     "refused" note, and no attempt row is written.
   - HTTP `GET …/sysws/project` returns 403, and registering the path over
     HTTP returns 403.
   - `wfe validate --workspace sysws` exits 1.
   - The dashboard lists the registration as unavailable, "refused: …".
   - The ordinary workspace still requests normally.
5. **Unregister.**
   - Refused while a job is queued for a second workspace of the project.
   - After the job is dropped: the project, both workspaces and the root
     entry are removed; the shared repository and the folders stay.
   - The removed id is then "not registered" for request, and a stale job is
     dropped at claim.
   - A second unregister is refused.
6. **Malformed `system.json`.** `loadBoundary` and registration are refused.

| Check | Result |
| --- | --- |
| `npm run check` | 93 tests pass (87 earlier + 6), 0 fail; tsc and build clean |
| `checks/runs.ts`, `checks/agdev.ts`, `checks/exec.ts` | all pass (33, 13, 18) |
