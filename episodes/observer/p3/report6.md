# observer p3 step 6 — failure, waiting and recovery end to end (2026-10-14)

Source at agdevworld `5dacbbe`, agobserver `35ea6c5`, agautolab `1d2bb78` (not yet deployed; step 7). Trial evidence (logs, stores, results) is in the ignored `pj-agdev/.local/observer-p3/step6-matrix/` and `…/step6-realtrial/`.

## How the trials ran

**Matrix** — a private area with disposable projects `demo` (`ship`: work → more; `ask`: talk → more) and `agops` (`recover`, the real recovery workflow and guide). Real code throughout: the source `wfe`, autolab's `wfexec` loop (scripted agents through `agent_command`; a scenario line in the request decides what the agent does), Observer's `WorkflowMonitor` and `Recoverer` looking every 3 s with bounds shortened for the trial (`quiet_log` 20 s, `escalate_uncertain` 90 s, `executor_stale` 20 s, `unobservable_report` 20 s, graces 15 s), the real local model for judgments. Zulip is a loopback that hands posts in the recovery channel to the area's intake (`wfe intake receive`/`process`), as the transport does. Faults were injected locally: no real site's rate limit was needed.

**Real path** — a second private area (`p3trial`/`study`, `p3ops`/`recover`) on the real Zulip realm: the executor ran with `[notify]` and `[intake]` enabled (it provisioned `wf-p3trial.*` and `wf-p3ops.*`), Observer ran as `python -m agobserver.wfmonitor` with its own bot credentials (incident notes in `#agobserver-agstudio1`), and the recovery agent and the continuation were the real Claude Code harness (`sonnet`). Only the target's first attempt was scripted, to make the motivating mistake exactly.

## Results (matrix, third run, all passing)

