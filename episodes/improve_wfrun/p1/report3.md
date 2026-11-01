# improve_wfrun p1 step 3: published and editing states in the editor

Date: 2026-11-02. agdevworld `040f4a7`. Not yet released.

## What changed

**Project screen** (`src/projectView.ts`):

- The Workflows card has a **Published | Workspace (editing)** switch; Published is the default
  (the choice is remembered per viewer). The Published list comes from `GET
  api/workspaces/<ws>/published` and says where it was read: `<repository>@<commit> on <branch>`
  and how fresh (or "Gitea not reachable …; cached …"). Each published workflow can be read in
  place (read-only YAML with its repository and commit) and opened for editing when this workspace
  has the same id ("Not in this workspace" otherwise). The Workspace list is the previous editing
  list, with "Not published: requests cannot run it yet" for ids absent from the published revision.
  Switching keeps drafts, the request text, the creator form and selections (nothing is rebuilt
  but the card).
- **Workspace relation**: Simple shows the situation in one sentence and, when safe, **Update
  workspace**; Detail adds a table per repository (root and every pinned submodule): branch,
  workspace commit, published commit, relation with counts, uncommitted changes (counted apart),
  observation time and the fact that updating is optional.
- **Request a run**: "Runs: Published workflow | Trial of this workspace's file". The line under it
  states what will run: "definition and code from `<repo>@<commit>` (`<branch>`)", or for a trial
  "the bytes of this workspace's `devdocs/workflows/<file>` as they are now, on the published code
  `<repo>@<commit>`. Uncommitted project code is not included. The run is recorded as a trial."
- **Run view**: a Source fact for every run (`published — definition and code from …`, `trial — the
  bytes of … (sha256 …) on the published code …; uncommitted project code not included`,
  `workspace …`, or `not recorded` for older runs) and a "Trial of editing content" chip. The old
  "current definition: same/changed" comparison is relabelled "this workspace's file: …", which is
  what it compares.

**Trials** (`server/runs.ts`): `trial: {file, sha256?}` on a run request takes the workspace file's
exact bytes (refused with `definition-conflict` when they differ from the hash shown) and makes the
copy from the published source. `definition.source = {kind: 'trial', workspace, file, sha256, head,
dirty, baseline: {project, repository, branch, commit}}`. Without Gitea a trial is refused (it has
no code baseline). API: `POST …/runs` with `trial`; CLI: `wfe request <workflow> --editing`.

**Workspace relation and synchronization** (`server/workspaceSync.ts`):

- `workspaceRelation`: `same | behind | ahead | diverged | unknown` with counts, decided from
  objects a side is known to have (the access cache holds every Gitea branch; a head it lacks
  carries unpublished commits; a published commit the workspace lacks is missing work).
  `atCommitted` is not used. Dirtiness is separate. Submodules are related to the pins of the
  published revision, nested ones too; one not checked out is said so.
- `syncWorkspace` only fast-forwards: the root must be on the published branch, clean and only
  behind; each submodule whose pin moved must be clean and at its old pin (a branch is
  fast-forwarded, a detached checkout moved to the pin). Otherwise it changes nothing and returns
  the concrete reasons. Before each change it fetches (`--no-recurse-submodules`) and reads the
  state again; nothing is stashed, reset, merged or rebased. Steps are reported per repository; a
  partial update says what moved and what did not, with a recovery note, and running it again
  continues. A dry run lists the planned steps.
- `GET api/workspaces/<ws>/published` (source, workflows, relation), `GET …/published/<file>`,
  `POST …/sync {dryRun}` (409 with the result when refused); CLI `wfe workspace status [--refresh]`,
  `wfe workspace sync [--dry-run]`, with help text.

## Checks run

- `npx tsc --noEmit` clean; `npm run build` ok.
- `node --test 'test/**/*.test.ts'`: **672/672**. One intermediate failure was mine: a global
  `--trial` CLI flag collided with `wfe challenge … --trial <dir>`; renamed to `wfe request --editing`.
- New `test/workspaceSync.test.ts` (6): same; behind with an untracked file (relation and dirtiness
  apart, sync refused without change, dry run, then root and submodule fast-forwarded); ahead and
  diverged reported and refused; another branch refused with the reason; a partial update (the
  submodule's remote unreachable after the root moved) reported with recovery, then continued; a
  trial with a stale hash refused and a trial run carrying the editing bytes on the published commit
  without the workspace's uncommitted code.
- Browser check against live data, before release (`.local/improve-wfrun-p1/check/ui.mjs`,
  `run.mjs`; screenshots in `.local/improve-wfrun-p1/shots/`): a scratch service from the working
  tree on port 8199 with a copy of the registry and the production access cache. gimmickimg shows
  Published `developer/pj-gimmickimg@d4d8ea9` with `freework` and `partial-animation`, "6 commits
  behind" and Update workspace; Detail's table; Workspace view; trial wording; Simple at 390 px with
  no horizontal overflow; no page errors. A pre-p1 run (freework/run-018) shows "Source: not recorded".

## Finding outside WFE

The published `partial-animation` (gimmickimg, integrated by freework/run-018) has a validation
error: `confirms-type` — node `review` "confirms the result but is "review"; only a talk node asks
the person." So even discoverable, it cannot be requested until gimmickimg fixes the definition.
Integration did not check the validity of the definitions a run adds. Recorded for step 4 and the
final report; gimmickimg's content is left for its own agent.

## Not done / open

- Update workspace was not exercised on a live workspace (only in fixtures): live workspaces stay as
  they are in p1; the person can press it.
- Editing a published workflow that is absent from the workspace has no shortcut (the workspace must
  be updated first, or the file created).
