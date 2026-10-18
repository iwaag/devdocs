# workflow_editor p13 — scoped context and visibility: final report

## Result

The context agents get when a workflow runs or is authored is now one
model with explicit scope and storage, editable in the editor and the CLI,
and inspectable per attempt.

| Scope | Default (repository) | Override (local, not in Git) |
| --- | --- | --- |
| System | the executor's common contract (by hash); the authoring entry for IDE sessions | `<area>/.local/contexts/system.md` |
| Project | `project.yaml` intent, goals and the new `guides` | `…/projects/<p>/project.md` |
| Workflow | the workflow's intent and `execution.guides` | `…/projects/<p>/workflows/<w>.md` |
| Run | none | the run's own input (request, answers, instructions) — unchanged |

- **One resolver** (`server/context.ts`) for the claim, the "next attempt"
  preview, the Context view, `wfe context` and authoring. Sources carry
  scope, category, applicability, origin, provenance, delivery
  (body/reference/absent/error) and hash. A guide named by project and
  workflow is delivered once.
- **Execution**: the claim saves the bundle and its exact text as a local
  snapshot, records only ids and hashes in run.json, and hands the executor
  those bytes; autolab composes contract → context → brief and records
  whether the prompt was delivered. The context fingerprint is part of the
  session configuration: a change starts a new session with a reason naming
  each source; an unchanged one resumes.
- **Overrides**: revisioned with author, time and exact content, concurrent
  edits refused, hand edits kept, saved only for future attempts.
- **UI**: Context entrances on the dashboard, project and workflow views
  (matrix, editor, history, resolved preview with copy); the run view's
  Context section compares what an attempt was given with the next attempt.
- **Authoring**: the authoring entry names `wfe context show`; the operating
  area's entry is generated at `<area>/authoring/` on every release, beside
  the workspaces so no executing harness loads it.

## Verified

- Tests: WFE 395/395, autolab executor 22/22, browser check 23/23 in
  Chromium and WebKit (desktop and 390 px).
- A real Claude Code authoring session found `wfe context show` on its own,
  followed a project preference (node ids, description form) and committed a
  valid workflow without the preference's wording.
- Deployed trials: English by system preference; Japanese where the request
  asked for it; project and workflow instructions honoured in both; a
  changed override reached the next attempt with a recorded session
  replacement; a removal during an attempt did not change that attempt;
  snapshots survived a service restart; no override text in run records or
  pushed branches.

## Revisions

agdevworld `0311800` … `40b3903` (installed release `40b3903`), agautolab
`e66e1dd`, `4c18502`; pj-agdev pin `140237f`. Managed projects:
agsfx `85e38dd`, archsage `82d5cf2`, cagent `85c0b23`, forge `75c84e4`,
jevlab `224ffa0`, playprey `234f9ecc`, ppt `66eac3d`, agops `6e5c26e`
(the `context-check` trial workflow).

Step reports: [report1](report1.md) (inventory, resolver), [report2](report2.md)
(execution), [report3](report3.md) (UI), [report4](report4.md) (authoring),
[report5](report5.md) (deployment, demonstration).

## Inspect

On the editor's published route (`<editor>/`, see the host notes):

- `#/context/agops/context-check` — the matrix and its revision histories;
- `#/ws/agops/run/context-check/run-002` → Run details → *Context* — three
  attempts, the v2 change and the removal compared with the next attempt;
- `#/context` — choose any project; `bin/wfe context --help`.

## Limits and follow-ups

- The common contract's text is the executor's file; WFE records its hash
  only, so a snapshot shows the contract as "included, by hash".
- Authoring context is available (command, copy), never verified as
  delivered to an IDE session.
- Snapshots have no retention policy yet; deleting them makes earlier
  attempts "unavailable", never rebuilt.
- Workflow approvals in the seven updated projects are stale (records, no
  gate); re-approve when convenient.
- Runs with session policy `run` that were created before p13 start one new
  session on their next attempt ("created before scoped context").
- No Zulip editing entrance yet; it would call the same store.
- The system preference was exercised and cleared again. Setting real
  Developer-wide preferences (e.g. the English default, shared-resource
  notes that are not in `shares.json`) is the Developer's choice, in
  `#/context` or `wfe context set system`.
- Nothing stops an agent from quoting an instruction in a file it writes;
  that is its output, not the store.
