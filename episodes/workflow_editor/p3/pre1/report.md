# workflow_editor p3/pre1 — report

## Summary

The editor is ready for the p3 trial. A person gives an IDE agent in VS Code
a braindump. The agent executes a workflow and records each step with
`wfe run`. The person follows it in the browser: what finished, what is
active, what waits for whom, questions and answers, reports, child runs and
history.

- Every validation criterion passes on synthetic evidence. This covers the
  unit suite, the run UI browser checks and the readiness rehearsal, which
  used a stand-in agent.
- The person's area `pj-agdev/.local/workflow-editor-p2/` is refreshed for
  execution. Its registry and project `rts-vs-bot` are unchanged.
- **The human-led p3 trial has not been run.** Nothing here used the
  person's actual braindump.

Step reports:

- [report1](report1.md): contract and baseline
- [report2](report2.md): run records and CLI
- [report3](report3.md): run UI and observation
- [report4](report4.md): IDE handoff
- [report5](report5.md): rehearsal

The contract is
`pj-agdev/experiments/workflow_editor/docs/runs.md`
(`ag.workflow-run.v1`).

## Review correction: report files at a commit

The review found that a historical run view showed the committed execution
state but opened report files from the working tree. The file request now carries
the displayed commit ID, and the service reads the file from that same devdocs
commit. Current run views continue to read saved working-tree files.

Missing historical files return an error rather than falling back to current
content. Artifacts outside devdocs are unavailable from a devdocs commit; the
error directs the person to the current run. File-size and path restrictions
also apply to historical reads.

Validation after the correction:

- `npm run check`: type check, 67/67 tests and production build passed.
- The HTTP regression checks two committed report versions, an uncommitted
  version, deletion from the working tree, missing historical files, invalid
  revisions and paths.
- `node checks/runs.ts --repeat 3`: 35/35 browser checks passed, including
  historical and current report contents. This is a functional regression run;
  the original 30-save measurement remains the latency evidence below.

## Starting the p3 trial

1. **Restart the editor for the area.** The service on `:8097` predates
   p3/pre1. It serves the new UI against the old API, so run pages do not
   work until the restart. Stop it (Ctrl-C in its terminal), then:

       pj-agdev/.local/workflow-editor-p2/wfe serve

   Open http://127.0.0.1:8097/.
2. In VS Code, open `pj-agdev/.local/workflow-editor-p2/` and start a **new**
   agent session there. It reads the refreshed `AGENTS.md`, which now has a
   "Runs" section.
3. Give it the execution prompt from `START.md`, with your braindump after
   it:
   > Execute workflow build-game of project rts-vs-bot with the braindump
   > below. Save my words as the run's braindump with me as the author,
   > record the run with wfe as you work, and write your plan and reports in
   > the run folder. Ask me here or as a run question when you need me.
   >
   > <your braindump>
4. **Follow the run.** In the browser, go to the project view → **Runs** →
   **Open run**, or `#/ws/rts-vs-bot/run/build-game/run-001`. The folder is
   `pj-rts-vs-bot/devdocs/build-game/runs/run-001/`. `build-game` is linear:
   study-rts → spec → confirm-spec (talk) → implement → playtest →
   human-check (talk). The two talk nodes are where the agent will wait for
   you.
5. **Answer** in the run view, or in VS Code. After a browser answer, tell
   the agent in VS Code ("I answered q1"). Recording an answer does not wake
   it.
6. **Decide on the result** in the run view (Result → Accept / Reject, with
   evidence) once you have checked it. Commit devdocs, and the project root's
   gitlink, when you want the stage kept.

`build-game` currently has no approvals. Approval is not required to run it;
the run records the approval states as facts.

## What was built

All of it is in `pj-agdev/experiments/workflow_editor/`.

