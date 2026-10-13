# p8/ex2 step 6 — integration, screens, a real agent, documents

Step 6 of [plan.md](plan.md). It is committed in agdevworld `45769ca` and agautolab `2e74777`. Nothing is
deployed yet: the installed editor is `6bdcd3b`, and step 7 releases it.

## Acceptance criteria

| Criterion (plan) | Evidence |
| --- | --- |
| **Simple**: activity visible in long work, no logs/ids/paths shown all the time; creating, keeping input and answering complete in Simple | activity line (report 2); creation (report 1); a freework run's question answered in Simple (report 5); at both sizes, the visible text of the project and of a freework run has no path, run id, attempt id or log text (below) |
| **Policy**: graphs, step counts and document requirements for every combination; old format read, saved; frozen graphs replayed | `test/custompolicy.test.ts` (8 combinations; default/manual byte-identical to v2; v2 run adjusted under v3); all 11 production runs `usable` with the new code (report 3) |
| **Changes**: two rounds, a scope change, waiting on a question, restart, conflict, retry after failure; started nodes, access and policy changes still refused | `test/growth.test.ts` (10). New in this step: **a scope change by agreement** (the proposal asked about while the work runs, applied at the boundary; q1 cannot authorize it; another text is not covered), **two adjustments from one view at once** (one applies, the other is `revision-outdated`, no stray revision), **a retry after a refusal** (an unacknowledged instruction refuses the combined operation; after the acknowledgment the same receipt applies) |
| **Generality**: another id, multi-node workflows, no final report | ids `adhoc`, `freework`, `conversation-first`; with node reports; without a final report (report 4) |
| **Placement**: new, missing, present, edited, same-name conflict, unplaceable, duplicate workspaces; re-running adds nothing | `test/basics.test.ts` (7, report 5) |
| **Activity**: duplicated events, reconnection, unreadable log, retry, another host, history | `test/activity.test.ts`, `test/activityline.test.ts`, `checks/freework.ts` (report 2). New in this step: **an unreadable log** is its own fact (`problem`), never counted and never described as "another host" |
| **Integration**: `npm run check` and the related checks; 1440 × 1000 and 1280 × 800 | below |

## A real agent ran freework

Placing freework in production is step 7. Before that, I ran the real executor and a real agent (autolab's
`wfrun` role, Claude Code) on freework in a **private area**:

- a project created by the new code, so it had freework at creation;
- a `wfe` launcher pinned to that area;
- `agautolab.wfexec --settings <area>/wfexec.toml --once`, with notify and intake off.

I answered as the person. Nothing of it is in the production catalog, Gitea or Zulip.

| Run | What the agent did | Result |
| --- | --- | --- |
| run-001 (word counter; "ask me about characters first") | session 1: started the talk, asked about characters, ended. Session 2: took the answer up, **adjusted with `--completing talk`** (source `request`), built, wrote the final report | completed, `usable`. **Finding:** it wrote the dependency as `after: [talk]` inside the node, which is not a contract field; the file kept it silently and `build` waited for nothing |
| run-002 (CSV→JSON; "show me, then I decide --pretty") | session 1: completed the talk with the adjustment, added `build` and a **new talk `confirm`** with **proper top-level edges**, built, asked in `confirm`. Session 2: took the answer up and implemented `--pretty` **inside the `confirm` talk**, then the final report | completed, `usable`. **Finding:** agreed work done inside a talk does not appear in the graph |
| run-003 (same shape: uppercase, then `--lower`) | session 1: plain completion of the talk, then an adjustment before the final report (the other boundary form), build, ask in `confirm`. Session 2: **added `add_lower` as its own node** after `confirm` (r3, with `--completing confirm`), did it, final report | completed, `usable` |

The six sessions took 5–29 turns each and cost $2.64 in all (from their logs' result events).

What changed because of the findings:

- **`after:` (run-001).** The agent read `wfe run help adjust`, which never said how dependencies are written.
  - The help now shows a node and the top-level `edges` list.
  - The result of an adjustment (CLI and run view) now notes an added node that nothing leads to, and node
    fields the contract does not know.
  - The next agent, in run-002, read the new help and wrote correct edges. That is one observation.
- **Work inside a talk (run-002).** The guide gained one sentence, with its trial reference: work a talk agrees
  on is done in a node of its own after the talk. In run-003 the agreed option became its own node. Again one
  observation.

## Fixes made in this step

- **The request form's default workflow.** It defaulted to the first workflow in the list, even one with
  validation errors. It now preselects the first workflow a run can be made of; the person's choice stays.
- **WebKit for the checks.** `WFE_BROWSER=webkit` runs a check in Safari's engine. I installed
  `webkit-2336` into the shared Playwright cache, alongside the existing browsers, with nothing removed.

## Verification

| Check | Result |
| --- | --- |
| `npm run check` | tsc clean; **255/255** tests |
| `checks/freework.ts` (Chromium) | **68/68**: creation, activity, policy, freework, and both sizes (no horizontal scroll; no paths, run ids, attempt ids or log text in Simple's visible text) |
| `checks/freework.ts` (**WebKit**) | 3 of 4 runs passed all 68. The first run, the first WebKit launch on this Mac, ended without its summary; I did not keep its output, and it did not happen again in three runs |
| `checks/growth.ts`, `simplerUi.ts`, `displayMode.ts`, `runInteraction.ts`, `docsteps.ts`, `conversation.ts`, `appearance.ts`, `review.ts`, `nodes.ts` | all passed; no fixture presentation server left listening |
| agautolab `tests/test_wfexec.py` | 7 passed (step 4) |

Screens at both sizes: [project 1440](images/project-1440x1000-simple.png),
[project 1280](images/project-1280x800-simple.png), [run 1440](images/freework-run-1440x1000-simple.png),
[run 1280](images/freework-run-1280x800-simple.png).

## Documents

- `docs/contract.md`: Documentation policy (custom), Basic workflows, `documentation-steps`, the digest.
- `docs/adjustments.md`: at a completion; growing a run's work; the setting compared.
- `docs/runs.md`: Activity.
- README: Create workflow, documentation settings, freework, the activity line, the new checks.
- The wfrun guide: growing the run's work; documentation steps per policy; work in its own node.

## Still open

- **Native macOS IME is not verified.** The creation form and the documentation control were checked with
  synthetic input only.
- **Save as reusable workflow** is out of scope, as planned. A grown definition stays in the run's revisions.
