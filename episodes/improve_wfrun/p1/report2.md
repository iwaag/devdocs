# improve_wfrun p1 step 2: published resolution and run creation

Date: 2026-11-02. agdevworld `8d90c56` (on top of step 1's `0220144`). Not yet released.

## What changed

**One source contract** — `workflow_editor/server/published.ts`, built on the access caches of
`server/access.ts` (no second Git cache, no second registry):

- `resolvePublished(ctx, project, {rev?, fresh?})` resolves a project to an immutable root commit:
  its default branch, or a full root commit named explicitly. The result (`ag.wfe-published-source.v1`)
  names the project, root repository, branch, commit, selector (`default-branch`/`commit`), the
  repository and commit `devdocs/workflows` is read at (the root's pin when devdocs is a submodule),
  and the fetch state. With `fresh` (ordinary execution) a default branch that could not be fetched
  now is refused (`fetch-failed`, naming the cached commit and its age); a named revision is usable
  from the cache. A missing cache is a failure, never an empty list.
- `PublishedWorkspace` is a read-only `Workspace` whose `project.yaml`, workflow files, submodules,
  repositories (root at the commit, submodules at their gitlinks, areas) and devdocs mode come from
  that commit. Listing, validation and the bundle a run freezes therefore use the same code as an
  editing workspace. Writes are refused (`published-read-only`). Parsed files are memoized by
  repository and commit.
- `publishedWorkflows`, `publishedPeers` (delegate targets: projects with an allowed, available
  workspace here, their published workflows) and `publishedGraph` (the cycle check).
- Discovery reads (editor peers, intake routes, notifier targets, conversation targets) may use a
  cache up to 30 s old (`DISCOVERY_TTL_MS`), labelled with its fetch state. An integration that is
  `integrated` or `partial` calls `access.invalidate(ctx, project)`, which marks the root's and its
  submodules' cache stamps due (a file shared by every process), so the next discovery fetches.

**Run creation** (`server/runs.ts` `createRunInCopy`, `server/runspace.ts`):

- With Gitea access (`access`), the source is resolved once. The definition is read from the
  published view and the copy is reserved with the same source: `baseFrom: 'published'`, a
  `source` block in `copy.json`, root base = the commit, integration target = the root's default
  branch. The copy is cloned from the access caches; **every** pinned submodule is initialized at
  its gitlink, nested ones too, whether or not the workspace has it checked out; each targets its
  repository's default branch. A submodule that cannot be read is left uninitialized with its
  reason, and creation is refused when the definition binds it. The workspace is not fetched into,
  read for content or changed.
- The same receipt keeps the revision its copy was reserved with (a crash after the reservation
  and a later remote advance do not mix revisions). `rev` creates at a named root commit.
- Run numbers also count the run folders of the published revision and the run branches in the
  root cache, since the workspace may be behind both.
- `run.json` `definition.source` (new, portable): `{kind: 'published', project, selector,
  repository, branch, commit, definitions, fetch}`; or `{kind: 'workspace', workspace, reason,
  head, branch, dirty}` for an IDE run in the workspace or an area without Gitea. The `trial` kind
  is declared for step 3. The reducer stores `definition` as given: no reducer change.
- Host-local resources (`.local/` regular files copied into the copy) are unchanged.

**Callers moved together:** `ExecStore.request` (person/conversation requests), the intake's Zulip
requests and delegation children, `DelegateStore.dispatch` (target exists/validates/accepts on the
published default branch, fetched now; cycles over published definitions), intake `routes()`,
notifier `scan()` channel targets, editor/CLI validation peers (`registryPeers` with access), and
conversation target availability.

**Delegation revision:** `delegation.dispatch` carries `target.source = {kind: 'published',
repository, branch, commit, definitions}`; the reducer (`shared/run.ts` `targetSourceOf`) keeps it
when well-formed. The intake creates the child with `rev` = that commit, so a delayed post never
starts a different definition. Older records have no `target.source` and children of them follow
the default branch.

## Checks run

- `npx tsc --noEmit`: clean.
- `node --test 'test/**/*.test.ts'`: **666/666 pass** (log in the session scratchpad; 657 before,
  one of which was the step-1 assertion "no definition source is recorded", now inverted).
- New `test/publishedSource.test.ts` (12) — the step-1 fixture turned into the contract:

  | Case (plan table) | Test |
  | --- | --- |
  | Stale workspace | the published list and a run of `partial-animation` (absent from the workspace); refs, HEAD, status unchanged |
  | Dirty/ahead/diverged | uncommitted definition edit + unpushed commit on an `editing` branch: remote revision used, target `main`, workspace unchanged |
  | Revision race | reserved copy + remote advance + retry with the same receipt → original commit for definition and code; `rev` honoured; a new request takes the new tip for both |
  | Submodules | a study absent from the workspace and its nested module initialized at their pins; devdocs-as-submodule read at the root's pin (unpinned devdocs work is not published) |
  | Live discovery | inside the 30 s window the cache is read; `invalidate` makes the next read fetch |
  | Source failure | Gitea down: discovery labelled `stale`, execution refused `fetch-failed`, a named cached revision still works, an uncached project fails; missing and invalid workflows refused; a workspace-only workflow is not started |
  | Existing run | its frozen bundle and source are unchanged after the remote moves |
  | No Gitea | the record says `workspace` with the reason |

- New `test/publishedDelegation.test.ts` (3), on the delegation fixture with a Gitea stand-in
  (`setup({published: true})`):
  - stale target workspace (kid reset to before `make.yaml`): dispatch reaches the published
    workflow and records `target.source`; the kid's default branch then moves on; the delayed
    request post creates exactly one child **at the dispatched commit** (definition and copy); a
    repeated delivery adds no child; the kid workspace's refs are unchanged;
  - a workflow only in the target workspace is refused at dispatch, naming the revision examined;
  - a workflow pushed while the stores run: after `invalidate`, `scan()` adds its channel target as
    `pending`, intake reports "not provisioned yet"; once recorded, it is routed.

## Not done here / open

- The editor UI still lists the workspace; published/editing views, trials and workspace sync are step 3.
- Request readiness and child provenance in tool output are step 4.
- `docs/` are updated in step 5.
- A pre-p1 release replaying a record with `target.source` ignores the field (no failure); a
  pre-p1 release cannot finish a reserved-but-unfinished `baseFrom: published` copy (it clones the
  workspace, which may lack the commit). Recovery notes go into the release record (step 5).
