# p17 ex1 step 5 — deployed, and the ordinary path proven

Plan: [plan.md](plan.md), step 5. Machine-specific details (backup path,
logs, timestamps, recovery commands) are in the ignored
`pj-agdev/.local/p17-ex1/notes.md`.

## State before, and deployment

- Read first: `nctl status` ok; `nctl drift --host agpc` converged; agpc
  worker on wfe `36c6613`, executor `cb83634`, active. The WFE service ran
  `3c8b438` and no attempt was open.
- Backup: every area sqlite store, `workers.json` and `present.json`. No
  store schema changes.
- WFE: `wfe-release.sh deploy` → agdevworld **`ae7d050`**. Its release
  check ran 600/600, and the service and `bin/wfe` report the commit. All
  365 run records on the host read under it (0 problems).
- agpc: the stored state has a new field, so the worker must replay it.
  Nautobot placement `agautolab-agpc` → `wfe_version ae7d050`, rendered,
  then `setup_wfe_worker.yml` (ok=22, changed=3). agpc now runs `ae7d050`
  and executor `cb83634`.
- Executor (agautolab) and aghangout did not change.
- Rollback: release `3c8b438` cannot read a run whose cleanup entry has
  `release: true`. Code and data rollback are written together in the
  notes; the records involved are the agops release-check runs listed
  there.

## Fixture

agops workflow `release-check` (pj-agops `10f0826`): `present → review →
cleanup`, custom documentation with a final report, as in run-013. Agent
`check-agent` (sonnet) does the work itself. The check actor
`p17ex1-check` is the requester and reviewer. Every decision is marked
synthetic, and nothing is recorded under the Developer's name.

## Results

| Run | Condition | What happened | Recovery used |
| --- | --- | --- | --- |
| release-check/run-001 | **ordinary path through the real review page** (headless Chrome, Start review then Approve) | The agent presented p1 and left it serving, then prepared rv1 and ended normally. agpc fetched the page (200). Approval at +0 s; verified stop at +2 s (agpc then got no answer); job `review-approved:…/rv1@16` claimed; attempt 2 began at +5 s. Its cleanup node said "p1 already stopped by review cleanup; verified nothing serves it". The run completed at +21 s and its integration was queued. | none |
| run-002 | **lost callback + restart** (controlled) | Approval through a `ReviewHost` without execution callbacks queued no job. The executor was restarted 1 s later, and its reconcile queued the continuation 1 s after that from the run's facts. The run completed. | none |
| run-003 | race attempt (missed) | The attempt ended 3 s after prepare, before the approval landed. In effect a second idle-path run; it completed. | none |
| run-004 | **approval inside the open attempt** (controlled) | The agent's process group was paused right after `review.prepare`, approval went over HTTP, then the agent resumed. `release.during` = that attempt; its end was `exited` with no hold although `cleanup` was ready. One job, one later attempt, never concurrent; completed. | none |

None of these runs has `run.recover`, and Observer started no recovery
run (agops `recover` is still at run-037, from before this episode). The
four runs show 2 attempts each; no person action came after the approval.
After the checks no instance is serving except as noted below.

The runs were driven by fixtures and controlled conditions; the agent's own
work was autonomous. The agent presented, prepared the review and did the
explicit cleanup in each run without instruction beyond the workflow's node
descriptions. The approval, the lost callback, the restart and the paused
process were Omni's controlled actions.

## Left for the Developer

The Developer's completed waterfall review (gimmickimg/freework/run-013)
was not touched.

Waiting example: agops **release-check/run-006 rv1**, requested as "Omni
Agent (for Developer)". Its page is stopped; Start review restarts it, and
Approve should stop it and run the cleanup without anything else.
Review page: `http://agstudio.local:8093/wfe/#/ws/agops/review/release-check/run-006/rv1`.

- did "stop run-006's page after the agent left it serving, so the example
  waits stopped" for check-agent — handoff candidate.
- run-005 was cancelled: Omni requested it under the Developer's name by
  mistake, and the cancellation reason says so. run-006 replaced it.
