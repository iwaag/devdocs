# workflow_editor p3/pre1 — step 5: readiness rehearsal

## Setup

**All evidence in this step is synthetic.** It comes from the rehearsal area
`pj-agdev/.local/workflow-editor-p3-rehearsal/` (ignored), served by
`./wfe serve` on port 8099.

| Role | Played by |
| --- | --- |
| Person | Me, via `checks/rehearsal-runs.ts` (Playwright, browser actions only) plus two file edits a person would make in VS Code |
| IDE agent | A subagent in the area, told to use `AGENTS.md`, the docs it points to and `--help` — and **not** the implementation. Its two turns are the two "VS Code" turns. |
| Data prep (`.prep.sh`) | The state a p2 trial leaves behind. Project `notes` with `study/refs` (readonly) and `wedo/tool` (editable). Workflow `make-cli`: survey → scope (talk) → {implement, docs (delegate → write-docs)} → review (join). Workflow `write-docs`: draft → proofread. The person's approvals of `make-cli`. |

The person's prompt was the `START.md` execution prompt with a synthetic
braindump: "I keep losing small thoughts … `note add <text>` … `note list` …
maybe search some day …".

## What happened

| # | Who | Action | Evidence |
| --- | --- | --- | --- |
| 1 | Person | Waited on the project view; the run appeared and was opened from **Runs → Open run**. The watcher saw scope waiting on the person 17 s after the agent started. | `rehearsal-follow-question.png` |
| 2 | Agent | `run create make-cli --braindump … --author "Rehearsal Person" --executor "IDE Agent" --backend …`; survey start/complete with `report-survey.md`; scope start; `run ask … --to "Rehearsal Person"`; committed devdocs `d208c5e`; ended its turn asking the person. | `run.json` seq 5 |
| 3 | Person | Renamed `write-docs.yaml` → `docs-flow.yaml` and changed its draft step, in its own devdocs commit `035f0e7`. My unquoted colon also made it malformed YAML. | — |
| 4 | Person | Answered q1 in the run view as "Rehearsal Person". The card became `Answered · IDE Agent to take up`, and the banner said the answer does not notify the agent. Then told the agent "I answered q1 in the run view". | `rehearsal-answered-q1.png`, seq 6 |
| 5 | Agent | take-up q1 → complete scope → implement (`wedo/tool/note`) → start docs → `run delegate` → child `write-docs/run-001` (draft, proofread) → resume docs → complete docs (child evidence: completed at seq 5) → review (fixed one defect) → completed. Committed devdocs `e1bc0ff` and three `wedo/tool` commits. | seq 16 |
| 6 | Person | The watcher saw the run go to `Completed` live (seq 6 → 16 in 55 s). Read every report from the run view (braindump, plan, three reports, the child's request and proofread report, USAGE.md). Accepted the result in the browser with evidence. | `rehearsal-inspect-*.png`, `rehearsal-decided.png`, seq 17 |

Afterwards:

- `wfe run check` reports both runs **usable**.
- The person's page showed no page errors in any mode.

## Criteria (rehearsal + checks)

| # | Result |
| --- | --- |
| 1 provenance, collisions | Parent: `{"kind":"braindump","author":"Rehearsal Person","recordedBy":"IDE Agent"}`. Child: `{"kind":"request","requester":"IDE Agent","entrustedBy":{"kind":"run","workflow":"make-cli","run":"run-001","node":"docs"}}`, and its generated `request.md` says no person wrote it. `run create make-cli --name run-001` against the finished run → "already exists … nothing was changed", and the `run.json` md5 is unchanged (after the fix below). |
| 2 snapshot | The child was created **after** the source rename and edit, yet ran the captured `write-docs` ("Write USAGE.md …"). It shows `current source: deleted`. `make-cli/run-001` still shows its own graph, and the malformed `docs-flow.yaml` affects no run. |
| 3 branch / join | `implement` and `docs` were both ready after scope. `review` became ready only after both completed. Failure blocking a join was shown in `checks/runs.ts` (step 3, rerun now). |
| 4 talk | q1 kept its text while waiting. The browser answer was recorded, then taken up (agent), then scope completed: three entries (seq 6, 7, 8). The 5-hour-old wait check passes. |
| 5 delegation | Parent ↔ child links in both directions in the run view. Parent completion recorded the child as `completed at seq 5`. |
| 6 Git | `wfe run show make-cli/run-001 --rev d208c5e` → seq 5, "scope waiting … next move: Rehearsal Person", q1 open, bundle make-cli + write-docs. `--rev e1bc0ff` → seq 16, completed. The working tree is at seq 17 (decision). Both commits are the agent's own. |
| 7 UI robustness | `checks/runs.ts`: **33/33** (final rerun). |
| 8 latency | See below. |
| 9 person's route | Every person step above used only `START.md` and the browser (plus telling the agent in "VS Code"). The agent never read implementation code; its friction notes are below. |

## Update measurement (final)

`checks/runs.ts --repeat 30` on its own data and service:

- **Workload**: a 5-node graph plus a 1-node child. The run's history grows
  from 9 to 43 entries during the run. There are 4 runs in the project.
- **Save interval**: 0.3–1.0 s, random, so saves fall off the poll phase.
- **Poll**: definition/run poll 1 s; Git poll 3 s.

| | n | min | p50 | p90 | max | over 2 s |
| --- | --- | --- | --- | --- | --- | --- |
| progress save → run view | 30 | 193 ms | 506 ms | 1110 ms | 1229 ms | 0 |
| report file saved → listed | 1 | | 1285 ms | | | |
| burst of 5 → last shown | 1 | | 160 ms | | | |

- **Idle with the run view open (10 s)**: 9 Git commands (6 `status`, 3
  `config -f`), 6 file reads, 90 `readdir`, 213 `stat`, 58 ms CPU.
- **During the 30 updates (about 40 s)**: 32 SSE events, 148 Git commands
  (66 `rev-parse` + 34 `symbolic-ref`, i.e. the per-request availability
  check; 28 `status` and 14 `config -f` from the Git poll), 234 file reads,
  503 ms CPU in total, about 17 ms per update.
- **Definition-update regression**: `checks/measure.ts` after step 3. Workflow
  view p50 738 ms, max 1155 ms; all 21 probes pass (details in report3).

**Machine**: Apple silicon, local SSD, one machine. Read these figures as the
usability target being met here, not as a guarantee.

## Fixes made from the rehearsal

Guide additions are facts, each carrying its rehearsal reference:

| Evidence (agent's report or my checks) | Change |
| --- | --- |
| A named collision against an invalid definition was refused for the *definition*, not the collision | `createRun` reports an existing named run first, before anything else is read. |
| "Where do I put the person's chat text?", "what is my executor name?" | Guide: `--braindump -` reads stdin; `--executor` is the name you go by with the person; `--backend` when known. |
| The generated child request lacked the agreed scope; resuming a delegate node was found only in runs.md | Guide: taking up a child's result is `start` then `complete`; a parent's settled context reaches the child only through `delegate --request`. |
| A mid-run source change was noticed and silently set aside | Guide: `show` and the run view say when the current definition changed. A new run is the person's decision, and they learn from you that you noticed. |
| "Published" was unclear for a local project, so gitlinks were left modified | Guide: the project root records which devdocs/submodule commits belong together only when its gitlinks are committed. |
| An outcome said "14 lines" for a 13-line script and cannot be edited | Guide: records are never edited; a correction goes in a report or a later note. |
| `show` cut texts with no marker; "started (or resumed)" was ambiguous; the files line hid `definition/` | `show` marks cut text with `…`. `start` says started / resumed / restarted. The files line names the bundle. |
| Run view listed run-folder artifacts twice | "Recorded artifacts" lists only paths outside the run folder. |

## Left as observations

These are not changed:

- `review` fixed a defect itself because it binds `tool` as editable. Whether
  a review should send work back is a property of the workflow, not of the
  tool. There is no "reopen a completed node"; a new run is the path.
- The agent asked both in the conversation and as a run question. Both are
  allowed, and the guide does not choose.
- The agent recorded survey's start and completion back to back, because it
  had read before creating the run. The record shows a near-zero duration;
  that is truthful.
- The delegated `write-docs` was unapproved while `make-cli` was approved. The
  agent proceeded, because approval is not a gate. The run view and
  `wfe run show` show the approval states recorded at creation.

## Deus ex machina note

I wrote the rehearsal data (project, repositories, workflows, approvals) as
a stand-in for the p2 trial's outcome, and played the person. I did none of
the agent's run work.
