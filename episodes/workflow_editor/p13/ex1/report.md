# workflow_editor p13 ex1 — context inside the editors: final report

## Result

Context is now part of each editor, usable in **Simple** mode, with Detail
adding provenance and history. Opening or editing it never switches the
display mode or leaves the editor.

| Where | What the Context section holds |
| --- | --- |
| Home | Instructions for all projects; no project is chosen |
| Project editor | **Shared with this project**: purpose, goals and guides (`project.yaml`, its one draft and *Save project.yaml*), with guide content. **Your instructions for this project** (local). **Inherited instructions**: all projects, editable in place |
| Workflow editor | Purpose (intent) and the workflow's guides (the file's draft and Save); local instructions; inherited project purpose, goals, guides and instructions, and all-projects instructions; *What a new run is given (saved settings)*. The graph stays on the page |
| Run view (Context card) | Original request (read only); *Instructions for this run* with *Add an instruction*; *Used for this attempt*; *Changes for the next attempt*; *Settings for future attempts* (workflow, project, all projects) in place; Detail: every attempt |
| `#/context` | The scope matrix as an overview, linking each cell to its editor's section, with a quick edit using the same editor |

- **One control per value.** Intent and goals moved into Context; *Intent &
  approvals* shows the saved text it approves.
- **Separate saves.** Definition values save with their file. Each local
  instruction has its own Save and result through the p13 override API.
  Nothing implies one atomic save of both.
- **Drafts are kept** through mode switches, live refreshes and background
  re-renders, with focus and caret. The navigation guard counts both kinds
  of draft. A conflicting save keeps the draft, shows the saved text, and
  offers *Use the saved text* or *Save mine over it*.
- **Scopes are named.** Editing a wider scope from a narrower editor says
  so and never creates a narrower override. From a run, a saved preference
  is stated to reach future attempts of all affected runs. It starts
  nothing and sends nothing to the run. Steering the run is the separate
  instruction form, the existing `instruction.receive` record, offered only
  while the run takes input.
- **Runs show the truth.** Each case is stated as it is: prepared and
  delivered, prepared but not delivered, delivery not recorded, snapshot
  unavailable, before context was recorded, or no attempt yet.

## Verified

- `checks/contextEditors.ts`, fixture journeys 1–6 and edge states,
  starting in Simple:
  - Chromium and WebKit at 1400 px: 54/54 each;
  - Chromium and WebKit at 390 px: 58/58 each.
- `checks/context.ts` (p13, updated): all pass in both engines.
  `npm run check`: 395/395.
- Deployed (`f998049`): the read-only journeys pass in both engines,
  desktop and narrow ([report5](report5.md)).

## Revisions

- agdevworld:
  - `9ddbc0a` (parts);
  - `6f143d4` (Home, project and workflow);
  - `4626a35` (run card);
  - `f998049` (docs and check) — the installed release.
- pj-agdev pin: `8bd8665`.

Step reports: [report1](report1.md) (parts), [report2](report2.md)
(editors), [report3](report3.md) (runs), [report4](report4.md)
(verification), [report5](report5.md) (deployment).

## Inspect

On the editor's published route (`<editor>/`, see the host notes):

- `#/home/context` — instructions for all projects;
- `#/ws/agops/context` — project purpose, guides, local and inherited
  instructions;
- `#/ws/agops/wf/context-check.yaml/context` — the workflow's context
  beside its graph;
- `#/ws/agops/run/context-check/run-002` → *Context* — the request, the
  attempt context and the next-attempt changes;
- `#/context/agops/context-check` — the overview.

## Limits and follow-ups

- **Deployed writes not exercised.** The permission classifier refused
  trial writes to the live service, so saving on the deployed route was
  not tried. The fixture covers it with the same build. Allowing such
  trials is the Developer's choice (a permission rule), or try a save
  yourself in Home.
- **Failures that predate this episode**, identical on the baseline, all in
  run answer and decision flows this episode did not change:
  - `simplerUi`: three failures;
  - `displayMode`: two failures and a timeout;
  - `runInteraction`: one failure.
- **Slow live host.** Under the host's current load, context and project
  reads take 3–5 s. The sections show "Loading…" until they arrive.
- **Previews show saved settings only.** They say so when edits are
  unsaved; no draft-aware preview was built.
- **No general guide editor.** Guide files are shown read-only; their
  references are edited in place, as the plan intended.
- **Run instructions from Simple** use the name typed in the card, which
  is shared with *Recording as*.
