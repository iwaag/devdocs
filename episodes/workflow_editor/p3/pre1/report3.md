# workflow_editor p3/pre1 — step 3: run UI and observation

## Result

The browser now shows runs, and a running service follows them. A run's
graph is always drawn from its fixed snapshot. Editing the source definition
changes only the "current definition" note, never the run's graph.

| Part | Where | What |
| --- | --- | --- |
| Observation | `server/watch.ts` | Uses the same 1 s poll as definitions. Each run has two keys: `run.json`, re-read only when its stat changes, and the run folder's listing, compared by stat only, so report bodies are never read. Events are `run` (with `<workflow>/<run>`) and `runs` (a run appeared or disappeared), merged per run and tick. A (re)connection still re-reads everything. |
| Run list | `src/projectView.ts` | A **Runs** card on the project view. Each run shows its state chip, node counts, `⏳ node — next move: <holder>`, ready and failed nodes, input kind, executor, last update, parent, and decision. A run that cannot be used shows its problem. Run events refresh only this card, so a project draft and the rest of the page are untouched. |
| Run view | `src/runView.ts`, route `#/ws/<ws>/run/<wf>/<run>` | Header: execution state, "Snapshot of <wf>" with **Open current definition** and how the current source compares (same / changed / deleted / renamed), input and authorship, executor and backend, created / started / ended, last recorded update (with "… ago"), node counts. Lines for **Active**, **Waiting → next move**, **Ready**, **Failed**, **Blocked**, Cancelled and Decision. The canvas is the fixed graph, read-only; every card carries its state as **text** (Pending, Ready, Running (reported), Waiting · holder, Answered · <executor> to take up, Completed, Failed, Cancelled, Blocked) and color, with a legend. Side panel: "Recording as" name, selected node detail (times, wait and holder, outcome, child evidence, failure, notes, its questions), questions with answer forms, related runs (parent, children, predecessor, entrusting run), reports and files (readable in place), result (Accept / Reject with evidence, Cancel run), and full history. |
| History view | `#/ws/<ws>/run/<wf>/<run>/at/<devdocs commit>` | The record and bundle as committed, read-only, labeled with the commit. |
| Canvas | `src/canvas.ts` | An optional per-node run status. Binding badges resolve against the repositories recorded at run creation. |
| Definition editor | `src/workflowView.ts` | Ignores `run` and `runs` events entirely. |

The view says plainly, in the header, that "running" means the executor
recorded a start. It also says that neither the time since the last update
nor the live view tells whether an agent is still working; the VS Code
conversation does. Under every answer form it says the answer is stored in
`run.json` and does not start or wake the IDE agent. After an answer, the
confirmation tells the person to continue in VS Code.

Stale and obsolete data:

- Loads are serialized, and a disposed view ignores late responses.
- Every operation sends `expectSeq`. A refused submission keeps the typed
  text and reloads the view.
- Text being typed (answers, evidence) keeps its value and focus across
  refreshes.
- A malformed or inconsistent record keeps the last valid reading on screen,
  read-only, with the error. Operations are disabled until the file is fixed.

## Checks

`checks/runs.ts` builds a temporary project and its own service (production
build, port 8196, bench counters). It records operations through the module
the CLI uses and drives Chromium. All **33 checks pass**:

| Plan criterion | Checked |
| --- | --- |
| 3 branches / join / failure | Labels `survey Completed, build Running (reported), ask Waiting · Check Person, handoff Waiting · run sub/run-001, join Pending`. The summary lists both waiting branches and the active one. A failed build shows `Failed`, and the join shows `Blocked`. The failed branch and the waiting branch are visible together. |
| 4 talk question | A browser answer is recorded with name and route `browser`. The node still waits and the answer is not taken up. The card shows `Answered · Omni Agent to take up`. Take-up and completion follow as separate steps. A 5-hour-old wait stays waiting and shows "5 h ago". |
| 5 delegation | Child → parent and parent → child links work. The child's input reads "request.md — by Omni Agent". |
| 6 history | The `/at/<commit>` view shows the committed stage (seq 38: build running, ask completed) while the working tree has moved on. |
| 7 UI robustness | Burst of 5 ends on the last seq. A typed answer survives refreshes with focus. An outdated answer is refused, `run.json` is unchanged, and the text is kept. Malformed `run.json`: error in 0.96 s, last graph kept, operations disabled, file not overwritten, recovery in 1.0 s. Report saved into the folder appears in 1.3 s and is readable. Outage: "not live" and a banner; the change made during it appears 1.1 s after restart. Fast switch R1 → R2 with R1's response delayed ends on R2. An unsaved definition draft survives 4 run updates with no "changed on disk" banner. Editing the source keeps the snapshot and shows "changed since this snapshot". |
| page errors | 0. The 4 network errors are the deliberate aborts, outage and 409. |

Screenshots are in `pj-agdev/.local/workflow-editor-p3pre1/screenshots/`
(`p3-runs-project`, `p3-run-view`, `p3-run-answered`,
`p3-run-failed-branch`, `p3-run-source-changed`, `p3-run-malformed`,
`p3-run-at-commit`).

The unit suite now has 66 tests and all pass (`npm run check`: types, tests,
build). The new watcher test checks that a `run.json` change, a new report
file and a new run produce separate events, and that an idle tick reports
nothing.

## Measurements

These are preliminary; step 5 repeats them as part of the rehearsal.

**Run progress → run view.** Twenty `node.progress` saves, spaced 0.3–1.0 s
apart off the poll phase. The poll is 1 s and the run has 25–45 history
entries.

| n | min | p50 | p90 | max | over 2 s |
| --- | --- | --- | --- | --- | --- |
| 20 | 158 ms | 545 ms | 1118 ms | 1127 ms | 0 |

- **Idle**, with the run view open, over 10 s: 9 Git commands (6 status,
  3 `config -f`), 6 file reads, 90 `readdir`, 213 `stat`, 51 ms CPU.
- **During the 20 updates**, about 30 s: 20 SSE events, 90 Git commands,
  160 file reads, 363 ms CPU in total, so about 18 ms CPU per update. The Git
  commands are 2 `rev-parse` and 1 `symbolic-ref` per request, which is the
  existing workspace availability check on every API request. The rest is the
  3 s Git-state poll. Nothing run-specific runs Git.

**Definitions (regression check).** `checks/measure.ts --skip-large`, before
and after this step:

| | Baseline (step 1) | After step 3 |
| --- | --- | --- |
| workflow view p50 / p90 / max | 607 / 864 / 878 ms | 738 / 1133 / 1155 ms |
| project view p50 / p90 / max | 630 / 1112 / 1112 ms | 654 / 884 / 884 ms |
| over 2 s | 0 | 0 |
| update probes | 21/21 | 21/21 |
| idle `readdir` per 15 s (workflow view) | 15 | 45 (the run scan: devdocs listing + `runs/` per directory) |

The latency differences are within the spread of the 1 s poll phase. One
save lands just after a tick and another just before it.

Raw data:

- `.local/workflow-editor-measure/p3pre1-step3.{json,txt}`
- `.local/workflow-editor-p3pre1/checks-runs.json`

## Notes

- The run view's canvas is the editor's own canvas in read-only mode, with
  cards and edges unchanged. Run state is added as a label line and a colored
  ring.
- `run.json` itself is listed under "Reports and files", so it can be read in
  the browser. Editing happens in VS Code.
