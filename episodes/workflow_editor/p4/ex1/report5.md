# workflow_editor p4/ex1 — Step 5: acceptance and documentation

Step 5 of [plan.md](plan.md).

## Existing product behaviour, against the integrated code

| Plan §6 item | Evidence |
| --- | --- |
| Contract tests, type check and build from the new location | `npm run check` in `agdevworld/workflow_editor`: 93 pass, 0 fail. The same check passed inside each installed release (`35ab47a`, `2678c9f`, `6fe2b61`), as `wfe-release.sh build` requires. |
| agdevworld frontend build | agdevworld `npm run check` (build + package check) passes, including from a clean clone (report2). |
| autolab executor tests against the new CLI location | `tests/test_wfexec.py`: 3 passed with the default `WFE_PACKAGE` (source) and 3 passed with `WFE_PACKAGE=.local/agdev/service/current` (installed release). |
| Browser checks: dashboard, project editing, execution | `checks/runs.ts` 33, `checks/agdev.ts` 13 and `checks/exec.ts` 18, all pass, after the boundary change too. |
| Create/read/history in both devdocs modes | `test/runs.test.ts`: creation and history in directory and submodule mode, `root:<commit>` through the gitlink. |
| Gitea publication and retry | `test/exec.test.ts`: publication checkpoints and the retry of a failed root push. |
| Shared-repository listing and addition | `test/gitea.test.ts` (disposable Gitea), `checks/agdev.ts`. |
| Delegate refusal | `test/runs.test.ts`: preflight, reducer, CLI and HTTP. |
| Question / answer / resumption; stop / restart | `test/exec.test.ts`, `checks/exec.ts`, `test_wfexec.py`. |
| Unavailable information vs. an empty catalog | `test/gitea.test.ts` (Gitea down: "unknown", not "unused"). A refused registration is listed as "refused", not dropped silently (`test/boundary.test.ts`). |
| RTS through agdevworld after restart | The dashboard at `localhost:8093/wfe/` lists the RTS project and `developer/study-rts` (used by `rts-vs-bot (study/study-rts)`). The editor rows stay absent after two restarts of the service and the executor (report4). |

Run history, answers and publication state for RTS: **there are none yet.**
No real run has been made in the RTS workspace since the p4 migration
(`devdocs/runs/` is absent, and the exec store held no job or attempt). The
p3 MVP run (`build-game/run-001`) is kept as knowledge in
`devdocs/notes/p3-mvp/` under the p4 contract, not as a run record.

## Real RTS acceptance: done (2026-10-11)

The person requested one improvement through agdevworld, and it completed
there. The Omni Agent wrote no request, answer or acceptance.

**Request.** `improve-game/run-001` in workspace `rts-vs-bot`, workflow
`improve-game`, from the browser. Braindump by iwaag: 「兵士ユニットを一種類追加してください。詳細は任せます。」

**Attempts.** Three, all by autolab through `com.agdev.agautolab-wfexec`
(`claude_code / anthropic/claude-sonnet-5`). Each ended its process at a
human wait or at completion.

| Attempt | Reason | Window (UTC) | Ended at |
| --- | --- | --- | --- |
| `amv3cmld3` | start | 04:54:30–04:55:14 | asked q1 (confirm-scope) |
| `amv3cofck` | resume after answer | 04:55:56–05:00:26 | implement, verify; asked q2 (human-check) |
| `amv3cy7d1` | resume after answer | 05:03:32–05:03:53 | human-check completed; run completed |

**The person's answers.**

- **q1, scope.** Proposal: a melee 「重戦士」 trained at the barracks; the bot
  unchanged; done when the unit is added, the headless test is updated and
  the browser is checked. Answer: 「ok、詳細は任せます。」
- **q2, result.** HP 160, melee 22, range 10, speed 55, 120 ore, 14 s; Q/W
  to train at the barracks, F/G to select all; 14/14 headless checks;
  browser verified (`verify-report.md`). Answer: 「OK」, taken up, and
  `human-check` completed as the person's acceptance.

**Publication.** Three checkpoints, one after each attempt, all `published`:

| Commit | After |
| --- | --- |
| `8181278` | attempt 1, awaiting the person |
| `095c914` | attempt 2, awaiting the person |
| `96cbe25` | completed |

The `study-rts` gitlink is unchanged (`d2734db`).

**Fresh Gitea checkout.** In a scratch directory:
`git clone --recurse-submodules developer/pj-rts-vs-bot`, with the token as a
scoped header; nothing was written to `.git/config`.

- `HEAD` = `96cbe25`, and the clone is identical to the workspace.
- It holds the run folder (`braindump.md`, `run.json`, `definition/`,
  `verify-report.md`).
- Game changes since `7e662e1`: `game/sim.js`, `main.js`, `README.md` and
  `test/headless.js`. `bot.js` is unchanged, as agreed.
- `node game/test/headless.js` in the clone: **all 14 checks pass**,
  including "barracks trains a heavy warrior".
- `wfe run check run-001`: usable. Through agdevworld,
  `/wfe/api/workspaces/rts-vs-bot/runs/improve-game/run-001` returns 200.

**Observations, not failures:**

- **No separate result decision.** `run.json` `decisions` is empty: the
  run view's Accept/Reject result decision was not used. The person's
  acceptance is the `human-check` answer, which is what the `improve-game`
  definition asks for. Whether that talk node should also record a
  decision is a design question for later.
- **Backend shown as unknown.** `wfe run show` prints "Executor: autolab
  (backend: unknown)". The attempts record their backend
  (`attempt.begin`), but the run header reads only the creation-time
  executor field. This is a display gap.
- **Artifact list includes `bot.js`.** `node.complete` for `implement`
  lists `game/bot.js` as an artifact, although that file has no change
  (the agent declared it).

## Documentation updated

| Where | What |
| --- | --- |
| `agdevworld/README_DEV.md` | commands (`editor:install`, `check`, `wfe-release.sh`), files, section "The workflow editor": ownership, separate process, operating area, releases, launchd, operating boundary, tests outside the package |
| `agdevworld/workflow_editor/README.md` | system-component head, explicit registry, operating boundary, `wfe project unregister`, fixture paths |
| `pj-agdev/devenv/launchd/com.agdev.wfe-service.plist.in` and `README.md` | the installed release from agdevworld's script; the plist does not change between releases (the installed copy was regenerated) |
| `agautolab/tests/test_wfexec.py` | `WFE_PACKAGE`, default `agdevworld/workflow_editor` |
| `pj-agdev/experiments/workflow_editor/README.md` | frozen archive, not a code source; points at agdevworld |
| `p4/report3.md`, `p4/report4.md` | correction references where they said to run `improve-editor` or deploy from the editor project |
| `p4/plan.md` | the person's revision (corrected goal, stage 4 and completion criteria), committed with this step |
| `pj-agdev/.local/devenv.md` (ignored) | host note: deploy, boundary, catalog, recovery copy, leftover candidate folder |

The autolab `wfrun` guide needed no change. It names `wfe` and its own
records, not where the editor comes from, and a refusal is already a fact it
works under.
