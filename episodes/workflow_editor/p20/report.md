# Workflow editor p20: workflows without workspace management

**The goal is met on the deployed system.** The workflow graph shows what the next ordinary run uses. An agent's change reaches an open graph without synchronizing anything, and a person can edit, save, try and apply from the GUI with no Git operation. In the deployed case the placeholder node shows in 21 of 21 projects, with their checkouts still 1 to 83 commits behind.

Check it: <http://agstudio.local:8093/wfe/#/ws/ponderer/wf/freework.yaml>. For a small edit: <http://agstudio.local:8093/wfe/#/ws/wfrun-trial/wf/stamp.yaml>, then Edit, then Save and apply.

## What changed for a person

| Before | Now |
| --- | --- |
| The only graph showed one checkout's file, usually old | The graph is the published definition, refreshed within 10 s while open |
| Published was text only; two tabs to understand | One list: Applied, or Editing with changes not yet applied |
| Save wrote a file; someone else had to commit and push | Save draft is durable and changes nothing for runs; Apply publishes |
| A trial ran a checkout's file | Try draft runs an exact saved revision and records it |
| A run's definition link opened today's file | It opens what that run started with, from its record, retired or not |
| Unrelated dirty files blocked sync | Nothing about workflows depends on the checkout |

Checkout facts and Update workspace remain in Detail mode's Developer details and the CLI. Contract: `workflow_editor/docs/drafts.md`.

## Revisions

- Source and deployed: agdevworld `66bdad5`, pushed. Commits `5027560`, `d053143`, `66bdad5`; previous release `d7a3bd1`.
- agpc worker: wfe `66bdad5`, executor `a9b82f8` unchanged.
- pj-agdev pin `83035a2`, and the devdocs reports, are committed locally and **not pushed**. Both repositories were already ahead of their remotes before this work.

## Verification

| Check | Result |
| --- | --- |
| `npm run check` in the release build | 735/735, typecheck, both builds |
| `test/drafts.test.ts` (part of the above) | 17/17 |
| Browser, scratch service, real Gitea test project (`ui.mjs`, `ui2.mjs`) | 27/27, 26/26 |
| Browser, deployed, real executor (`deployed.mjs`, `deployed2.mjs`) | 6/6 before the script's own stop, then 7/7 |
| Open views for all 21 projects, 60 s, scratch service, one run | 402 checks, median 151 ms, p95 262 ms, no failures |

Scenario by scenario: [report6.md](report6.md), [report7.md](report7.md). Evidence is in `pj-agdev/.local/workflow-editor-p20/` (`notes.md`, `check/`), ignored by Git.

The browser checks found two real defects that the service tests had passed, both fixed before release: a second tab could silently overwrite a newer save, and text saved before a draft had loaded could store an empty definition ([report3.md](report3.md)).

## Migration

No workflow edit existed only in a checkout across the 21 projects, so nothing was imported. Import itself was exercised on a scratch checkout. Run evidence was not changed. No store schema changed.

## Remaining limitations

**Not verified on real data**

- **Submodule mode ran only in test fixtures**, against bare repositories. All 21 real projects keep devdocs in the root, so a real publication with a pin has never happened.
- A lost response is simulated by editing the journal, and a stale source was checked only on a scratch service with an unreachable Gitea.
- From another LAN PC, viewing was checked and editing was not.
- Going back to `d7a3bd1` is described in the notes and was not rehearsed.

**Behavior to know**

- Merging is by text lines. Edits on adjacent lines of the YAML can be reported as a conflict although they touch different nodes. It errs towards asking.
- The name on a draft and on an applied commit is what the person typed; no login exists. Anyone on the LAN who can open the editor can apply to any project's default branch, using the service's Gitea credentials. This fits the isolated home LAN, and is the one thing here worth a deliberate look if that changes.
- In submodule mode, advancing the pin also carries other commits already on that branch, as an integration does.
- Validation errors block Apply. Warnings and approvals do not.

**Outside this phase**

- The project page's metadata still reads the checkout, so that page needs one available, and `project.yaml` editing still writes the checkout. Published reads and drafts do not need it.
- Run copies are still made beside the workspace.
- HTTP routes for checkout files are kept for the read-only checkout view and for tools. The GUI no longer writes through them.
- Drafts are in no backup, and applied or discarded ones are never cleaned up.

**Left behind by the checks**

- Test project `wfrun-trial` has many check commits on `main`, and `stamp` is left renamed "Stamp p20 679983". `stamp/run-010` and `run-011` are unaccepted.
- Unrelated and left alone: the watchdog's observer `degraded` state and 21 jobs queued since 2026-10-27 ([report7.md](report7.md)).

## Reports

[Step 1](report1.md), [2](report2.md), [3](report3.md), [4](report4.md), [5](report5.md), [6](report6.md), [7](report7.md). The baseline is kept as history in [explanation.md](explanation.md).