| Deliverable | Where |
| --- | --- |
| Run contract: layout, inputs, snapshot, states, waits, questions, delegation, summary, acceptance, access, operation matrix, observation, future boundary | `docs/runs.md` |
| One reducer: history replay = state, readiness, blocking, summary | `shared/run.ts` |
| Run files: exclusive creation, byte-copy bundle with digests, atomic operations, delegation from the parent's bundle, history at a commit | `server/runs.ts` |
| CLI | `wfe run create/list/show/check/start/progress/wait/complete/fail/cancel/ask/answer/take-up/withdraw/delegate/attach/decide` (`cli/run.ts`); runs in `wfe status` |
| HTTP | `GET runs`, `GET runs/<wf>/<run>[?rev=]`, `GET …/file`, `POST …/ops` |
| Observation | `run.json` and run-folder listings in the 1 s poll, compared by stat; `run` / `runs` events |
| Browser | Runs card on the project view; run view on the fixed graph with node states as text and color; history view `…/at/<commit>` |
| IDE handoff | `templates/AGENTS.md` (Runs facts, access convention), `templates/START.md` (execution entry point), `wfe setup` prompts |
| Checks | `test/runs.test.ts` (16 tests, including CLI/HTTP parity); `checks/runs.ts` (33 browser checks); `checks/rehearsal-runs.ts` (the person's browser side) |

The suite passes: `npm run check` runs the type check, **67/67 tests** and
the production build.

## Operation matrix

| Operation | Person (browser) | CLI | HTTP |
| --- | --- | --- | --- |
| Create a run | — (input given in VS Code) | `run create` | — |
| List, inspect, history at a commit | Runs card, run view, `/at/<commit>` | `run list/show/check`, `status` | `GET` |
| Read reports | run view | files, `run show` | `GET …/file` |
| Answer | run view | `run answer [--from]` | `POST …/ops` |
| Decide, cancel run | run view → Result | `run decide`, `run cancel` | same |
| Start, progress, wait, complete, fail, ask, take-up, withdraw, delegate, attach | shown, not made | `run …` | same |

The asymmetries are deliberate, and the explanations are in the run view,
`START.md` and `docs/runs.md`:

- **Execution records are the executor's reports of its own work.** The
  browser shows them; anyone can still record them with
  `wfe run … --by <name>`.
- **A run starts where the braindump is given**, which is the VS Code
  conversation.
- **The same operation gives the same record** whether it comes through the
  CLI or HTTP; the parity test checks this.

## Measurements

| | Result |
| --- | --- |
| Run progress → run view (30 saves, 1 s poll) | p50 506 ms, p90 1110 ms, max 1229 ms, 0 over 2 s |
| Definition edits → editor, after the change (20) | p50 738 ms, max 1155 ms, 0 over 2 s; baseline p50 607 ms, max 878 ms |
| Idle with a run view open (10 s) | 9 Git commands, 6 file reads, 58 ms CPU |
| Per run update | about 17 ms service CPU, 32 SSE events for 30 updates |

Details are in report3 and report5. Measured on one machine.

## Limitations

- **Synthetic evidence only.** The stand-in agent was a subagent with a rule
  against reading the implementation. The person was a script plus my two
  file edits.
- **Writers take turns.** `--expect-seq` refuses outdated views, but two
  writers between a read and a rename are not detected.
- **Records are not authenticated.** Actors are declared names, and
  "running" is reported progress, not process health. No timer acts on
  anything.
- **Repository access is self-checked by the agent** (`docs/runs.md` →
  Access). There is no allowed-command schema.
- **No conditional branches, loops or retries.** A failed node can be
  resumed by hand with a reason. A completed node cannot be reopened, and
  there is no record correction except a later note or report.
- **Operations are refused on a broken record.** Hand-editing `run.json`
  breaks the replay check, and `wfe run check` names the difference. There is
  no automatic repair.
- **Run listing is not paged.** It scans every `devdocs/*/runs/*`, capped at
  500 runs.

## Notes

- The rehearsal area `.local/workflow-editor-p3-rehearsal/` is left in place
  (ignored); its service on `:8099` was stopped. Measurement data is in
  `.local/workflow-editor-measure/p3pre1-*` and
  `.local/workflow-editor-p3pre1/`.
- The p1 fixture services (`:8095`, `:5175`) were not touched. Neither was
  the person's `:8097` service, which needs the restart in step 1 above.
- Deus ex machina note: I wrote the rehearsal data and played the person; I
  did none of the agent's run work.
