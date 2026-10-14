# Agent session p1 ex1 — launch and session mismatch fixes

## Scope

Fix the two review findings in p1 and correct its report. Keep the changes
small; no new session policies, cost experiments or legacy compatibility work.
The Developer requests minimal, fast testing. Creating this plan does not
change services or start implementation.

## 1. Keep ownership of a launched process until it is stopped

In `agautolab/src/agautolab/wfexec.py`, retain the child/process-group handle
immediately after spawning it. A failure to send the prompt or record
`exec started` must not leave a live child while recording the attempt as ended.

- On startup failure, terminate and reap the spawned group before finalizing
  the attempt. If termination cannot be confirmed, retain ownership and the
  occupied slot, and expose the failure instead of permitting another writer.
- Address the executor-crash window between spawn and PID registration: use a
  minimal launch handshake so work cannot start until its process identity is
  durably registered. An abandoned pre-registration child must exit without
  executing agent work. A stale `launching` row alone is not proof that no
  writer exists.
- Preserve interrupted-run recovery and the existing process-group cleanup.

## 2. Stop and refuse a mismatched session

When the harness reports a different session ID, record both IDs and the
mismatch, stop the affected attempt and confirm termination. Do not silently
adopt the reported ID or continue work in an unverified conversation.

- Make WFE session selection refuse automatic reuse of a mismatched session.
  Expose an actionable reason; the existing explicit retirement/replacement
  route follows investigation of the run and any effects already produced.
- Keep the mismatch visible to Observer/recovery. Retiring a session must not
  bypass a human hold, permit a second writer or reset the recovery budget.
- A transient failure to record attachment must remain pending for retry,
  rather than marking the attachment as successfully handled.

## 3. Verify only the changed failure boundaries and correct the report

Use focused tests with stub processes and injected failures; no paid model
calls, long sleeps, full RTS rerun or broad suite rerun.

| Fast regression | Required result |
| --- | --- |
| Spawn succeeds, `exec started` fails | Child group is gone before attempt finalization; no orphan writer |
| Executor disappears before registration | Child cannot start agent work; restart cannot overlap writers |
| Harness reports another session ID | Attempt stops, mismatch is retained, next claim refuses reuse |
| Attachment write fails once | Recording retries; evidence is not silently dropped |
| Explicit retirement after mismatch | A later permitted attempt selects a new session; recovery budget is unchanged |

Use events/pipes and short bounded polling rather than timing-based long waits.
Run only the affected test cases and the relevant TypeScript type check. Do not
rerun the 88-second executor suite or the full project suites merely for
coverage. Run mandatory release-script checks once during deployment; do not
duplicate them beforehand or disable the release gate.

Correct `report5.md`: the displayed four-run costs total **$14.06**, not
$11.06; reconcile with the retained measurements. Review the overall trial-cost
summary for the same error. Describe compaction and per-node session splitting
as untested improvement candidates, not established savings. Keep the `fresh`
default and the small-sample qualification.

## 4. Deploy and report

Commit/push the fixes and required submodule pointers. Before activation, check
Nautobot/nctl and WFE state, then deploy matching editor/executor changes without
interrupting unrelated work. Verify release identities, executor/Observer health
and that the existing human hold remains intact; no new live workflow trial is
required after the focused regressions pass.

Write a short `report.md` with the fixes, exact tests/results, deployed versions
and any remaining limit. Keep machine details in ignored local notes and
commit/push the report. Completion requires both failure boundaries to be
closed, not merely a narrower claim in the original report.
