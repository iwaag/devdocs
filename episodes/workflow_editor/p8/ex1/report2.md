# Workflow editor p8/ex1 — step 2: the visual direction on real data

**Source**: agdevworld `b746e9a`. **Deployment**: none; the installed release is still `8cda5c9`.

The fixture is the p8 state fixture: real server modules and records written through `runOp`. The captures are in [image_after/step2/](image_after/step2/), with Simple and Detail at 1440 × 1000 and Simple at 1280 × 800. p8's captures of the same states are in [../image_after/](../image_after/).

## Project (Simple)

- **Hero band.** The project name is the page's heading (32 px). Below it is the purpose as text (16 px, the intent's first paragraph). The one entry, "Project settings", opens Detail on the project form with the keyboard in its first field. If project.yaml cannot be used, the project is incomplete, or settings hold unsaved edits, the hero says so in one line with an icon and points to Project settings.
- **Runs card.**
  - The existing request form (workflow, your name, your words, Request run) is unchanged. It is the same elements, so a draft survives the switch.
  - Five recent runs follow it. Each has a large state badge and one word: In progress, Waits for you, Failed, Held, Completed, Accepted, Rejected, Cancelled, Not started, or Cannot be read.
  - Each run also shows its workflow's name, the run's name, how long ago it changed, and Open.
  - "All runs (n)" opens Detail at the sixth run.
- **Workflows card.** Each workflow has an icon, its name in 15.5 px, and one state with an icon: Approved, Draft, Changed since approval, n errors, or Cannot be read. Then Open.
- **Gone from Simple**: commit ids, branches, uncommitted and unpublished chips, remotes, the sub-repository groups, the workspace with branch @ commit, the run folder path, executor and input names, node-count chips, workflow files and ids, and the creation forms.

## Completed run (Simple)

- **Title band.** It holds the run's title (29 px, the presentation's title as recorded) and the Completed and Accepted chips. A progress bar reads "6 of 6 steps done", counting authored work. Documentation steps are counted in Detail.
- **Focus card.**
  - It has a 48 px situation icon (a check on green here) next to "Completed and accepted".
  - The lead is "Accepted from Sample Person's answer". The question id, recorder and exact time are Detail's.
  - The review line keeps its availability explicit: "Night mode for the settings page — review stopped, no longer served".
  - Below are the actions: **Read final report** (primary), Original request, Reports & files (10), and All records.
  - The six-line recorded-outcome list is Detail's. Nothing replaces it; no summary is generated.
- **Graph.** Simple cards are described under "The graph" below.
- **Side pane.**
  - It appears only when something needs it: a selected step, an open file, or a form that records under the person's name (a question, a resume, a decision, Stop).
  - On a completed and decided run nothing needs it, so the graph gets the full width. At 1440 px that shows 6 of 8 cards readable at 90 % instead of 4 of 8 at 72 %.
  - Selecting a step shows its name (19 px), state, description, recorded outcome, its report with "Read it", and "More details".

## The graph (shared with the editor)

- **Simple card.** Each card shows:
  - a 34 px icon on a light tile (the node's appearance icon, else its type's);
  - the name (17 px, two lines at most);
  - the run state pill in a run;
  - the report icon in its corner.
- **Removed from the card**: the type label, the description and the repository badges. The type stays in the card's accessible name and its tooltip.
- **Geometry**: cards are 200 × 96 on the stored positions × 0.76. The readable scale is 90 %.
- **Compact/Mini** is untouched: it is Detail's setting and keeps its own key.

## Comparison with p8 and Take 3 (same viewport, 1440 × 1000)

| | p8 ([run-completed](../image_after/run-completed.png)) | Simple now ([run-completed](image_after/step2/run-completed-simple.png)) |
| --- | --- | --- |
| Text blocks above the graph | title, counts line, path/revision line, 3 chips, Run details and Open definition, focus head, lead with exact time, stopped-review line with note, "Recorded outcomes" with 6 items and a sub-heading, 4 links | title, progress, 2 chips, focus head with icon, one-line lead, review line, 4 actions |
| Side pane | Recording as with its explanation, 9 section labels | none (the graph takes the width) |
| Graph | 4 of 8 cards in view at 72 %; type labels, two-line descriptions and 9.5 px state pills | 6 of 8 cards at 90 %; icon, name and state |

Take 3's direction is followed in three ways:

- the result's title is prominent;
- the decision and completion are shown apart and early;
- the main actions are "Read final report" and "Original request", and the graph carries icons and names.

Take 3's "What changed" prose, with its test and review check marks, is not reproduced. It would be generated or invented text. The plan asks for the recorded outcomes in Detail instead.

Before propagating the styles, the first captures showed three problems:

- **The graph was framed too early.** It was framed while the side pane still took width, so it opened with 4 of 8 cards in view. The side pane is now decided before the canvas is first framed.
- **The selection card repeated itself.** Its heading "Selected node" repeated what the card shows. That heading is Detail's now.
- **The "All records" entry did not line up** with the other actions. It now sits among them.

## Not yet done (later steps)

- **Other states**: the other run states' Simple presentation, and the actor row's presence for each task, are step 4.
- **Interaction**: the editor's selection, mode-switch framing and alignment checks are step 3. Drafts, scroll and disclosure across switches are step 5.
- **Dashboard and User nodes** have their first Simple pass:
  - **Dashboard**: project tiles with a 44 px icon, the name, the purpose, status icons and the waiting runs.
  - **User nodes**: template cards with large icons.

  They are captured in step 6.
