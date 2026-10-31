# Built-in workflow consultation

Added `workflow__talk` as a standard agdev node type, specializing talk to the run's workflow. Freework now starts with `talk → workflow__talk`; the consultation presents actual work nodes before substantive execution. Agreement records direction rather than granting execution permission.

## Behavior

`workflow__talk` asks the executing agent to:

- Add appropriate work nodes to the actual run graph before doing the work.
- Present that graph, its order, intended results and checks to the Developer or requesting agent, and invite agreement or corrections.
- Keep that invitation visible even when existing authorization allows work to proceed without waiting. Never claim agreement without an answer.
- Use the normal delivery route and ordinary run questions when an answer is needed, then perform substantive work on its own nodes and revise the graph as needed.

This is a built-in type, not a new entry in this installation's editable node catalog. It can be used under any node ID in any workflow. It appears in the standard Add node palette and type selector, inherits talk's visual treatment and consultation classification, and uses ordinary start, progress, ask, answer and complete operations. It is distinct from result confirmation and human review.

The built-in instructions are included in initial and continuation execution briefs while the node is pending, running or waiting, including before it becomes ready. They do not depend on reading an optional reference or on a local catalog template. The freework skeleton and the Free work creation template both include it as the second authored node; optional documentation remains separate.

## Implementation

System source: agdevworld `799c039176b54e0439cb0a583c877c41790f18a8` (implementation `e37bfda`, invitation/intake expectation `3fcb73e`, template-picker description `799c039`).

Changes cover the shared node model, catalog type validation, reminder classification, freework generation, execution briefs, editor palette and inspector, graph rendering, documentation and integration tests. No executor code or store schema change is required.

## Verification

- TypeScript compilation and production frontend/embedded-graph builds passed on the source checkout.
- The eight focused basic-workflow tests passed, including initial graph generation, delivery of consultation instructions before readiness, classification as consultation rather than substantive work, and the normal question/answer lifecycle without adding an approval gate.
- The initial full release check passed 640 of 641 tests. Its sole failure was an intake test still expecting only talk and the final report in a renamed Free work template; that expectation was corrected and its focused rerun passed. The final release check passed all 641 tests (295.12 seconds), TypeScript compilation and both production builds.
- A private-service Chrome check passed: the built-in appears in the standard palette, adding it supplies its instructions, and both editable workflow and run graphs render it without page errors. The check discarded its draft additions. A final read-only browser check against the deployed ponderer freework also passed, including the standard palette entry, with no page errors.
- Two isolated real-model trials used the normal executor with Sonnet and a local fixture project, not a production conversation. The task requested a standard-library CSV totals tool, sample data, documentation and verification without explicitly requesting node operations. Both trials added `build` and `verify` before implementation, started and completed `workflow__talk`, and completed in one attempt without retrospective records or recovery. In the second trial, graph adjustment is history seq 5, consultation completion seq 9 and build start seq 10, before file creation. Its consultation outcome invites graph corrections and explicitly does not claim agreement.

These trials establish observed behavior for two small, self-contained tasks, not universal model compliance. Neither requested a human answer or entered a human wait; the normal question/answer lifecycle is covered by integration testing. No mandatory human-response gate was added.

## Rollout

The coordinator service and installed CLI now report agdevworld `799c039176b54e0439cb0a583c877c41790f18a8`; activation restarted the service between attempts and passed its health check. The GPU execution host uses the same WFE revision, confirmed by remote Git read-back and an active worker service. Its existing managed deployment completed with ok=22, changed=3, unreachable=0 and failed=0. The executor remains `8f158702ae7b3adee923ac4b5baa818f6a9294fa`. The pj-agdev superproject pin is published as `84c2ca2`.

All 20 registered projects' freework definitions were committed and pushed. The new node follows talk; gimmickimg's existing presentation, result confirmation and cleanup chain remains after it. Original definitions are backed up, and unrelated local submodule changes were preserved. Read-back verifies the new type, its consultation provenance (no catalog template), graph edges and successful compilation in every project. Live API reads of ponderer, curator and gimmickimg return the new node without validation errors.

| Project | Published freework revision |
| --- | --- |
| rts-vs-bot | `1e04089289d3` |
| ppt | `9fefb8af1ff0` |
| agsfx | `a3a12d4d29ab` |
| jevlab | `32fbfe561cfd` |
| agops | `44df265505f9` |
| forge | `038caf019d5f` |
| archsage | `4b8594d89b79` |
| cagent | `0a7144cef9ed` |
| playprey | `51d2d9b699dd` |
| front | `36722077e15b` |
| glossary | `9fc4b6e3a0e2` |
| lowerpoly | `7109aa340544` |
| aghangout | `3c94b6ceb536` |
| ponderer | `1097b1ae9e29` |
| characters | `c254dd3d414b` |
| gimmickimg | `1482d0e2197e` |
| misc | `7f81df5673de` |
| protoprey | `8ae4d5a181b8` |
| curator | `724f5680f40e` |
| aesthete | `2da0a78e02f9` |

Operation logs, original project definitions, private trial records and browser screenshots are retained in the ignored `pj-agdev/.local/workflow-improve-p1/` directory; remote deployment records are under `pj-clusterintent/nctl/.local/workflow-improve-p1/`.

Existing runs retain their frozen definitions. A rollback to code that predates this type must first account for workflow definitions and newer run records containing it; changing only the release cannot teach the old validator the new type. Preserve those records.
