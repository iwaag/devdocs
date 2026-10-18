# p13 ex1 Step 4 — verification of the journeys

## Browser journeys (`checks/contextEditors.ts`)

The check uses a private fixture: a project, a workflow, a run with one
delivered attempt, and the editor served on the LAN host name over plain
HTTP. It starts in **Simple**, as a fresh browser does. Each journey drives
the UI and then checks the stores and the run record directly.

| # | Journey | Verified |
| --- | --- | --- |
| 1 | A project-local instruction saved in the project editor, read as inherited in the workflow editor | saved "for every workflow of project p"; shown in the workflow under *Inherited instructions*; Simple throughout |
| 2 | Workflow intent and a local instruction together | separate saves and results; both drafts survive Detail ⇄ Simple, a live refresh (`project.yaml` changed on disk) and a refused navigation; the preview says unsaved edits are excluded; a conflicting local save keeps the draft, shows the saved text, then *Save mine over it* |
| 3 | Instructions for all projects from Home without a project, and from a child editor | the scope is named; the wider save creates no narrower override |
| 4 | A run's request and attempt context in Simple; a workflow preference changed from the run | request in place; attempt delivery stated; the next-attempt change appears; current attempt text, run sequence and attempt count unchanged; graph selection kept |
| 5 | An additional instruction from the run card | recorded once (i1) with its author; in no override; not listed as a setting |
| 6 | Keyboard, labels, focus, long text, narrow layouts | section toggles by keyboard; every context control labelled; focus and caret kept through a background refresh while typing; no horizontal scroll at 390 px on Home, project, workflow and run |
| — | Edge states | deleted snapshot → "unavailable"; no attempt yet; cancelled run hides the form and says why; matrix link opens the project's section |

| Engine and viewport | Result |
| --- | --- |
| Chromium, 1400 px | 54/54 |
| WebKit, 1400 px | 54/54 |
| Chromium, 390 px | 58/58 |
| WebKit, 390 px | 58/58 |

The 390 px runs add the four layout checks. The screenshots are ignored
files: `agdevworld/.local/wfe-fixture/screenshots/p13ex1-*.png` (home,
project, workflow, workflow-conflict, run).

## Release check and regressions

- `npm run check` (tsc, unit tests, vite build): **395/395**, clean. The
  server API did not change (the p13 override and context routes serve
  every new surface), so no unit test was added.
- `checks/context.ts` (p13), updated: all pass in Chromium and WebKit.
- Failures that predate this episode, identical on the pre-change baseline:
  - `simplerUi`: answer recorded; accepted while cleaning up; completed
    without a decision.
  - `displayMode`: accepted while cleaning up; completed without a
    decision; then a timeout at the decision evidence field.
  - `runInteraction`: an unchanged submitted draft is recorded and
    cleared.

  All of them are in run decision and answer flows this episode did not
  change. They are listed as a follow-up.

## Documentation

agdevworld `workflow_editor/docs/context.md` has a new section, *Where it is
edited in the browser*: the editors, their sections and routes, the save
separation, wider-scope editing, and the overview's role. Steering is now
described as an additional instruction from the run view, the
conversation, or the CLI.

Revisions: agdevworld `9ddbc0a`, `6f143d4`, `4626a35`, `f998049`.
