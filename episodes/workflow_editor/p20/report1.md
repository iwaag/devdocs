# Step 1: current paths and source identity

## What was traced (baseline WFE `d7a3bd1`, source `9975c43`)

| Path | Where | Reads |
| --- | --- | --- |
| Project workflow list | `src/projectView.ts`, `GET published` | Published list and checkout list, as two tabs |
| Published definition | `togglePublished` | YAML text only, no graph |
| The only graph | `src/workflowView.ts`, `GET workflows/<file>` | The checkout's file |
| Save | `PUT workflows/<file>` → `Workspace.saveWorkflow` | Writes the checkout; no commit |
| Live updates | `src/live.ts`, `server/watch.ts` | The checkout's files and Git state only |
| Request intake, delegation | `server/runs.ts` `createPublished`, `server/delegate.ts` | Published source, fetched now |
| Trial | `createPublished` with `trial` | The checkout's bytes, on published code |
| Run's definition link | `src/runView.ts:405` | Opens the checkout's file, not the run's |
| Default-branch integration | `server/integrate.ts`, `server/exec.ts` | Pushes from a run copy, then marks access caches due |
| Workflow context | `server/contextView.ts`, `src/contextParts.ts` | Workflow and guides from the checkout |
| Every project route | `server/api.ts` `workspace()` | Refuses when the checkout is unavailable |

Deployment facts are in the ignored notes (`pj-agdev/.local/workflow-editor-p20/notes.md`).

## What was done

Agdevworld commit `5027560`, which also holds the service side of steps 3 and 4 (the code was written together; its reports are separate).

- `DefinitionDescriptor` (`shared/api.ts`): project, workflow, the actual definition repository/path/revision, the root's repository/branch/commit as a separate fact, digest, fetch state, and draft or run identity.
- `PublishedWorkspace.overlaid(file, text)`: one definition's bytes on a resolved revision. A draft's validation, approval and trial read through it, so all of them see the revision's repositories, sibling workflows and published peers.
- `published-head`: the check an open view polls, coalesced per project, at most 4 s old.
- `published`, `published-head` and `drafts` routes take the registration, not `workspace()`: a missing or stale checkout no longer blocks them. The checkout relation beside the list is a separate fact, reported as a problem when absent.
- The context matrix reads the published workspace; project-relative guides are read as committed. Before this, a published graph's context came from the old checkout.
- A project without a published source is an `absent` error, which the view states as not yet applied. There is no fallback path to the checkout.
- Exact API and error states: `workflow_editor/docs/drafts.md`.

## Checks

- `tsc` clean. `test/access.test.ts`, `context.test.ts`, `publishedSource.test.ts`, `drafts.test.ts` pass (the last new, run at this commit: 15/15).
- Not run at this step: the whole suite (run in step 6), browser checks (steps 2 onward).
