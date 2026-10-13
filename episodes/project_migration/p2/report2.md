# project_migration p2 step 2 — migration commit and registration (2026-10-13)

`trustedagent/pj-jevlab` `main` went from `87e42c6` (the person's last commit) to `7e67310` (migration) and one more commit for the approvals. History is kept; `study/`, `lab/` and `devdocs/episodes/` did not move.

## What was added

| File | Purpose |
| --- | --- |
| `project.yaml` | `ag.project.v2`, id `jevlab`, `devdocs: directory`, `areas: study, lab` |
| `agent/guide.md` | the project, its folders, what a fresh workspace lacks, pre-registration and publication rules |
| `agent/study.md`, `experiment.md`, `improve-tools.md` | what is particular to each workflow |
| `devdocs/README.md` | the project's purpose line (from `todo.md`) and a note on `episodes/` |
| `devdocs/workflows/` | `study`, `experiment`, `improve-tools`, `freework` |

`todo.md` was removed (its content is in `devdocs/README.md`; the `fruits/` line named a folder that does not exist).

## Workflows

| Workflow | Nodes |
| --- | --- |
| `study` | survey → synthesize → explain |
| `experiment` | run → analyze → report → explain (`confirms: result`); the design is the plan, agreed through the system's plan step |
| `improve-tools` | implement → verify → present → human-check → cleanup |
| `freework` | talk (system default), bound to `study` and `lab` |

All four validate with 0 errors and 0 warnings; intent and definition were approved in the name of the Developer (`iwaag`) on the Developer's instruction to proceed.

## Registration

`wfe project register trustedagent/pj-jevlab`, `wfe project obtain jevlab`. Intake needed no change: the notifier provisioned folder `wf-jevlab` and channels `wf-jevlab.{experiment,freework,improve-tools,study}` (#283–#286) on its own.

## What the checklist did not say

- A fresh workspace has none of the project's ignored `.work/` folders, and `.local/localenv.md` has to be written for the guide to be true. Both are now in the checklist.
- A `design` node cannot be given `confirms: plan`; plan agreement is the system's. A workflow that wants an agreed design says so in its guide.
- Workflow approvals are written into the workflow files, so after approving, the workspace has changes to commit and push.

## Host details

New files contain no host name, address, absolute path or token. Existing history was not touched; about 250 tracked files (logs, tables, reports) carry home paths or host names from the experiments and stay as published history.
