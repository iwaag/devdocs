# Workflow editor p8/ex1 — final report: Simple and Detail

**Source**: agdevworld `5d4519c`, from `8cda5c9`. **Deployment**: done 2026-10-13 at the person's request. The installed release is `5d4519c`, with `8cda5c9` as previous; see [report7](report7.md).

The step reports are [report1](report1.md) (contract), [report2](report2.md) (visual direction), [report3](report3.md) (graphs), [report4](report4.md) (run states), [report5](report5.md) (continuity) and [report6](report6.md) (validation). The captures are in [image_after/](image_after/).

## What changed

Every screen's top bar ends with **Simple | Detail** (`src/displayMode.ts`).

- **The preference**
  - It is a per-browser preference, Simple by default. It falls back to Simple when the stored value is malformed or storage is unavailable.
  - It is kept separate from the graph's Compact/Mini, which is now offered in Detail only.
  - The top bars stay in view, so the toggle is always one click away.
- **One DOM for both modes**
  - Elements are marked `.d-only` or `.s-only`. A switch rebuilds and clones nothing.
  - Two things redraw on a switch: the graph (Simple cards have their own metrics) and the project's hero.
  - A switch requested during an IME composition waits for it to end.
  - Each mode's scroll positions are remembered per screen.
  - A hidden focused control hands the keyboard to the toggle.

**Simple**, screen by screen. Simple holds the purpose, the situation and the person's next action.

- **Dashboard**:
  - project tiles with a large icon, the name, the purpose and status icons;
  - the runs waiting, and who holds the next move;
  - Open project, and one New project entry.
- **Project**:
  - the name (32 px) and purpose, and Project settings;
  - the existing request form;
  - five recent runs with large state badges;
  - workflows, each with one state.
- **Workflow**:
  - the name and purpose, and one approval state (the way into Intent & approvals);
  - validation only when it reports a problem;
  - Add node, Save, and the graph.
  - Selecting something shows its compact explanation, its everyday edit (name and description), and More details.
- **Run**:
  - the result's title with a work progress bar, and the execution and decision states;
  - the focus card, led by a situation icon, with the current human task and its one form, an executor observation that never implies health it cannot see, and the document actions;
  - the graph.
  - The side pane appears only for a selection, an open file, or a form that records under the person's name.
- **User nodes**: large template cards, the selection, Edit template and New template.

**Graph cards in Simple** show an icon, the name and a short state, plus the report icon. They sit on the stored positions, drawn closer and readable at 90 %. Selection and the person's place survive a switch, and Compact/Mini stays as it was.

**Detail** is p8's screens with their full information and controls. Contextual entries switch to Detail and reveal their target:

- Project settings, All runs, New project;
- the approval state, More details, Reports & files, All records;
- Adjust future work, the Execution, Presentations and Coordination links;
- New template and Edit template.

Data, API, execution semantics, approvals and persistence are unchanged. There are no generated summaries, new node types or new controls.

## The visible information contract

[report1](report1.md) has the full table: for each screen, what Simple shows, what a selection shows, and what is Detail's.

- **Simple excludes**: commit ids, remotes, branches, internal paths and file names, backend or model identifiers, exact timestamps, run references, routine technical explanations, the per-step outcome list, the record sections, and repeated counts.
- **The breadcrumb** names the project. On a run it ends at "Run".

## Before and after

- **Measured reduction** ([report6](report6.md)). On the same states at 1440 × 1000:
  - Small text on the first screen fell by 75–95 %: Dashboard 1930 → 106 characters, Project 1072 → 221, completed run 1203 → 145, definition 841 → 60.
  - Small text blocks fell by 55–90 %.
  - Detail is within 1–14 % of p8.
- **Graph.** On the completed run, 6 of 8 cards are readable on opening, against p8's 4 of 8.
- **The braindump's complaint.** Commit ids, remotes and the Git chips no longer appear unless the person asks for Detail. That was the complaint of "lots of small text, commit ids and remotes every time".
- **Take 3's hierarchy**: a large result title, completion and decision early, document actions, and icon cards. It was followed without Take 3's generated "What changed" text. The recorded outcomes stay in Detail, as the plan requires.

## Validation

- **Automated**, all passing:
  - `npm run check` (211 tests);
  - `displayMode.ts` (96 assertions);
  - `simplerUi.ts` (71), `runInteraction.ts` (39), `runs.ts` (33), `docsteps.ts` (35), `appearance.ts` (15), `review.ts` (22), `nodes.ts` (20), `conversation.ts` (12), `exec.ts` (20), `backend.ts` (5);
  - `agdev.ts` (13, disposable Gitea);
  - `e2e.ts` through `ui.ts` (45).
- **One stale check.** `setup.ts`, a pre1 check of a home page that p4 replaced, fails the same way on `8cda5c9`. It was not changed.
- **Manual**, at 1440 × 1000 and 1280 × 800: 96 captures reviewed, including long Japanese and English names.
- **Not verified**: native macOS Japanese IME during a switch and during background updates. Only synthetic composition events were exercised.

## Defects found along the way and fixed

These were present in the installed `8cda5c9`:

- **Project view with blocked storage.** The Project view threw when local storage was unavailable: the request form's actor name was unguarded.
- **Dashboard refresh and focus.** The dashboard's 20 s refresh took the keyboard away from whatever create or register field was being typed in. The text survived; the focus did not.
- **`docsteps.ts` race.** Its "null bounding box" was a race in the check, not in the view.

## Remaining limitations

- **Native IME.** Not verified; see above.
- **Large graphs.** In Simple, a large graph still opens partly in view. That is the plan's choice: stored positions, no automatic re-layout. Fit shows the whole graph.
- **Dashboard without Gitea.** Workspaces registered by path have no purpose to show in their tile. Projects registered through Gitea do.
- **User nodes' page.** It keeps its dark background, and its pass was minimal, as planned.
- **Simple's "Recording as" field.** It shows in the side pane whenever the focus card holds a form. A person who has not set a name meets it there, as in p8.
