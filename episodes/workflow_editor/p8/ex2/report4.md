# p8/ex2 step 4 — growing a run's work from its conversation

Step 4 of [plan.md](plan.md). It is implemented and committed in agdevworld `dfacad7` and agautolab `77c751c`
(the wfrun contract). The editor is not deployed: the installed release is still `6bdcd3b`.

## The four boundary cases

| Case (plan §2) | Result |
| --- | --- |
| A talk is answered and completed, then work is added before the final report starts | **Works with the existing operation.** After the talk only `_final-report` is ready; the adjustment rewires it after the new work |
| The added work is done, a new talk adds more; a finished talk is never reused or rewritten | Works: rewriting the completed talk is refused (`adjust-started`); the new talk has its own id |
| With node reports | Works. Work is added while `_report.talk` is pending, or after it completed, and the talk's report keeps its edge |
| **No final report: the terminal talk's completion would end the run** | **Did not work** (report 0). Fixed with the minimal general extension below |

## The extension: an adjustment at a node's completion

- **Reducer.** `run.adjust` may carry `completing: <node>`. That node must be running and must be the only
  running node; `mid-node` exempts exactly it. Every other rule still applies: identity, dependencies, the
  boundary's instructions, authorization, revisions and attempt fencing.
- **Service.** `adjustRun({completing: {node, outcome, artifacts}})` writes the adjustment and that node's
  `node.complete` as the next entry, **in one locked write**.
  - A refusal of either part refuses both, and the revision file is removed.
  - The dry run tries both on a copy, so it lists every refusal.
- **Integrity.** `replay` refuses a `completing` adjustment that is not followed directly by that completion
  (`completion-owed`). Every read and `wfe run check` replays, so no record can show work changed in the middle
  of a node.
- **Retransmission.** The completion's receipt is derived from the adjustment's (`<receipt>/complete`). A
  retransmitted adjustment is now answered from the record **before anything is written**. Before this fix it
  left an unreferenced revision file. This applies to every adjustment, not only `--completing`.
- **CLI:** `wfe run adjust <run> --definition … --reason … --source … --completing <node> --outcome …
  [--artifact …]`. `wfe run help complete` points to it.
- **What does not change:**
  - a completed run is never revived;
  - nothing is held back for a particular workflow;
  - a run whose last node adds nothing ends with its plain completion, as before.

The guide (`agautolab/agent/guides/wfrun/guide.md`) gained **"Growing the run's work from the conversation"**,
written as facts:

- the graph is where the work agreed in conversation is written down;
- each change is an adjustment with the source that says why (`request`, `answer:<q>`, `instruction:<i>`, or
  `agreement:<q>` for a change of the agreed scope);
- each concrete step needs no confirmation of its own;
- adjust at the boundary, after a dry run; then choose the next node from the updated graph;
- a later consultation is a new talk;
- `--completing` when the completion would end the run.

Its documentation paragraph now says which steps each policy generates (default, custom, manual).
`docs/adjustments.md` gained "At a node's completion" and "Growing a run's work from its conversation".

## A defect found and fixed

`wfe run revisions <run> --show r1` printed `(definition/<workflow>.yaml)` instead of the original definition.
The text was read through `readRunFile`, which serves only top-level run files. This is the documented way to
start an adjustment draft, so an agent growing a run from r1 would have got a broken draft. It has been broken
since p7. The rehearsal below found it. The CLI now reads the frozen copy from the run folder, and the help text
says `--show r<n>`.

## Verification

| Check | Result |
| --- | --- |
| `npm run check` | tsc clean; **243/243** tests |
| `test/growth.test.ts` (new, 6), workflow id `adhoc` | (1) final report on: talk → add `build` → do it → rewriting the talk refused → new talk `review` → answer → add `pretty` → final report → completed, three revisions, the history replays; (2) node reports on: added before and after `_report.talk`; (3) **no final report**: plain adjust while the talk runs is `mid-node`, adjusting after a plain last completion is `run-completed` (no revival), `--completing` dry run writes nothing, applied → `run.adjust` + `node.complete` adjacent, run in progress; a retransmission is a duplicate with no stray revision; the added work's plain completion ends the run; (4) refusals: not running yet (`state`), another node running (`mid-node`), a stale view from r1 (`revision-outdated`), an attempt that is not open, and **a failure partway** (an unacknowledged instruction at the last completion) records neither part and leaves no revision file; (5) a hand-edited history missing the completion, or with entries swapped, is not usable; (6) the same structure under the ids `freework` and `conversation-first` behaves the same |
| `checks/growth.ts` (new, CLI, 15/15) | the agent's view through `wfe`, across **three attempts** claimed from the exec store (talk asks → session ends → the answer queues the resumption → the old session is fenced → take-up → a plain adjust refused → `--completing` dry run, apply, retransmission → work → a new talk asks → a new session takes up and ends the run); `run.json` completed with r2, the adjustment directly followed by the completion, `wfe run check` usable. Transcript: [growth-transcript.md](growth-transcript.md) |
| `checks/runInteraction.ts`, `checks/simplerUi.ts`, `checks/freework.ts` | all passed (the browser's adjustment path goes through the refactored `prepareOp`/`applyOp`) |
| agautolab `tests/test_wfexec.py` | 7 passed |

## Note on the live guide

autolab reads the wfrun guide from its working tree at each claim, so the new section is live now, before the
editor that supports `--completing` and `custom` is released. I checked the risk:

- Every production workflow (`improve-game`, `create`, `study`, `research`, `improve-tools`) is `default`, with
  a final report, so none needs `--completing`.
- Nothing is queued or executing.

Step 7 releases the editor these parts refer to.
