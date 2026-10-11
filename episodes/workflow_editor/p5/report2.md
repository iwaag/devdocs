# p5 step 2 — execution and documents

Plan sections: [plan.md](plan.md) §3 and §4, §10.2.

## What was done

Commits: agdevworld `c337437` (`workflow_editor/`) and agautolab `ffcba25`.

### Run creation and reading (`server/runs.ts`, `shared/run.ts`)

- **Frozen graph at creation.** Creating a run compiles the root workflow and freezes the result in `run.json` as `definition.effective`: compiler version, policy, nodes with their step specs, and edges. It is stored in both the `run.create` entry and the state.
  - A workflow the compiler refuses (no nodes, broken edges, unknown policy) gets a 422 before anything is written.
  - Delegate refusal is unchanged.
- **The frozen graph is what runs follow.** Readiness, replay and every operation use it. Reading checks it against the bundled definition:
  - the policy is the same;
  - the authored nodes are the same;
  - every edge joins known nodes;
  - generated ids and steps correspond;
  - when the recorded compiler is the current one, it is exactly what that compiler produces.

  A disagreement makes the record unusable (`bundle`). It is never silently recompiled. A graph frozen by another compiler version is followed as recorded.
- **Older runs.** A record without `definition.effective` was created before p5. It follows its authored graph and is not converted, so the RTS `improve-game/run-001` keeps its meaning.
- **`run.create` cross-check.** It also refuses a graph that differs from the frozen one.

### Document evidence and plan agreement

- **Document check (file layer).** `checkDocument` runs before `complete` of a documentation step, and before `ask` from `_plan`. It requires the step's document to be a regular, nonempty file of the run folder: not a symbolic link, and not resolving outside the folder. It then passes `{path, sha256, bytes}` to the reducer. Evidence a caller supplies is discarded.
- **Evidence check (reducer).** The reducer checks the evidence's shape and path, stores it on the node (`document`), and lists the document first among the outcome's artifacts. Authored nodes refuse document or agreement fields.
- **Plan agreement.**
  - A question from `_plan` records the `plan.md` it asked about.
  - `_plan` completes only if the latest question asked from `_plan` was asked about the same content and its answer was taken up. The question is recorded as `agreement`, and `--agreement` can name it.
  - A later question supersedes earlier answers. A plan edited after the question was asked is refused (`plan-not-agreed`). A withdrawal is not agreement.
- **Fencing and lifecycle.** Generated steps go through the same operations, so attempt fencing, receipts, stop/hold/resume and interruption apply to them unchanged.

### Briefs, CLI and guides

- **Attempt brief.** `documentationBrief` (in `server/exec.ts`) adds the policy and, for each running, ready or waiting step, its purpose, its document path, a note when the authored work is already complete ("only its report remains"), and the frozen instructions. Under manual policy, and for runs created before p5, it says no plan or report is asked unless a node or the person asks.
- **`wfe run show`.**
  - Policy line with separate work and documentation counts.
  - Each step's document state: not written yet / written, not yet recorded / checked (bytes, sha256).
  - For `_plan`, the agreeing question.
- **`wfe run complete`.**
  - Takes `--agreement`.
  - Reports the checked document.
  - Help text describes the steps and the short-outcome convention.
- **Agent guides.**
  - autolab's `agent/guides/wfrun/guide.md` states the policy facts, the report content expectations, the outcome length, that the final report is not acceptance, and partial reports on failure.
  - The IDE-agent template `templates/AGENTS.md` states the same policy facts.
- **Docs.** `docs/runs.md` gains "the effective graph is frozen" and "Documentation steps (p5)" (table, evidence, agreement, failure, interruption, brief, error codes).

## Verification

### Workflow editor

`npm run check` (tsc, 119 tests, vite build) passed. The 14 new tests are in `test/docsteps.test.ts`.

| Plan §11 item | Test |
| --- | --- |
| 2 — branches/joins wait for reports; completion waits for the final report; delegates still refused | `reports gate successors…`, `a join waits…`, `creation refuses … delegates` |
| 3 — the frozen graph is not changed by source edits; tampering is visible | `creation freezes the effective graph…` |
| 4 — plan written before consultation; a revision needs a new question; withdrawal ≠ agreement | `the plan is written before…`, `work waits for agreement…`, `a withdrawal alone…`, and in files: an edit after the question is refused |
| 5 — required reports exist and are nonempty; work and reporting are observable separately | `reports gate successors…`, `the file layer checks the document…` (missing, whitespace-only, symlink out of the folder, then evidence) |
| 6 — an interrupted report resumes without repeating work; a stale attempt is refused; failure fabricates nothing | `interrupted while reporting…`, `failure blocks the report and the run…` |
| 7 — manual policy has no steps and no prompts | `manual policy…` |
| Pre-p5 runs | `a run created before p5…` |

### autolab

`uv run pytest -q`: 339 passed. In `tests/test_wfexec.py`, against the real `wfe` CLI with stub agents:

- The p4 fixture declares `policy: manual`, keeping its meaning.
- A new test runs a full default-policy run through three attempts:
  1. Plan written and asked about; the process exits during the human wait.
  2. Answer recorded, then take-up, plan agreed, work done. The agent crashes while writing the report, so the run is interrupted and held.
  3. The person resumes; the report and final report are completed.
- What it asserts:
  - the question carries the `plan.md` evidence;
  - the work is completed exactly once;
  - `agreement` is `q1`;
  - no decision is recorded;
  - the checkpoint published `report.md` to the remote.

## Limitations

- **Agreement is the executor's claim.** Whether an answer is agreement is the executor's reading of the person's words. The record makes it a claim (take-up plus completing `_plan`) tied to a specific plan revision; it cannot judge the words. This is stated in `runs.md`.
- **Browser views not updated.** They still draw the authored graph and are not yet aware of generated steps. That is step 3, so the run view of a default-policy run is incomplete until then. Nothing is deployed yet.
- **Notifications not updated.** `notableEvents()` is unchanged; that is step 4.

## Next entry point

Step 3:

- policy editing and the compiled preview in the workflow view;
- the plan talk card, report icons and the final-report endpoint in the definition and run views;
- document and status inspection;
- browser checks.
