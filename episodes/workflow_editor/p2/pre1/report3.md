# workflow_editor p2/pre1 step 3 — IDE handoff

## What was built

- `templates/AGENTS.md` (tracked) is the IDE agent's guide.
  `templates/START.md` (tracked) holds the person's start steps.
- `cli/setup.ts` provides `wfe setup <area> [--port]`. It creates or
  refreshes an authoring area and never creates a project. It writes:

  | File | Rule |
  | --- | --- |
  | `AGENTS.md`, `START.md`, `CLAUDE.md` (`@AGENTS.md`) | generated with a first-line stamp; rewritten when the template changes; kept if the stamp was removed (edited by hand) |
  | `wfe` | launcher that sets the area's `WFE_REGISTRY`, `WFE_AREA`, `WFE_PORT` and runs `cli/wfe.ts` |
  | `registry.json` | created empty when missing; otherwise kept. A malformed one is reported (exit 1) and not replaced |
  | `.claude/settings.json` | created once: `Bash(<area>/wfe *)`, `Bash(./wfe *)`, `Bash(git *)`, `Edit(./**)`; kept after that |
  | `sources/` | created |

  Machine-specific paths appear only in these ignored, generated files.
- README: "Authoring area" section.

The Claude Code facts behind `CLAUDE.md` and the settings come from the
official docs (memory and permissions pages), via the claude-code-guide
agent:

- Claude Code reads `AGENTS.md` only when no `CLAUDE.md` exists. `@AGENTS.md`
  in `CLAUDE.md` imports it.
- `Edit` rules cover writes.
- The VS Code extension reads the same settings files as the CLI.

No `CLAUDE.md` or `AGENTS.md` exists in any directory above the area, so
nothing else is loaded into the agent's context.

## The guide

The head says:

- **Who the agent is:** an agent in the person's IDE.
- **Who it works with:** the person, who also uses the browser editor at the
  given URL.
- **What it has:**
  - file editing and a shell;
  - Git;
  - `wfe`, by its absolute path and `./wfe`, with what it does and how to get
    `help`/`--help`.

The body states facts:

- where projects live, and that the area itself is not a project;
- which files are the authority, with the absolute path to
  `docs/contract.md`;
- what to read for an existing project (`project.yaml`, `.gitmodules`,
  workflow files, `wfe status`);
- what the task authorizes (creating and registering projects, editing their
  files, local Git);
- that browser and agent take turns, and what each sees of the other's
  writes;
- how the service starts.

The guide has no quotas, cost rules, approval gates or security rules. Usage
details live in `wfe --help` and the file format in the contract; neither is
copied into the guide. Every `wfe` command it names exists (tested). The
reading path is this guide, then tool help when an operation is needed, then
the contract when writing definitions, then the project files for an
existing project. No episode report or implementation code is needed.

## Verification

- `test/setup.test.ts`, 2 new tests (15 in the file, 46 in the suite). They
  cover:
  - A fresh area gets every file and an empty registry, and no `pj-*`
    directory. All placeholders are filled, and the contract path in the
    guide exists.
  - Setup prints the URL, the service command and the prompt.
  - `./wfe help` from the area reports the area's registry.
  - Every `wfe <command>` the guide names answers `--help`.
  - After a `create` through the launcher, `wfe status` inside the project
    resolves the workspace.
  - Re-running setup keeps a hand-edited `AGENTS.md`, the registry and the
    project. A malformed registry is reported and left as is.
- Two runs in a scratch area: the first wrote everything, the second reported
  `unchanged`/`kept`.
- `npm run check`: tsc, 46 tests, build.

## The person's p2 area

`wfe setup pj-agdev/.local/workflow-editor-p2 --port 8097` was run. Port 8097
avoids the p1 service, which still runs on 8095 with the p1 fixture registry.
The area holds `AGENTS.md`, `CLAUDE.md`, `START.md`, `wfe`, an empty
`registry.json`, `.claude/settings.json` and an empty `sources/`, and no
project. A smoke test of `./wfe serve` from the area served the UI (HTTP 200)
and `GET /api/workspaces` (`exists: true`, no workspaces) on 8097. The
service was then stopped. `./wfe status` in the area says it is not inside a
registered workspace and how to proceed.

Start instructions (also printed by setup and written to `START.md`):

1. `pj-agdev/.local/workflow-editor-p2/wfe serve`, then open
   http://127.0.0.1:8097/.
2. Open `pj-agdev/.local/workflow-editor-p2` in VS Code and start the agent
   there.
3. Prompt: "Create a project in this authoring area for <purpose>. Clarify
   its intent and goals, establish the repositories it needs, and create one
   workflow. Make the result available in the editor for me to review and
   adjust."
