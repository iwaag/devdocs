# project_migration p8 ex1 — Step 1 report: trace and reproduce

Date: 2026-10-19. Read-only investigation plus one controlled fixture; nothing
was deployed or changed on the host. Raw evidence (observation snapshot,
record excerpts) is in the ignored `pj-agdev/.local/project-migration-p8-ex1/step1/`.

## What is deployed now (observed 2026-10-19 ~12:40 UTC)

| Component | Revision | Notes |
| --- | --- | --- |
| WFE service and `wfe` | agdevworld `3095992` (release) | agdevworld `main` is `61778d4`; the four commits after the release touch `integrations/slack` only |
| Executor `com.agdev.agautolab-wfexec` | agautolab `3ba5abc` | lanes: `default` 1, `conversation` 1 on `studio`; both idle |
| Observer `com.agdev.agobserver` + watchdog | pj-agdev `83c2dab` tree (`agobserver/` is a directory of pj-agdev) | |
| Front project | Gitea `developer/pj-front` `cbac8cb` | workflow `converse`: one `talk` node, lane `conversation` |

`nctl status`: ok (Nautobot 3.1.3, worker idle). `nctl drift --host agstudio`:
converged; placements agautolab, agobserver, agdevworld-web, slack-front
applied.

Outstanding runs at the snapshot: jevlab `experiment/run-001` stopped by the
person (legitimate); agops `recover/run-013, 017, 023, 024, 025` read as
`not-started` with `investigate` ready although each had one attempt (see
finding 5).

## run-027, reconstructed from its record

`front/converse/run-027` (one message: a jevlab literature survey) →
child `jevlab/study/run-002`.

| Seq | What | Effect |
| --- | --- | --- |
| 5 | `run.adjust` r2: adds delegate node `survey` (jevlab/study), **no edge, no node after it** | talk and survey are independent |
| 6–7 | `node.start survey`, `node.start talk` | both running |
| 8–9 | `delegation.dispatch`, `bind` | survey `waiting` on d1 |
| 10 | `attempt.end exited` | **held `interrupted`**: "talk recorded as running" |
| 11 | `run.resume` by Front's *other* run (converse/run-028, as operator) | |
| 12–15 | attempt 2 reads the record, has nothing to do, exits | **held `interrupted` again** (talk still running) |
| 16–20 | five `delegation.event completed` (child seqs 12, 14, 16, 18, 20) | five waking facts for one result |
| 22 | `run.recover` by agops `recover/run-024` (incident `idd2703abac`) | |
| 23–30 | attempt 3: takes up, disposes satisfied, completes survey **and talk**, replies | completed |

The child itself ran normally (one attempt, 11:21–11:39). Its recovery run
`recover/run-024` was queued at 11:22 and waited 17 minutes for the default
slot the child held.

## Reproduction (controlled fixture)

`agdevworld/workflow_editor/test/delegatewait.test.ts` (new; the delegation
fixture of `delegateflow.test.ts` moved to `test/delegateFixture.ts` so both
use it):

1. **run-027 reproduced**: a one-`talk` converse workflow; the attempt
   starts talk, adjusts in a delegate node beside it, starts and dispatches
   it, exits 0 → `interrupted`, hold reason names `talk recorded as running`,
   the delegate node is `waiting`. This is also the proof that genuinely
   unfinished work still ends as an interruption.
2. **corrected shape, existing machinery**: the same attempt adjusts *at
   talk's completion* (`completing`), adding the delegate node and a
   `deliver` talk node after it; dispatches; exits → `exited`, no hold, no
   open slot; the child is claimed in the released slot, completes; one
   continuation; the parent takes up, disposes, completes `survey`, runs
   `deliver`; two parent attempts in all.

Both pass. So the shared wait operation and the executor already express a
normal delegation wait; the failure was the shape Front gave its graph.

## Ownership as it is

