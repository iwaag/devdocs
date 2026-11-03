# improve_wfnode p1 step 5: verification and delivery

Date: 2026-11-03. Plan: [plan.md](plan.md). Host-specific records: `pj-agdev/.local/improve-wfnode-p1/` (ignored).

## Real-agent trials

Disposable project `wfrun-trial`, workflow `scene-world` (three `delegate` examples in a variable list, then a gallery node with `webbrowser` required, `native` and `s3` preferred). Profile sonnet.

| Run | Node | Request | Specialization | Expectations |
| --- | --- | --- | --- | --- |
| run-001 | studio | beach and harbour | 2 items: two examples in place, one omitted; before any delegation | none stated; no presentation |
| run-002 | gpupc | indoor cellar, no landscape | 0 items: all removed, list kept empty, `compose → gallery` added | none stated; presented, then stopped |
| run-004 | studio | a desert | 1 item: `specializes` naming all three examples of r1 | agent tried, the command refused it (below), passed the reminder with `--yes` and said so in its outcome |
| run-005 | gpupc | mountain and valley | 2 items: new nodes with `specializes` | webbrowser Met with presentation p1 as evidence; native and s3 Alternative, explained; labeled as the agent's report |

In every run the agent replaced the examples through `wfe run adjust` before delegating. No example was started and none was treated as a fixed three. The children were delegated from concrete nodes. This was read from each run's graph, revisions and history, not from its outcome text.

The evidence does not all point one way. run-005's "Met" says "Served over HTTP (200); not opened in a browser by me": an honest, partial check. No trial exercised the zero-item boundary refusal, because the agent stated the edge correctly the first time; the refusal is covered by tests only.

## What the trials changed

1. **Unstated expectations (run-001, run-002).** The brief and the reference note were not enough. I added the reminder `expectation-unstated` at a node's completion. This is the evidence the step 2 report was waiting for.
2. **`wfe run expectation` could not be used (run-004).** `--status` and `--commit` are global boolean flags, so the values never arrived. The agent tried several spellings, then used `--yes` and reported the failure. My tests had called the reducer and never the command. Now the status is positional, a commit is `--commit-id` as in `record-done`, and a test runs the command line.
3. **`wfe run show` crashed (run-005).** The history line read an assessment's `evidence` list as text. I found it reading the run, and it was deployed for a few minutes. Fixed, with a test.

run-005's log confirms the corrected path: its completion was refused by the reminder, the agent read `wfe run help expectation`, recorded all three, then completed.

## The plan's verification list

| # | Item | Result |
| --- | --- | --- |
| 1 | YAML and GUI round trips; digest | Passed: `nodeExpectations.test.ts`, deployed `check/edit.mjs` |
| 2 | Invalid references, malformed strengths | Passed: tests |
| 3 | Draft, apply, trial and normal runs carry the same meaning; later edits do not rewrite | Passed: `drafts.test.ts` for save and apply, deployed `wfe draft try` (run-003, cancelled before any work, read back through the API), `specialize.test.ts` for later definition edits. Later catalog edits rest on the existing snapshot tests |
| 4 | Two, five and zero items; boundary repair | Passed: tests for 2, 5, 0 from one definition. Real agents did 0, 1, 2 and 2, not 5 |
| 5 | Examples cannot be dispatched, inflate no counts, do not vanish | Passed: tests; run-004 and run-005 graphs. Dispatch is closed by `node.start`, since dispatch needs a running node |
| 6 | List order creates no edges; cross-group edges; stale revisions | Passed: tests; the deployed edit check confirms a boundary does not block edges |
| 7 | Required unmet, alternative and unverified are distinct | Passed: tests; browser check on a fixture. No real agent reported a **required** unmet |
| 8 | Browser understandability | Passed by DOM checks (26, 15, 7) and my own reading of screenshots; **not reviewed by the person** |
| 9 | Real agent specializes before delegating and reports with evidence | Passed in run-005, after the corrections above. Earlier runs are the failures that led there |

## Delivery

- WFE agdevworld `7fe52df` on the coordinator; worker gpupc pinned to `7fe52df`, executor `f067be0`. Store unchanged. Pre-activation, the first release (`e2d90b9`) replayed every run in all 22 workspaces: 665 usable, none refused. Later releases changed no reducer code.
- `npm run check` on `d50a928`: 768 of 768. The release build of `7fe52df` ran the suite again: 768 of 768. agautolab `tests/test_guide.py`: 4 of 4; the rest of its suite was not run, because only a reference note changed.
- Catalog `present` r2 → r3 (generation 12 → 13), by "Omni Agent (improve_wfnode p1)": description reworded and the expectation added, the person-visible expectation text kept. The other 8 templates compare identical.
- Demonstration: `wfrun-trial/scene-world` published, with runs 001, 002, 004, 005 kept. run-003 is a cancelled draft trial.

## Not done, and found on the way

- `wfe conversation workflow attach --commit <c>` reads an option that the global flag `commit` takes, so it looks broken the same way. Found by reading the code, not run, not changed: it is outside this episode.
- `stamp/run-015`'s agent did the work and recorded nothing. It was held `nothing-recorded`, and warden's recovery continued it without help. Existing behavior, unrelated to this change, and it cost a few minutes in run-001.
- `--artifact` evidence from a remote worker was still not exercised: run-005 cited a presentation.
