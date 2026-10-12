# Workflow editor p7 — step 7: verification, deployment and release record

## Tested commits

| Repository | Commit | What |
| --- | --- | --- |
| agdevworld | `cde57cd` (deployed) | p7 editor: appearance (`5e38f75`), presentations and lifetime (`25aa19f`), adjustments (`cb65490`), CLI flag fix and shorter decision evidence (`cde57cd`) |
| agautolab | `d31f654` | wfrun common contract: presentations and adjustments |
| pj-rts-vs-bot (Gitea) | `72b69cf` | `improve-game` with presentation and cleanup, `agent/review.md` |
| pj-agdev | `0dcfe08` | submodule pointers |

## Verification

- **Editor.** `npm run check` (tsc, tests, build): **191/191 pass**. That is 172 before p7, plus appearance 4, presentations 8, and adjustments 7 (including the CLI flag test). The release build ran the same check in its own directory: 190/190 on `cb65490`, then `cde57cd`.
- **Executor.** agautolab `test_wfexec`, `test_wfnotify`, `test_wfintake`, `test_wfconversation`: **31/31 pass** against the p7 editor.
- **Browser checks** (private services, isolated registries):
  - `appearance` 15/15;
  - `review` 23/23 (review link, review screen, non-served paths, after-stop view, adjusting from the run view);
  - `runs`, `exec`, `conversation` all pass;
  - `docsteps` still fails at the same pre-existing spot recorded in p6 (`attached()`, a null bounding box). It is not introduced by p7 and not investigated.
- **Fake notification transport.** Message rendering for `presented`, `cleanup-failed`, `decided` (cleanup pending) and `adjusted` is tested in the editor suite. No Zulip post was made in p7's verification.

### Required evidence

| # | Evidence | Where |
| --- | --- | --- |
| 1 | Appearance round-trips and does not change execution or obscure state | `test/appearance.test.ts`; `checks/appearance.ts` (running pill and ring keep their colors under a custom color) |
| 2 | A review link opens the game and the request-specific explanation together, through the intended route | `checks/review.ts`; **live smoke below**: the person's route (a browser on this host, opening `http://127.0.0.1:<port>/`) with the real RTS game |
| 3 | Agent exit during a human wait leaves the server running | `test/present.test.ts` (process group killed); live smoke twice: the starting shell's whole process group was SIGKILLed, and a wfexec-style session was killed with `killpg`; the server stayed and the review answered 200 |
| 4 | Workspace edits do not change a presented revision; stale answers cannot accept a newer one | `test/present.test.ts` (copy unchanged after a source edit, `presentation-superseded`); `test/adjust.test.ts` (the correction's old answer cannot be decided) |
| 5 | Repeated start/stop, restart recovery, startup and shutdown failure, no duplicate or unrelated process affected | `test/present.test.ts` (reuse, reconcile after kill, tampered bundle, pid-reuse bystander, unverifiable stop then retry); live smoke (repeated stop harmless) |
| 6 | An adjustment keeps past state and replays; stale, cyclic, mid-node, question-destroying and access-expanding changes are refused | `test/adjust.test.ts` |
| 7 | A correction leads to new work and a new review without resetting completed nodes or reusing the prior decision | `test/adjust.test.ts` (correction after review) |
| 8 | Acceptance goes to cleanup without a second acceptance; a cleanup failure blocks completion and stays retryable | `test/present.test.ts` (one decision; `presentation-open` until the verified stop; stop-failed then retry); live smoke (final report refused while p1 served) |
| 9 | Cancellation stops owned resources through its own path and reports what remains | `test/present.test.ts` (cancellation) |
| 10 | Pending instructions cannot be bypassed by an adjustment or by finishing | `test/adjust.test.ts` (authorization test) |

### Live smoke with the deployed release and the real game

This ran in a scratch area (in the session scratchpad): a clone of `pj-rts-vs-bot` at `72b69cf`, the release's own CLI (`releases/<commit>/cli/wfe.ts`), a private service of the release on :18196, and ports 18700–18709. No catalog entry and no Zulip post was made.

- `improve-game/run-003`:
  - The plan was agreed, and implement and verify were recorded with their reports.
  - `wfe present register … --source game --entry index.html` froze 7 files (revision `13d18bdd…`). `present start` served it, and the starting shell's whole process group was SIGKILLed with the server staying up.
  - The human-check question was asked with `--presentation p1`.
  - In the browser: the run view showed "Open the review: 兵舎のアップグレード（スモーク）" → `http://127.0.0.1:18700/`. One click showed the real RTS game running (the canvas painted, 11957 lit samples) beside the Japanese explanation, with the answer link to the run view. There were no page errors.
  - The answer "合格！" was taken up and the decision recorded, bound to `p1`/`13d18bdd…`.
  - The final report's completion was **refused** while p1 was served (`presentation-open`). `present stop` was verified (process gone, port closed); a repeated stop was harmless. The final report then completed.
- `improve-game/run-004` (after the fix release): the agent was launched as wfexec does (a new session) and killed with `killpg`; the server survived (review 200). `present stop --all` was verified, and repeating it was harmless.

The smoke found two defects, fixed in `cde57cd` and tested:

- `wfe present stop <run> --all` and `wfe run adjust --dry-run` took the next word as their value.
- A presentation decision's evidence listed every file; it now names the presentation and its revision.

## Release

- **Before.** No open attempt, no queued job, every run in the catalog completed, no presentation. Stores were backed up to `.local/agdev/.local/backups/p7-20261012-141108/` (exec, notify, intake, zulip.json).
- **Deployed** with `agdevworld/scripts/wfe-release.sh deploy .local/agdev`: `cb65490`, then `cde57cd`. Both were healthy: the service and `bin/wfe` report the commit. `http://localhost:8093/wfe/` (agdevworld's route) answers.
- **After.** All six existing runs (`rts-vs-bot` 2, `ppt` 4) check `usable`. `bin/wfe present status` reports no presentations.
- **Host configuration.** `.local/agdev/present.json` (ignored): bind `127.0.0.1`, ports 8300–8399, `answerBase` `http://localhost:8093/wfe/`.
- **Executor.** No wfexec restart was needed: autolab's code is unchanged, and the contract guide is read at every claim.

### Schema changes and affected records

- `run.json` stays `ag.workflow-run.v2`. The additions (`presentations`, `definition.revisions`, `definition.active`, three operations) appear only in runs that use them. No existing record changed or was converted.
- New host store `present.sqlite` (v1). `exec.sqlite`, `notify.sqlite` and `intake.sqlite` are unchanged. The compiler stays `ag.wfe-compile.v2`; the notify protocol stays 2.

### Rollback constraints

- `wfe-release.sh rollback` returns to `cb65490` (also p7).
- To go to the pre-p7 release, activate `0c2c072` explicitly. Afterwards, any run that used a presentation or an adjustment cannot be read by that release (`op-unknown`), though its files are intact. Live presentation instances must be stopped first, because `0c2c072` has no `wfe present`.
- No database restore is needed. Restoring the backups after new deliveries would replay messages, as in p5/ex1.

## Limitations

- A reachable bind (`publicBase`) was not exercised. The intended route is the person's browser on this host.
- Instances are not watched: after a reboot, "not live" shows only when someone looks. Recovery is explicit.
- `docsteps` remains failing (pre-existing).

## Next

Step 8: a real RTS review cycle with the person's own improvement request.