| Trial | Outcome | Times (s from the matrix start) |
| --- | --- | --- |
| S1 lost background retry (+ the request post's answer lost, the post delivered to intake twice, Observer restarted after assigning) | 1 incident (`attempt`), 1 assignment, 1 post, 1 recovery run; `restored` on "node.complete (seq 9) in the continued attempt after the recovery (seq 5)" | opened 9.4, assigned 9.4, acknowledged/performed 16.1, verified 19.5 |
| S2 a 45 s foreground command inside its declared 90 s timeout; an external wait with a 30 min deadline; the second run queued behind the first | no incident; seen `working`, `queued`, `awaiting-external` | — |
| S3 an unanswered question and a person's stop, while another run proceeds and completes | no incident | — |
| S4 an attempt alive and silent (no open call, no CPU), then its process killed | `uncertain` → one local judgment (`stalled`, 43 s) → the process killed → same incident (`lost-attempt` → `interrupted`), recoverable only then; 0 assignments while the process lived; `restored` | opened 150.6, judged 203.4, recoverable/assigned 220.1, performed 225.8, verified 229.2 |
| S5 an answer queued behind healthy work (S2) versus an answer whose job was lost where reconcile cannot repair it (dropped under its receipt) | the queued one: no incident; the lost one: `continuation` incident → `run.recover` without a hold → `restored` | opened 247.0, performed 253.3, verified 256.8 |
| S6 executor killed with a run queued | one `executor` incident naming the queued run, owners told once, no per-run repairs; `resumed` when it reported again | opened 303.7, closed 303.9 after restart |
| S7 the run's remote removed: its checkpoint fails after completion | `publication` incident → the recovery agent retried the publication (failed again) and left it → `handed-off`; run stays completed with 1 attempt, no decision; after the remote returned, the person's retry published | — |
| S9 the person resumes / cancels while the recovery is pending | recovery refused by WFE (`recover-owned` / `run-cancelled`), its report says so, no `run.recover` recorded; outcomes `manual` and `cancelled` | — |
| S10 a held run's copy moved away; the observation command failing | `observation` incident told to the owners after 20 s, never a rescue; `resumed` when the copy returned; two failed looks: health `source: failed`, open incidents unchanged | — |

Model calls: 1 (S4). False positives: none (`false-alarm` outcomes: 0; no incident on S2/S3 runs). Interventions (all the harness's, standing in for faults and for the person): executor stops/starts, the agent kill (S4), the dropped queue entry (S5), the removed/restored remote (S7), the person's resume and cancel (S9), the moved copy (S10).

**Defects the matrix found, fixed and re-run:**

- the observation incident of a run whose record could not be read had no uid and never closed when it became readable (now matched by its place, closes `resumed`) — run 1;
- a lost attempt was sent to recovery while the record still had it open, with `--attempt none`; WFE refused the stale episode after the executor ended the attempt (now assigned only once the record holds it, naming the episode's attempt) — run 2;
- before the matrix: a lost queue entry had no recovery act (`run.recover` now also queues an unowned continuation, `recover-owned` when a job exists), and an escalated uncertain attempt confirmed lost did not move to the recovery path.

## The real-path trial (real recovery agent, normal Zulip delivery and wakeup)

| Time (JST) | Event |
| --- | --- |
| 12:55:28 | `p3trial study/run-001` requested |
| 12:55:33 | attempt `amv7kuaae-7af684` ended → `interrupted` (`fetch` running; the session ended with its background retry stopped) |
| 12:55:58 | Observer: incident `i98d9a6d76d` opened, assignment 1 journaled; note #16054 in `#agobserver-agstudio1 › incident-i98d9a6d76d`; request #16055 in `#wf-p3ops.recover › request-i98d9a6d76d` |
| 12:56:00 | the executor's intake transport read the post: `request → recover/run-001` (input kind `request`, requester Observer) |
| 12:56:01 | the recovery attempt launched (real `sonnet`) |
| 12:56:42 | `run.recover` recorded by `recovery-agent` (agent `p3ops/recover/run-001`, attempt `amv7kuxdf-956809`) |
| 12:57:09 | recovery run completed; its result posted in the request topic by WFE's notifier |
| 12:57:11 | continuation attempt `amv7kwfu0-b7bac7` launched; its brief said it was continued by recovery |
| 12:58:55 | `node.complete fetch` |
| 12:59:01 | Observer: `restored` |
| 12:59:16 | run completed |

The recovery agent's reason, verbatim in the record: it checked the record, `wfe exec status`, the copy index and the attempt's session log; the agent "backgrounded 'sleep 60 && curl export.arxiv.org/…' after a 429, then ended its turn; the session (and thus the background task) was killed before the 60s sleep elapsed, so the curl never ran. No external effect occurred, nothing was written beyond the original run files, and record/copy agree on fetch still being unstarted work. Safe to continue." Its report (kept locally) has the same content with the commands it ran. One incident, one assignment, one post, no manual restart; detection 25 s (one 30 s look), request delivered and taken up 2 s later, recovery decided in 44 s, verified 3 min 28 s after the interruption. Cost: recovery $0.39, continuation $0.30 (`total_cost_usd` of the attempts). Owner messages: none (nothing escalated). Human or Omni interventions: none after the request.

That restoration was verified on the agent's `node.start` of `write` (after its `node.complete` of `fetch`). A bare start is not proof of work, so restoration now requires a completion, progress note, question, wait, attachment or recorded failure (`35ea6c5`); the real trial would have closed one look earlier on the `node.complete`.

## Suites

- WFE 301/301, type check and build (agdevworld `5dacbbe`).
- agobserver 104 passed.
- agautolab `test_wfexec`, `test_wfnotify`, `test_wfintake`, `test_wfconversation`: 29 passed, 1 skipped, after fixing a test fake that predated this phase (it failed against the released WFE as well: projects have had a second workflow channel to provision since p8/ex2) — `1d2bb78`.
- The watchdog suite moves with the watchdog in step 7.

## Left in place

The real trial's Zulip folders `wf-p3trial` (#34) and `wf-p3ops` (#33) and their channels (#287–#290) remain until step 7's cleanup; the private areas live in the session scratchpad.
