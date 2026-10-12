# A calmer, more approachable workflow UI

## Design direction

The everyday UI should make it easy to see what is being created, with a clear path into the underlying work whenever inspection or manual adjustment is needed.

The concern in `braindump.md` is not that the existing capabilities are unnecessary. Most work is delegated to agents, and the UI must still support precise intervention. The opportunity is to change which information receives attention by default, while preserving access to those capabilities.

This proposal is based on the design notes and the current view implementation, rather than a visual review of the running application. The Run view brings together questions, presentations, definitions, execution information, related runs, files, and history. Each serves a purpose, but giving all of them similar prominence can make opening the application feel like taking on administrative work.

## 1. Make outcomes and human decisions the everyday focus

The intended human loop described in p7 provides a useful hierarchy: express a wish, receive an outcome, judge it, and develop the next wish.

The default Run view should therefore prioritize:

1. **What is being created:** the purpose and a preview of the latest available outcome.
2. **What is happening now:** a short status such as “Building the prototype” or “Waiting for your review.”
3. **What the person can do next:** open the outcome, answer a question, or provide feedback.
4. **How the work proceeds:** a compact workflow graph showing the current position.

The current implementation places the graph in the main area and presentations in the side pane. For everyday use, consider giving presentations and decisions more space. Keep the graph easy to expand when someone wants to explore the process.

## 2. Keep a clear, dependable path to details

Confidence can come from knowing where to find information and controls. They do not all need to remain visible continuously.

A possible interaction sequence is:

- Select a node to see its explanation and current state.
- Open “Details and manual adjustments” to reach the existing inspection and editing controls.
- Open history or files from predictable, consistently placed entry points.

Keep pending questions, execution stops, and connection failures visible in the default view. A quiet screen must still distinguish healthy progress from missing or stale information.

Frame this as switching between **viewing the work** and **adjusting the work**. Both activities belong to the same user, and switching should preserve context and any unfinished input.

## 3. Build approachability through meaningful visual objects

Whitespace and softer styling can help, but creative curiosity also needs something concrete to engage with. The UI should invite thoughts such as “What could this become if I changed this part?”

Start with existing node appearance and template capabilities:

- Use purpose-oriented labels such as “Build a prototype” wherever possible, while retaining access to the underlying node type.
- Show thumbnails or previews for supported outcomes, instead of relying entirely on filenames.
- Use restrained styling for ordinary states, with stronger emphasis on the current work and items needing human attention.

These changes can make the existing workflow feel more approachable without requiring an elaborate spatial world, new navigation metaphor, or extensive animation system.

## 4. Prototype the change in the Run view first

Keep the first iteration small:

1. Give available presentations and unanswered questions prominent positions.
2. Move execution details, definitions, files, and history into explicitly opened detail areas.
3. Close the node inspector when no node is selected.
4. Reuse the existing controls and persistence behavior for manual adjustments.

Prefer changing the layout and visibility of existing sections before introducing a separate simplified editor. Separate editors would add synchronization and input-state management costs.

Preserve the current protections for user input during background updates. Opening details, selecting a node, or receiving new run information should not discard a draft or unexpectedly move focus.

## Evaluation criteria

Evaluate the prototype against three concrete questions:

- Can the person immediately understand the current situation and whether their attention is needed?
- Can they open the available outcome without searching for its file?
- Can they move from an item of interest to its details and relevant controls without losing their place?

Use these criteria to compare layouts before committing to a broader visual redesign. They make the desired balance between calm presentation and dependable control concrete enough to test.
