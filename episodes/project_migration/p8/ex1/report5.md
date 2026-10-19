# project_migration p8 ex1 — Step 5 report: verify, deploy, demonstrate

Date: 2026-10-19 (≈ 13:30–14:10 UTC). Host details, commands and review
links are in the ignored local notes (`pj-agdev/.local/devenv.md`,
*project_migration p8 ex1 on this host*); raw evidence in
`pj-agdev/.local/project-migration-p8-ex1/`.

## Tests

| Suite | Result |
| --- | --- |
| WFE (`node --test test/**/*.test.ts`) | 440 pass |
| agautolab (`pytest tests`) | 47 pass, 1 skipped (pre-existing) — includes the new leftover-process test |
| agobserver (`pytest`) | 120 pass |
| Every existing run record (151, 11 workspaces) replayed by the new reducer (`wfe run check`) | all usable; one pre-existing missing-artifact warning |

## Deployment

Before: `nctl status` ok, `nctl drift --host agstudio` converged; `wfe
observe`: nothing executing, one person's stop (jevlab experiment/run-001)
and five malformed recovery runs. Stores and configs backed up.

| Component | Deployed | Previous |
| --- | --- | --- |
| WFE service + `wfe` | agdevworld `0251983` (release script; healthy on service and launcher) | `3095992` |
| Executor `wfexec` | agautolab `a06c530` (kickstarted idle; run contract `551b9ed` is read per launch) | `3ba5abc` |
| Observer monitor | pj-agdev `edf445c` tree (kickstarted) | `83c2dab` |
| Web (Front Desk) | container rebuilt from `0251983` | `3095992` |
| Front guide / recovery guide (Gitea) | pj-front `6c787ef`, pj-agops `72bb685` | `cbac8cb`… / `6e5c26e`… |
| Pins | pj-agdev `a7aecc9`, `6ec8ab6` (pushed) | `83c2dab` |

**Not deployed:** the Slack bridge release (it would poll a `due` run like
an executing one). The deploy command was refused by the session's
permission classifier; the change is committed and harmless to leave.

Rollback (code and records) is written in the local notes: activate
`3095992`, check out agautolab `3ba5abc` and agobserver `83c2dab`, kickstart;
the old reducer reads every record except one whose hold carries a `cause`
(none existed at deployment).

## Outstanding waits and holds

- jevlab experiment/run-001: the person's stop — left as it is
  (`Paused · iwaag asked …`).
- agops recover/run-013, 017, 023, 024, 025: each had run and acted
  (the target's record names its `run.recover`; four wrote `report.md`) but
  recorded no node, so they read `not-started` and Observer had told the
  owners twice today. Their `investigate` node was started and completed by
  *Omni Agent* with an outcome naming that evidence and who recorded it;
  history untouched otherwise. Observer closed the five incidents as
  completed. No hold was relabelled.

## Controlled end-to-end trials (deployed; actor `Omni Agent (for Developer)`)

**Trial A** — conversation `p8ex1-trial-a`: "have the glossary project add
'durable wait'".

- Front's run converse/run-029 adjusted at `talk`'s completion: `talk`
  completed, `add_entry` (delegate → glossary/freework) waiting, `tell`
  after it. The corrected shape, chosen by Front from its guide.
- The child glossary/freework/run-002 ran in the default slot and
  completed with its final-report step; the parent received **one**
  `completed` fact (and the non-waking `integrated`).
- **Found:** Front's session ended at 13:45 (final result written), but a
  backgrounded `find /` kept the harness alive; the attempt and the
  conversation slot stayed held until 13:54. This is the old "shell process
  left behind" limit, and it defeats the release of the slot. **Fixed** in
  the executor (agautolab `a06c530`): 30 s after the session's final result,
  leftovers are stopped and the attempt ends as the session did. Tested
  (fixture) and deployed.
- After the attempt ended, the fact queued the continuation; the new attempt
  took it up, judged it, completed `tell` and replied (#63) with links.
  Run: no hold at any point.

**Trial B** — conversation `p8ex1-trial-b`: "add 'execution slot'", with
the fix live.

| Time (UTC) | Fact |
| --- | --- |
| 14:02:44 | message posted; converse/run-030 starts in the conversation lane |
| 14:04:04 | dispatch recorded |
| 14:04:07 | parent attempt ended `exited`, no hold: **3 s after dispatch**; status `Waiting · glossary freework result`; no slot held by the parent |
| 14:04:13 | coordinator (`com.agdev.wfe-service`) restarted during the wait |
| 14:04:21–37 | an unrelated message (trial C, converse/run-031) served in the conversation lane while run-030 waited |
| 14:04:25 | worker (`wfexec`) restarted during the wait (it adopted run-031's live attempt) |
| 14:04:49–14:05:07 | child glossary/freework/run-003 in the default slot |
| 14:05:13 | child's result relayed (once) |
| 14:05:14–57 | parent resumed automatically, took up and judged the result, completed `report`, replied in the conversation; run completed, integrated |

Observer opened no incident for either trial's waits.

## Status in the LAN UI (deployed, `agstudio.local:8093`)

Screenshots in `…/step5/`: the Front Desk on trial B shows the reply and
`work: front/converse/run-030 — Done · Every step completed`; jevlab
experiment/run-001 reads `Paused · iwaag asked (answer to q1): …` in the
activity line and focus head; the converse runs read `Finished` /
`Completed`. Fixture captures of every waiting/held state are Step 4's.

## Against the completion criteria

| Criterion | Evidence | Kind |
| --- | --- | --- |
| run-027 reproduced; corrected Front waits without a hold | fixture tests; trials A and B (Front's own runs) | fixture + deployed |
| genuine active work still interrupts, with a short reason | fixture (`Interrupted · Talk node left running at exit`) | fixture |
| no parent process holds a slot while the child works; another request runs | trial B (parent ended in 3 s; run-031 served meanwhile) | deployed |
| hours-long wait through durable state and restarts | fixture (two store restarts, repeated reconciles); trial B across coordinator and worker restarts (minutes, not hours) | fixture + deployed |
| question, completion, failure, cancellation reach the parent; poor reports still wake it | fixture (question, completion, failure; cancellation in `delegateflow`); completion deployed | fixture + deployed (completion) |
| duplicates, multiple children, before/during/after exit, crashes between steps | fixture | fixture |
| missing delivery and queue records repaired, no duplicate child work | fixture | fixture |
| lost child process, interrupted parent, unavailable observation | Observer fixture tests | fixture |
| stop/cancel prevent continuation, keep evidence | fixture | fixture |
| short reason consistent in run view, list and Front Desk | fixture captures (all states); deployed captures (paused, done, desk) | fixture + deployed |
| deployment and end-to-end trial | above | deployed |

## Remaining limits

- A shell process left behind while the session is still working (before
  its final result) keeps the attempt open, as before; the run contract
  already says background work dies with the session.
- The deployed trials waited minutes; hours were simulated in fixtures.
- A child question deployed end to end was not exercised this time (it is
  in the fixtures and in p10's trials).
- Slack bridge `due` polling not deployed (above).
