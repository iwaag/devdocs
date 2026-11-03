# improve_wfnode p1 step 2: run semantics and specialization

Date: 2026-11-03. Plan: [plan.md](plan.md). Contract: agdevworld `workflow_editor/docs/runs.md` (*Examples*, *Expectation assessments*),
`docs/adjustments.md` (*Specializing examples*).

## What changed

- **Compiler v6** (`shared/compile.ts`). Authored nodes keep `example`, `expectations` and `specializes`. The graph carries `groups` and
  `meanings` (each used key's words), frozen with the run. An example gets no report step, and what follows it waits for it, the final report included.
  A definition with none of the fields compiles to the same nodes and edges as v5.
- **Reducer** (`shared/run.ts`).
  - `execution.examples` lists unresolved examples. They are outside `counts`, `ready` and `blocked`, and a run with one never reads as completed.
  - `node.start` and `node.cancel` on an example are refused as `example-not-executable`. Every other state change needs a running node, so these two close the path.
  - `attempt.end` holds a run left with only examples as interrupted, cause `left-examples`. Recovery, the Observer's stall reason, `dueReason` and `causeText` read it.
  - `run.adjust` applies `example-history`, `specializes-unknown` and `example-boundary`, records `examples: [{example, group, replacedBy}]` on the revision, and reports group changes in `diff.groups`.
  - `expectation.assess` and `expectationStates` are new. `assessmentWords` gives Met, Alternative, Unmet or Unverified.
- **Status** (`shared/status.ts`). A finished run with a required expectation unmet reads `Done · …; required unmet: …`, tone warn.
- **File layer** (`server/runs.ts`). The groups pass through `graphOf`, `followedGraph` and `graphOfState`. `checkedEvidence` is factored out of `recordDone` and also serves assessments.
- **CLI and agent context.** `wfe run expectation`; `wfe run help specialize`, `help expectation`; `wfe run show` lists examples, groups, resolutions and expectations; `wfe run adjust` reports groups and resolutions. `definitionMeaningBrief` joins the attempt brief, and a continuation names unresolved examples and unsettled expectations.

## Design choices the plan left open

- **Boundary edges are a refusal, not a note.** The plan says removal must not silently erase intent and edges are the agent's to supply. Reachability on the new graph is checked, with the edge to add in the message, so chains and parallel branches both pass. The refusal is a graph error, not an approval gate.
- **No reminder for an unassessed expectation.** Reminders refuse until `--yes`. Whether the brief alone is enough is a question for the real-agent trial, and a reminder would be justified by evidence from it.
- **No specialization helper yet.** The plan adds one only if needed, and the trial decides.
- Meanings live at graph level rather than on nodes, because the file layer compares a frozen graph's authored nodes against the bundled definition across compiler versions.

## Checks that ran

- New `test/specialize.test.ts`: 12 of 12. It covers two, five and zero items from the same definition, boundary repair, provenance, in-place, history, stale revision, held attempt, the three expectation outcomes, frozen words, and a history replay.
- Full suite before the test literals were updated: 762 tests, 759 passed. All 3 failures compared a literal `ag.wfe-compile.v5`.
- After updating those literals in 4 test files, those 4 files pass. The full suite was not rerun in this step; step 5 runs it again.

## Not yet covered

- Trial and draft run creation carrying the fields (step 5), catalog (step 3), editor and run view (step 4), deployment, and any real agent.
- `--artifact` evidence is checked in the coordinator's record copy. For a remote worker this relies on the forward's file upload and has not been exercised.

## Revision

agdevworld: the commit "run semantics of examples, list groups and expectations".
