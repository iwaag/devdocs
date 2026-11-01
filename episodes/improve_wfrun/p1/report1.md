# improve_wfrun p1 step 1: trace and reproduction of the source mismatch

Date: 2026-11-02. Code observed: agdevworld `47a83cc` (the release in service).

## Where definitions and run bases come from today

| Consumer | Code | Definition source | Notes |
| --- | --- | --- | --- |
| `wfe projects list/show` and `GET api/projects…` | `server/access.ts` `show()`/`list()` | Gitea default branch via the access cache (`<area>/.local/access-cache/`), devdocs at the root's pin | The only committed reader. 10 s fetch TTL; `stale` fetch state on failure. |
| Editor list/detail `GET api/workspaces/<ws>/workflows[/<file>]` | `server/api.ts` → `Workspace.workflowSummaries()`/`workflowResponse()` (`server/workspace.ts`) | workspace working tree (`readdir devdocs/workflows`) | Validation's repository bindings come from `Workspace.repositories()` (workspace checkout). |
| Delegate-target lists for validation and the inspector | `server/peers.ts` `registryPeers()` | each peer workspace's working tree | |
| Run creation (person request, conversation, Zulip post, delegation child) | `server/runs.ts` `createRunInCopy()` → `sourceBundle(ws.shared)` | workspace working tree, including uncommitted bytes | Callers: `exec.ts` `ExecStore.request`, `intake.ts` `applyRow` and `applyDelegationRequest`. |
| Run base commit | `server/runspace.ts` `reserve()` → `planRepositories()` | workspace `HEAD`, or `refs/remotes/origin/<branch>` after a best-effort fetch *into the workspace* when the workspace is only behind | Chosen independently of the definition. Root target = the workspace's checked-out branch; submodule target = its checked-out branch or remote HEAD. |
| Copy materialization | `runspace.ts` `materialize()` | clone of the workspace; submodules cloned from the workspace checkout, falling back to their remote | A submodule the workspace has not checked out stays uninitialized in the copy. |
| Intake routes | `server/intake.ts` `routes()` | workspace `workflowSummaries()` + `workflowResponse()` | |
| Notifier channel provisioning | `server/notify.ts` `scan()` → `ensureTarget('channel', …)` | workspace `workflowSummaries()` | |
| Delegation dispatch: target exists/validates/accepts, cycle graph | `server/delegate.ts` `dispatch()`, `definitionGraph()` | target workspace working tree; every workspace for the cycle graph | The dispatched target records only `{project, workflow, workspace}` and a `targetDigest`; no revision. |
| Conversation target availability | `server/conversations.ts` `target()` | workspace working tree | |
| Editing writes (save, approve, create, catalog instantiate, basics) | `workspace.ts`, `basics.ts`, `catalog.ts`, `conversationWorkflows.ts` | workspace | Editing; stays on the workspace. |
| IDE `wfe run create` (run in the workspace, not a copy) | `cli/wfe.ts` → `createRun` | workspace | An explicit editing/trial path; not ordinary execution. |

Two independent choices therefore decide one run: the definition (`sourceBundle`, workspace tree)
and the code base (`planRepositories`, workspace HEAD or its upstream). Nothing records which
revision the definition came from: `definition.workflows[id].source` is only the path.

## Reproduction

Fixture: `agdevworld/workflow_editor/test/sourceMismatch.test.ts` (Gitea stand-in of bare
repositories, as in `test/access.test.ts`). Project `g` keeps devdocs as a directory, project `h`
as a submodule; each has a registered workspace and a remote. Another clone pushes to `main`, as
an integration does. All six cases pass against `47a83cc`, i.e. the following behaviour is
reproduced:

| Case | Observed |
| --- | --- |
| Stale workspace | `show()` lists `freework, partial-animation`; the editor's `workflowSummaries()` lists `freework`; `createRunInCopy(partial-animation)` refuses `no workflow "partial-animation" in devdocs/workflows`. The run-start refusal, only inferred in problems.md, is confirmed. The workspace HEAD is unchanged. |
| Mixed sources | A run of `freework` freezes the workspace's definition ("Build it.") while its copy starts at the upstream commit whose `freework.yaml` says "Published intent." (`baseFrom: upstream`). `run.json` has no revision for the definition. |
| Dirty workspace | Uncommitted edits of `freework.yaml` become the run's frozen definition; the copy's tree has the committed text. |
| Ahead workspace | An unpushed local commit becomes the run base (`baseFrom: workspace`), and the root's integration target becomes the workspace's editing branch (`editing`), not the published default branch. |
| devdocs submodule | With the root pinning a newer devdocs commit, `show('h')` lists the new workflow; the stale workspace's list and run creation do not. |
| (setup) | — |

Live check on the coordinator (read-only, nothing moved): `wfe projects show gimmickimg` reads
`d4d8ea9` and lists `freework, partial-animation`; `GET /api/workspaces/gimmickimg/workflows`
lists `freework`; the workspace is at `1482d0e`, 6 commits behind `origin/main`, `atCommitted: false`.
The workspace is deliberately left as it is: it is the live reproduction and p1 does not pull workspaces.

## Consequences for step 2

- The committed reader in `access.ts` already gives identity (repository, commit, pin, fetch
  state). Step 2 builds the published view on it: a `Workspace` whose project, workflow files and
  repositories are read from one resolved root commit, so validation and summaries need no second
  implementation.
- Run creation must resolve once and use the same commit for the definition and the copy, keep it
  in the copy manifest (so a retry with the same receipt reuses it), and record it in `run.json`.
- The copy is made from the access caches at that commit, all of its submodules at their pins,
  with integration targets the repositories' default branches.
- Delegation must record the target revision at dispatch and the intake must create the child from it.
- `definition` is stored as given by the reducer, so a `definition.source` needs no reducer change;
  `delegation.dispatch` rebuilds `target`, so a dispatched revision needs a reducer addition.
- Only archsage's workspace has a submodule not checked out (`study/study-aesthete`); devdocs is a
  directory in every live project. Initializing every pinned submodule in run copies affects only it.
