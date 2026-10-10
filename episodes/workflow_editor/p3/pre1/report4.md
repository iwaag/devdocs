# workflow_editor p3/pre1 — step 4: IDE handoff

## Result

The authoring area can now carry workflow execution, and it starts from the
same place as before. Everything below is in
`pj-agdev/experiments/workflow_editor/`.

| Part | What changed |
| --- | --- |
| `templates/AGENTS.md` (the agent's guide) | **Head**: the agent also executes workflows when asked. The person gives the input in this conversation and follows the work in the browser; this conversation is the entrance. A new tool entry for `wfe run`, with what it tells and where its help is. **New "Runs" facts**: the run folder and its files, with a link to the run contract instead of a copy of it; braindump vs. request authorship; the fixed definition copy; `run.json` is written through `wfe run`, which enforces readiness and nothing else performs nodes; what start / ask / wait / delegate / progress / complete / fail mean to the person reading them; plans and reports are the agent's own; browser answers do not reach the agent by themselves; answer, take-up and completion are separate; acceptance belongs to the person; access self-check; saving vs. committing; waiting is not failing. The existing p2 facts keep their trial references, now written "p2/pre1 rehearsal". No procedures, quotas or required call sequences. |
| `templates/START.md` (the person's steps) | Retitled "Starting work in this authoring area". It keeps the authoring prompt and adds **Executing a workflow**: where to pick the project and workflow, the execution prompt with the braindump after it, the run's browser link pattern `<URL>#/ws/<project>/run/<workflow>/<run>`, the run folder `pj-<project>/devdocs/<workflow>/runs/<run>/`, what the run view shows, that a browser answer must be mentioned in VS Code, where to accept or reject, and `wfe run show`. It also says to restart `wfe serve` after the editor is updated. |
| `wfe setup` | Fills `{{RUN_CONTRACT}}`. Prints both prompts (authoring, and executing a workflow), the run link pattern and `wfe run help`. Still never creates a project or a run, and keeps the registry, the projects, the runs, `.claude/settings.json` and any stamp-less generated file. The existing allowlist already covers `./wfe run …`. |
| `wfe status` | Adds a **Runs** section: each run's execution state, what waits for whom, ready nodes, and the browser link when the service runs this registry. Help updated. |
| `docs/runs.md` | New **Access** section, the canonical home of the convention the guide summarizes. New **Setup and entry point** section, and a note on route parity. |

## The access convention

From `docs/runs.md` → Access; the guide states it as a fact:

- Repository bindings in the run's bundled definition are declarations. The
  executor checks them itself before changing a repository a node uses.
- Writing the run folder is the reporting capability every run has. It does
  **not** grant write access to a repository bound `readonly`. devdocs being
  `readonly` in a workflow does not forbid recording that workflow's run.
- The definition has no allowed-command schema. Restrictions given for one
  run are written into its `plan.md` and self-checked.

## Verification

`npm run check` passes: types, **67/67 tests**, and the build.

| Check | Evidence |
| --- | --- |
| Generated guide | `test/setup.test.ts`: no unfilled placeholders, and the file and run contract paths exist. Every `wfe <command>` the guide names answers `--help`. Every `wfe run <subcommand>` it names answers `--help` and says what it reads or writes. `START.md` has the execution entry point. Setup prints the execution prompt and `run help`. |
| Refresh keeps existing work | The existing setup test still passes. A refresh keeps the registry byte for byte, keeps the project, keeps a hand-edited `AGENTS.md`, and refuses a malformed registry without replacing it. |
| Both parties' routes | `test/runs.test.ts`, "operation parity". All 15 operation kinds (start, progress, ask, answer, take-up, attach, complete, wait, resume, fail, restart with reason, cancel, withdraw, delegate, decide, run cancel) run once through the CLI module and once over HTTP. The two records are identical apart from route, times and the run's name. The HTTP history is all `via: browser`. Delegation over HTTP creates the child with its parent link. The browser's own controls (answer, decide, cancel) were exercised in step 3's `checks/runs.ts`. |

## Operation matrix as delivered

| Operation | Person (browser) | Agent / anyone (CLI) | HTTP |
| --- | --- | --- | --- |
| Create a run | — gives the braindump in VS Code | `wfe run create` | — |
| List / inspect / history at a commit | Project view → Runs; run view; `…/at/<commit>` | `wfe run list`, `show [--history] [--rev]`, `check`, `wfe status` | `GET runs`, `GET runs/<wf>/<run>[?rev=]` |
| Read reports and files | Run view → Reports and files | the files, `wfe run show` | `GET …/file?path=` |
| Answer a question | Run view → Record answer | `wfe run answer [--from]` | `POST …/ops` |
| Decide on the result / cancel the run | Run view → Result | `wfe run decide`, `wfe run cancel` | same |
| Start, progress, wait, complete, fail, ask, take-up, withdraw, delegate, attach | — shown, not made | `wfe run …` | same |

The asymmetries are deliberate, and each is explained in the run view, in
`START.md` and in `docs/runs.md`:

- **Execution records are the executor's reports of its own work.** The
  browser shows them but does not make them, so a click cannot claim work. A
  person who wants to record one can use `wfe run … --by <name>`, so no
  capability is missing.
- **A run is created where the person gives the braindump**, which is the
  VS Code conversation.
- **The person sees updates live in the browser. The agent sees them in the
  files and `wfe run show`.** Answers made in the browser reach the agent
  when the person says so in VS Code. This is a difference of medium, not
  capability, the same as in p2.

## Not yet done

The person's area `pj-agdev/.local/workflow-editor-p2/` has not been
refreshed yet. Step 5 refreshes it after the rehearsal, so it gets the final
templates. Its running service on `:8097` still runs the pre-p3 code and needs
a restart by the person; `START.md` says so.
