# Redesigned screens (workflow_editor p8)

## Capture conditions

- **Source and date**: real screenshots of the redesigned UI, captured on 2026-10-13 (Asia/Tokyo) from agdevworld source at p8 step 6 (the production build of the working tree, `npm run build`). The release in operation was not used and is unchanged.
- **Data**: a private service on `127.0.0.1:8210` with the representative-state fixture of `workflow_editor/checks/states.ts` (project "Sample app", 6 workflows, 14 runs, one live presentation on this host). Fixture and service live in a temporary directory and are removed afterwards. The capture script is `checks/statesShots.ts` (`node checks/states.ts --shots <dir>`).
- **Browser**: headless Chromium (Playwright 1.62.1), device scale factor 1, viewport captures (not full pages), i.e. what a person sees on opening the screen. The 1280 × 800 set repeats the main states at a smaller desktop size.
- **Synthetic part**: one capture has synthetic host facts. No executor process exists in the fixture, so for `run-working` the run's `/exec` read is answered with one alive attempt (pid 4242) and an available executor. Everything else is recorded through the server modules the CLI uses.
- **Viewer state**: the viewer's name is preset ("Sample Person"). Side sections are closed (the default) except in `run-completed-details*`.

The baseline of the same states with the previous UI is in [../image_before_states/](../image_before_states/). The concept images are in [../../design/simpler/image_suggestion/take3/](../../design/simpler/image_suggestion/take3/).

## Index

### Workflow editor (1440 × 1000)

| Image | Screen and state |
| --- | --- |
| [def-improve.png](def-improve.png) | Nothing selected, default documentation policy; the graph opens at the readable scale (72 %) with "n of 7 cards fully in view" |
| [def-node.png](def-node.png) | The Present node selected: name and description first, groups for appearance, repositories, connections and template provenance |
| [def-edge.png](def-edge.png) | A connection selected |
| [def-step.png](def-step.png) | A generated report step selected (read-only) |
| [def-branch.png](def-branch.png) | Branch and join |
| [def-manual.png](def-manual.png) | Manual policy; every node with its own icon and color |
| [def-broken.png](def-broken.png) | A file that cannot be read (YAML error): banner, raw text, editing disabled |
| [def-large.png](def-large.png) | A large graph (16 nodes, 16 reports), opened readable from its start |
| [def-intent.png](def-intent.png) | "Intent & approvals" opened |
| [def-settings.png](def-settings.png) | "Workflow settings" opened |
| [def-add-node.png](def-add-node.png) | The Add node menu: base types, the person's templates, the User nodes link |
| [def-longtext.png](def-longtext.png) | Long Japanese and English names and text, a node selected |

### Run view (1440 × 1000)

| Image | State |
| --- | --- |
| [run-plan.png](run-plan.png) | An open plan question, with its plan.md and answer form |
| [run-working.png](run-working.png) | Work in progress, an attempt open (host facts synthetic, see above) |
| [run-review.png](run-review.png) | Waiting for the result review, live presentation |
| [run-answered.png](run-answered.png) | Answer recorded, not yet taken up |
| [run-held.png](run-held.png) | Held (stopped), with the resume form |
| [run-failed.png](run-failed.png) | Verify failed; later work blocked; recovery |
| [run-cancelled.png](run-cancelled.png) | Cancelled, with a presentation still recorded as available |
| [run-accepted.png](run-accepted.png) | Result accepted, cleanup still running |
| [run-completed.png](run-completed.png) | Completed and accepted, on the adjusted definition r2 |
| [run-nodecision.png](run-nodecision.png) | Completed without a decision: the decision offered in the focus card |
| [run-branch.png](run-branch.png) | Branch and join with two open questions |
| [run-large.png](run-large.png) | The large graph, not started |
| [run-broken.png](run-broken.png) | A malformed run.json with no earlier valid reading |
| [run-longtext.png](run-longtext.png) | Long Japanese and English text, an open question in Japanese |
| [run-history.png](run-history.png) | Historical view (as committed before Verify) |
| [run-completed-details.png](run-completed-details.png) | Expanded details: Run details, every side section and a selected node |
| [run-completed-details-end.png](run-completed-details-end.png) | The same, side pane scrolled to its end |

### Smaller desktop (1280 × 800)

[def-improve-1280.png](def-improve-1280.png), [def-large-1280.png](def-large-1280.png), [run-plan-1280.png](run-plan-1280.png), [run-review-1280.png](run-review-1280.png), [run-held-1280.png](run-held-1280.png), [run-failed-1280.png](run-failed-1280.png), [run-accepted-1280.png](run-accepted-1280.png), [run-completed-1280.png](run-completed-1280.png), [run-branch-1280.png](run-branch-1280.png), [run-longtext-1280.png](run-longtext-1280.png)
