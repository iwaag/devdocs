# Workflow editor p7/ex2 — step 6: update only the display affected by each change

Commit: agdevworld `0d4f335`.

## What changed

- **Host batches are scoped (`runView.applyBatch`).** After every batch, the host facts are compared with what was last displayed: exec view, exec error, notify view, presentation facts. The comparison leaves out `executor.at`.
  - **Heartbeat only.** If only the heartbeat time changed, a host batch renders nothing at all: no patching of the summary, pane or canvas. The new facts are still stored.
  - **Other host changes.** The summary is patched (review link, backend fact), and `RunSide.updateHost()` updates only the four sections that show host facts: Questions (presentation links inside questions), Presentations, Execution and Coordination. The pane's reading anchor is kept, as in step 5.
  - **Full batches** update the whole view as before. The patcher still writes only what differs.
- **Already in place from steps 2–3** (no further change needed):
  - Connection changes touch only the live pill and banners. Reconnection requests a full refresh through the coordinator.
  - The 30 s clock ages relative times inside their own text nodes.
  - A host read failure keeps the last facts, the run and the drafts. It is shown in Execution only, and the next cadence tick retries.
  - The canvas redraws only when its graph, states or selection change.
  - The old whole-response `JSON.stringify` comparison that rebuilt the pane is gone. The remaining comparison only decides whether host facts changed, and nothing is rebuilt either way.

## Verification

`checks/runInteraction.ts`, extended:

| Scenario | Observed |
| --- | --- |
| Heartbeat-only reads over 7 s | answer form: 0 mutations, 0 value writes, same control; **whole pane: no element added, removed or re-attributed** |
| Executor turns `unavailable` (intercepted host fact) | "Executor: unavailable — check-harness last reported 50 s ago" shown after 1.5 s |
| Host read fails (injected 500), then recovers | error shown beside the last facts; draft and run kept; error cleared and "Executor: available" shown on the next good read |
| Service stopped while composing `せつぞく`, a question asked meanwhile, service restarted, composition committed | "not live" shown; the same control stays connected and focused; the change made during the outage appears; `接続` commits into it (`compositionend:接続`) |

**What the pane-wide observation does not prove.** It would also have passed after step 3, because patching unchanged content writes nothing. The observable DOM behaviour was already correct then. Step 6 removes the work itself: a heartbeat no longer builds fresh section trees or recomputes the canvas key just to find nothing has changed. The heartbeat-only branch is exercised by the 7 s window. With the 3 s cadence, two heartbeat-only reads fall inside it.

Regression against this build:

- `npm run check`: 211 tests passed, plus type check and build.
- `runs.ts`, `docsteps.ts`, `appearance.ts`, `review.ts`, `nodes.ts`, `conversation.ts`, `exec.ts`: all passed.
