# How a project's workflows are stored and shown today

This describes the current system as of 2026-11-02 (WFE release `d7a3bd1`). It proposes no change.

## Why this was written

The Developer asked agents to add a `placeholder` node to every project's `freework` (`devdocs/episodes/fix/p1/`). The
change reached `main` in all 21 projects, yet the workflow editor still showed the old graph in all 21. The Developer
reasonably took this to mean the change had not happened, and expected the editor to show `main`.

Their observation: to a user, a project is just a project, and they should not need to know what a workspace is. The
system currently does require that knowledge, and the rest of this document shows where.

## In short

One project exists in three kinds of place, and they do not update each other automatically.

| Place | What it is | Who changes it | What uses it |
|---|---|---|---|
| **Published**: the default branch (`main`) on Gitea | The project's official state | Run integrations and direct pushes | New runs, request routing, delegation |
| **Workspace**: one registered checkout on the coordinator's host, such as `.local/agdev/pj-rts-vs-bot` | An editing area | Saves in the workflow editor, manual edits, and sync | The workflow editor; trials (`wfe request --editing`) |
| **Run copy**: one per run, under the workspace's `.local/wfe/runs/<workflow>/<run>/` | A run's own working tree, on its own run branch | The run's agent | That run only |

A run also keeps a fixed copy of the definition it started from, so later changes to `main` do not alter a run that
already exists.

## How a change travels

**A change made by a run:**

```
main --(new run takes the current commit)--> run copy --(agent edits and commits)--> run branch --(integration)--> main
```

The workspace is not on this path. It stays where it was and falls behind `main`.

**A change made in the workflow editor:**

```
workspace file --(Save)--> workspace file on disk --(someone commits and pushes)--> main
```

Save writes the file only: it never commits or pushes. Requests do not run the edit until it is on the default branch.
A trial can run the saved file before that.

**Bringing a workspace forward:** `wfe workspace sync`, or Update workspace in the project page's Workflows card. It is
optional and manual, and nothing runs it automatically. It only fast-forwards. If the workspace has uncommitted changes
or has diverged, it refuses and changes nothing, and a person has to reconcile it by hand.

## What each screen shows

| Screen | Shows | Notes |
|---|---|---|
| Workflow editor, `#/ws/<project>/wf/<file>` | **Workspace** file | Graph and Save. The URL names `ws`; the file line reads `workflows/<file> · workspace <id>`. |
| Project page, Workflows card, **Published** tab (default) | Published | **View** shows the YAML as read-only text, with no graph. |
| Project page, Workflows card, **Workspace (editing)** tab | Workspace | **Open Editor** opens the workflow editor. |
| Project page, relation table | Workspace against published | Shows counts such as behind, and the next action. |
| Run view | The run's own fixed definition | Its link to the definition opens the workflow editor, so it leads to the workspace file. |
| `wfe projects read`, `GET api/workspaces/<id>/published/<file>` | Published | Read-only. |
| `GET api/workspaces/<id>/workflows/<file>` | Workspace | What the workflow editor reads. |

The Published tab's **Edit in workspace** button also opens the workflow editor. So all these routes lead to one graph
view, and that view shows the workspace file.

## Where this breaks a user's expectation

1. **The only graph view can be out of date.** Published has text only, so a user who wants to see the graph sees the
   workspace version, whatever `main` holds.
2. **The workflow editor does not say it is behind.** No published-source or behind notice was found in its code
   (`src/workflowView.ts`). The relation appears only on the project page, one screen away.
3. **Falling behind is the normal state.** Every integration moves `main` and leaves the workspace behind. On
   2026-11-02 all 21 fan-out projects were behind, by 1 to 83 commits.
4. **Syncing can be blocked by unrelated files.** 4 of those 21 could not sync because of uncommitted changes. In
   rts-vs-bot those were five untracked `.DS_Store` files.
5. **Editing can start from an old version.** Opening the editor on a behind workspace and saving edits the old
   definition. WFE's own status message says so: "editing here starts from an older version". The save conflict check
   compares against the file on disk, not against `main`.
6. **The vocabulary is internal.** Understanding the screens requires the terms workspace, published, run copy, run
   branch and integration. The URL also uses `ws` where a user would think "project".

## Why it is built this way

From `improve_wfrun` p1 and `workflow_editor/docs/storage.md`. The docs state what the system does; the purpose given
for sync is this document's reading, not a recorded rationale.

- What a project offers and what a run follows do not depend on the state of one host's checkout: a workspace's age,
  local commits and uncommitted changes decide nothing.
- Sync refuses instead of stashing, resetting, merging or rebasing, which keeps a person's uncommitted work untouched.
- A draft can be tried (`wfe request --editing`) before it is published.

These reasons cover where runs read from. None of them requires the user to see or manage the workspace.

## Current state of the placeholder change

The state below is from 2026-11-02, about 21:15 JST.

- Published: the node is present in all 21 projects, so new runs get it.
- Workspaces: the node is absent in all 21, because no sync has been run.
- Sync would be accepted by 17 projects and is blocked in 4: rts-vs-bot, ppt, warden, protoprey.

## Sources

- `workflow_editor/docs/storage.md`: *The workspace and the published source*, *Where a run lives*
- `workflow_editor/docs/access.md`: *Published workflows*
- `workflow_editor/docs/contract.md`: *Saving and external changes*
- `workflow_editor/src/projectView.ts`, `src/workflowView.ts`, `src/runView.ts`, `src/api.ts`
- `README_DEV.md`: *Published workflows*, *Run records and working copies*
