# Observations on the current UI

## Capture conditions

Captured on October 13, 2026 (Asia/Tokyo), from the running installed Workflow Editor at `http://127.0.0.1:8098/`. These are screenshots of the actual application and its existing project data.

Browser: headless Chromium via Playwright. Viewport: 1440 × 1000 CSS pixels, device scale factor 1. Most images use full-page capture, so their height can exceed 1000 pixels. Independently scrolling panes show only the captured scroll position; full-page capture does not expand their entire contents. `04c-run-viewport.png` records the initial Run view at the exact viewport size.

The project is “シンプルRTS（対ボット）”, the definition is `improve-game.yaml`, and the run is `improve-game/run-005`. The run was completed and accepted, and its presentation was stopped. This review therefore does not establish how a live presentation or an unanswered question appears during an active run.

Interactions were navigation, selection, display-mode switching, and scrolling. No project definitions, run records, or presentation services were changed.

## Screenshot index

| Image | Screen and state |
| --- | --- |
| [01-dashboard.png](01-dashboard.png) | Dashboard with projects, shared repositories, and creation/registration forms |
| [02-project.png](02-project.png) | RTS Project Editor with project settings, repositories, runs, and workflows |
| [03-workflow.png](03-workflow.png) | Workflow Editor in Compact mode, no node selected |
| [03b-workflow-node-selected.png](03b-workflow-node-selected.png) | Compact mode with the Present node selected |
| [03c-workflow-mini.png](03c-workflow-mini.png) | Mini mode with the same node selected |
| [04-run.png](04-run.png) | Completed Run, right pane at the top; full-page capture |
| [04b-run-details.png](04b-run-details.png) | Same Run, right pane scrolled to the bottom of History |
| [04c-run-viewport.png](04c-run-viewport.png) | Completed Run at the initial 1440 × 1000 viewport |
| [04d-run-execution.png](04d-run-execution.png) | Same Run, right pane scrolled to Execution, Coordination, and Result |
| [05-node-library.png](05-node-library.png) | User nodes library, no template selected |

## Overall opinion

The current UI already has orderly panels, consistent controls, and useful visual distinctions. The main issue is the distribution of attention: administrative information is prominent, while the graphical work occupies a large area at a small scale.

The screenshots refine the earlier advice. The screen is not uniformly crowded. There is substantial empty canvas space alongside dense text in the header and right pane. Reducing all spacing or shrinking everything would miss this imbalance.

The most promising change is to give purpose, outcomes, and the person's current task more prominence, then make detailed administration available through predictable entry points.

## Screen-specific observations

### Dashboard

The project list is readable and its panels have comfortable spacing. However, creation and registration forms occupy much of the right side even when the person is returning to existing work. Technical details such as registry location, storage mode, and workspace provenance also appear before any visual representation of what the projects produce.

Move creation and registration behind clear action buttons. Use the recovered space for project outcomes, recent activity, or attention requests. Keep repository management accessible through a labeled section. The screenshot provides evidence for the layout imbalance; thumbnails would be a proposed addition, not an existing capability shown here.

### Project Editor

The project purpose is valuable, but it is presented as an editing form, alongside ID, devdocs configuration, editable goals, and repository controls. Runs are identified primarily by workflow/run IDs and status chips. “Add a large mythological monster” would be a more recognizable entry than `improve-game/run-005`, if a suitable title can be obtained from the run's request.

For the everyday project view, show purpose as readable text, make the request field and recent outcomes prominent, and offer an explicit action to edit project settings. Preserve this detailed form layout as the adjustment surface.

### Workflow Editor

The colored cards, icons, and directional arrows are a useful foundation. Selection also has a clear blue outline, and the inspector exposes precise controls in a consistent location.

At the captured initial scale, however, Compact cards contain descriptions too small to read comfortably. The workflow occupies a narrow horizontal strip inside a much larger canvas. Meanwhile, the dark header occupies roughly 290 pixels of vertical space, and the inspector remains visible even with no selection, showing documentation policy and repository bindings.

Mini makes the node names easier to scan by removing descriptions. It leaves the large header, toolbar, and inspector in place. This confirms that reducing card content alone will not produce a calmer overall experience.

Try a smaller default header with an expandable intent section, an “Add node” entry that opens the existing palette, and a collapsible inspector. Tune the initial graph framing so the names and structure are readable. Keep a comfortable scale rather than treating “fit every node” as the only framing objective.

### Run view

Completion and acceptance are visible, which is reassuring. The UI also carefully distinguishes recorded node state from actual executor activity. That distinction should survive any redesign.

The header nevertheless displays many timestamps, backend details, counts, policy, and explanatory text. The legend introduces all node states even though the captured run is complete. The graph is small, while the right pane initially shows two already handled questions in full.

In the exact initial viewport (`04c`), the Presentations section is below the visible area. In the full-page image (`04`), it appears beneath the question history and describes a stopped presentation. A visitor can see that the process finished, but has to read or scroll to understand the delivered change.

For a completed run, lead with the request or outcome title, a short completion summary, the accepted decision, and the available result material. Collapse handled questions into conversation history. Keep the stopped presentation's availability explicit, with access to retained reports or files where supported. These captures do not justify automatically restarting it.

For active runs, the corresponding proposal is to prioritize the current question or review task. That state needs a separate visual check because it was not present during this capture.

Execution and publication details are useful for investigation, but the long attempt IDs, revision information, and publication file lists in `04d` make sense inside an explicitly opened details area. History (`04b`) is similarly valuable as a record rather than as the main reading surface.

### User nodes library

This screen shows the potential of the existing visual language: node names, distinct icons, and restrained colors are more approachable when the cards are large enough to read. The empty editor also clearly asks the person to select a template.

Long expectation text and revision metadata make the list vertically extensive. A compact gallery could show the name, icon, and a short purpose, with expectations and provenance in the selected template's details. This is a secondary improvement after the Run and project views.

## Suggested first iteration

1. Reorder the Run view around the current human task or completed outcome; collapse handled questions and technical records.
2. Rebalance the graph and details areas, reduce routine header information, and check readable initial framing.
3. Hide Dashboard creation/registration forms behind action buttons.
4. Give the everyday project view readable purpose and recognizable run titles, with settings opened explicitly.

Reuse existing sections, controls, and state-handling behavior. In particular, preserve input drafts, focus, and reading position during live updates. The first iteration can establish a better visual hierarchy before investing in a richer world metaphor or additional visual assets.

Compare the result with these images at the same viewport. Check whether the person can identify the situation, find their next action or outcome, and reach detailed controls without searching. Also capture an active review state before concluding that the redesign supports the full daily workflow.