| Concern | Owner (source) | State |
| --- | --- | --- |
| The wait | parent `run.json`: `delegation.dispatch` sets `node.wait.on = {delegation}` (`shared/run.ts`) | correct |
| Ending an attempt | `attempt.end` reducer: exit with a running node, or ready work beside no wait (unless `defer-ready`), or an untaken waking fact from before the attempt → `interrupted` hold with a sentence | correct; the reason is prose only |
| Slot | `ExecStore.finished` ends the attempt row and its job; the slot is free at once | correct (fixture 2) |
| Child facts | `DelegateStore.relayOne` (intake pass every 5 s, reconcile every 60 s), facts by receipt `delegation:<uid>:<child seq>:<type>` from `childFacts()` | **duplicates `completed`** (finding 2) |
| Continuation | `DelegateStore.continuation` (first untaken waking fact; one open job per run), `finished()` after an attempt, `reconcile()` for lost jobs | correct; tested in `delegateflow` (lost job requeued once) |
| Event during an attempt | recorded with `during`; the attempt's end queues the next | correct |
| Interrupted/unknown holds | Observer `wfclassify` → incident → `agops/recover` run → `wfe run recover` | works; recovery runs leave malformed records (finding 5) |
| Status words | `situation()` + per-view word tables | **conflated** (finding 3) |

## Findings and the changes they need

1. **Front's graph shape (the root cause).** The guide's "Waiting" fact says
   the attempt ends and frees its slot, but nothing tells Front that its own
   `talk` must not be left running, nor where the result's handling lives.
   The common contract (agautolab `wfrun` guide) already states the
   interruption rule. *Change*: Front's guide states the run-027 fact and
   the shape (complete talk at the adjustment that adds the delegation; a
   node after the delegate node handles the result); `wfe delegate request`
   names nodes still running in the attempt and what the attempt's end will
   make of them; `wfe delegate help`/docs state the same. No new operation is
   needed: fixture 2 expresses the wait with existing ones.
2. **One result relayed five times.** `childFacts()` emits `completed` on
   every entry whose change is `→ completed`; documentation steps of the
   child reopen and recomplete the execution while its attempt still runs.
   *Change*: relay `completed` once the child's execution is completed and
   no attempt of the child is open (at its `attempt.end`, or at the
   completing entry when made outside an attempt); a later reopening and
   recompletion is a new result.
3. **Status vocabulary.** `situation()` returns `awaiting-person` for every
   recorded wait, and for an answer recorded but not taken up (the
   executor's move). Consumers inventoried:
   - WFE: `server/exec.ts` (checkpoint label, answer continuation),
     `server/observe.ts` (`situation`, child situation),
     `server/conversations.ts` (runs of a conversation),
     `server/delegate.ts` (`childNow`), `src/runView.ts` (chips, child
     words), `src/runSide.ts` (focus card, child words, Detail execution),
     `src/runActivity.ts` (activity line), `src/projectView.ts` (run badge:
     "Waits for you" for any wait; holds not shown — `RunSummary` has no
     hold), tests (`exec`, `observe`, `sessions`, `activityline`, `migrate`
     strings).
   - Observer: `wfclassify.py` (`QUIET`; it reads `waiting.responsible`, not
     `situation`, so a rename does not change its decisions), docs
     `workflow.md`, `wfmonitor.py` condition list.
   - agdevworld root: the Front Desk (`src/roomState.ts`) receives
     `runs[].situation` but shows no run status at all.
   *Change*: one shared interpretation in `shared/run.ts` giving state,
   short reason, target and continuation condition; `situation()` splits
   person / delegation / external waits and the executor's due move.
4. **Reasons.** A hold's reason is a sentence written by the reducer;
   `runActivity` shows only "Held"; `runSide` wraps the sentence in generic
   recovery prose. *Change*: structured cause on new interruptions (gated on
   new `attempt.end` entries so older records replay unchanged), concise
   text derived from it, the sentence kept as detail.
5. **Malformed recovery records.** Five `agops/recover` runs ended their
   attempt `exited` with `investigate` never started, so their record says
   `not-started` while work was done (they did `wfe run recover`). The
   reducer only interrupts an exit when execution is `in-progress`.
   *Change* (Step 3): an exit after which nothing was ever recorded is an
   interruption with its own cause (new entries only); the recovery guide
   states that its run's work is its `investigate` node.
6. **Already covered, to keep**: lost continuation requeued once by
   reconcile; parent hold keeps facts and queues nothing; cancellation
   leaves the child; dispatch by receipt; delivery crash points.

## Next

Step 2 implements 1, 2 and the structured cause (4, record side), with
lifecycle tests. Step 3 covers 5 and the reconciliation/Observer gaps.
Step 4 covers 3 and 4 in the views.
