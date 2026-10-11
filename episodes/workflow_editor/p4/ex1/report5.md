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

## Real RTS acceptance: outstanding

The plan requires one improvement through `improve-game` in the RTS
workspace. The person specifies or agrees on it, answers the scope question,
and accepts or rejects the result. A request or an acceptance must not be
invented, and none was. The route is ready:

1. Open `http://localhost:8093/wfe/`, or the operation room's "agdev
   dashboard ↗". Then open the Project editor of `rts-vs-bot` → "Request a
   run" → workflow "Improve the game", and enter your name and your words.
2. autolab (`com.agdev.agautolab-wfexec`, idle and available) claims it and
   runs `confirm-scope`. It asks the scope question in the run view and ends
   its process.
3. Answer in the run view. autolab resumes and runs implement → verify, then
   asks the `human-check` question.
4. Inspect the game and its test results, then Accept or Reject with your
   evidence.

After that the Omni Agent verifies the published files from a fresh Gitea
checkout and records it, completing p4.

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
