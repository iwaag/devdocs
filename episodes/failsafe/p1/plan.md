# Failsafe p1 — recover a silently stalled request

## Goal and scope

Prove that an accepted Front → autolab request remains accounted for when
its worker silently stops, and can resume and finish through the system's
own recovery path. Observer is a normal part of that path.

Cover this request path, Observer restart, and misleading conversation
state. Assume Zulip and the work host remain available for the recovery
trial. Cross-host reassignment, mandatory worker leases, comprehensive
artifact replication, and automatic recovery of Zulip are later work.

This is a private experimental environment and a breaking-change phase.
Backward compatibility is unnecessary. Replace unsuitable state models,
code, and experimental fixtures as needed; choose implementation details
freely. Keep constraints focused on the behavior being proved, rather
than adding production security or preservation requirements.

## step1 — establish the recovery contract and reproduce the gap

- Read `braindump.md`, `case_and_opinion1.md`, and the current implementation.
  Capture a small reproduction of the reported stall; use the historical
  case as evidence, not as a dependency on that mission's current state.
- Define the minimum durable facts: request identity, outstanding work,
  observed evidence and its freshness, next review, recovery attempts,
  and evidence of completion or cancellation. Choose their storage and
  representation during implementation.
- Separate request completion, worker/process status, conversation state,
  and recovery progress. A process exit, an answer classification, a
  legitimate-wait judgment, or a sent recovery request ends no obligation
  by itself. A replacement transfers the obligation to its successor.
- Record the contract and expected reproduction outcome briefly in the
  phase report. No separate architecture project is needed.

Hints: `pj-agdev/agobserver/src/agobserver/monitor.py` already has durable
tracking, incident adoption, judgment snapshots, and recovery verification.
Inspect `requests`, `retain`, `verify`, `recovered`, and `RECOVERED_STATES`.
The case remained tracked: the missing behavior was useful reconsideration
after a progress post was classified as an answer. Retention alone is not
the fix. `pyagag/src/agag/trace.py` derives the conversation states.

## step2 — keep unresolved work due for review

- Ensure accepted requests in scope enter tracking and remain reviewable
  until an evidenced terminal outcome or a recorded handoff.
- Persist enough state to resume reviews after Observer restart, including
  requests older than the recent-discovery window.
- Schedule another look when progress is unconfirmed, including work
  classified as awaiting a requester. Legitimate waiting can postpone a
  review without making the request disappear or suppressing reviews forever.
- Use available conversation and execution evidence. Treat missing or stale
  evidence as uncertainty; a live listener does not prove a live task.
- Make review scheduling independent of successful model judgments, so a
  failed or incorrect judgment leaves the request eligible for another look.

Hints: reuse the mirror and existing judgment worker where useful. Inspect
`pyagag/src/agag/serving.py` and autolab's listener/run records for execution
evidence. Worker heartbeats are optional evidence, not a new admission
requirement. Fix truncated progress markers or misleading run outcomes if
useful, while retaining recovery when those signals are absent or wrong.

## step3 — close the recovery loop through Front and autolab

- Have Observer bring an unresolved request to Front with the request link,
  observed facts, uncertainty, and the outstanding outcome.
- Give Front and autolab enough tools and guidance to inspect the work and
  choose a response: continue waiting, resume a stopped task, report a
  blocker, or seek a human decision. Avoid prescribing every harness failure.
- Persist attempts and the next verification time. Bound repeated nudges
  and escalate unresolved cases through the existing human-reporting path.
  Select and document practical intervals during trials.
- Verify fresh evidence of resumed work after a recovery action; continue
  tracking the original obligation through completion. Acknowledgment or
  another promise to report is insufficient evidence of recovery.
- When the original execution may still be active, investigate or wait
  rather than automatically starting a competing execution. Automatic
  reassignment under uncertainty is outside p1.

Hints: existing Observer incident topics, Front's ownership of the origin,
and autolab's mission workspaces provide the starting points. Preserve
useful partial work when resuming. The reported harness timeout is an
unverified inference; recovery must not depend on that exact explanation.

## step4 — demonstrate behavior with bounded failure trials

Use a small disposable Front → autolab mission and focused regression tests.
Before changing running services, inspect Nautobot or `nctl status` /
`nctl drift` as described in `pj-clusterintent/nctl/README.md`. Use the local
environment notes for deployment and dependency pins.

| Trial | Required evidence |
|---|---|
| Worker ends after a progress post, without a failure report | Observer detects unresolved work; Front/autolab resume it and reach recorded completion. |
| Healthy work takes longer than the review interval | Review does not launch duplicate work; tracking continues through normal completion. |
| Progress is misclassified as an answer | The wrong state does not indefinitely exempt unfinished work from review. |
| Observer restarts during tracking or recovery | Pending reviews and attempts survive; recovery continues without repeated actions caused by lost state. |

Combine live trials with controlled replay where appropriate. Exercise the
silent-stop recovery end to end with real in-system agents, and restart
Observer with an outstanding request. Use existing fixtures and fault hooks
when helpful; extend them only where the behavior needs coverage.

Record detection time, recovery time, false interventions, duplicate
execution or side effects, retained outputs, and human interventions.
Set a detection target from the configured review interval before the trial.
Developer fault injection is expected; Omni Agent rescue is an intervention,
not a successful autonomous recovery. Diagnose failures and repeat the
affected trials after fixes.

## step5 — consolidate and report

- Remove superseded paths made unnecessary by the new contract, and update
  affected guides and development documentation to describe actual behavior.
- Write `report.md` with the implemented contract, relevant code locations,
  test results, trial evidence, measured timing, and remaining limitations.
  Keep machine-specific details in ignored local notes.
- Commit and push implementation and documentation changes in their owning
  repositories, updating dependency references as needed.

P1 is complete when the four trials meet their stated outcomes, the silent
stall reaches completion without Omni Agent rescue, and restart preserves
the recovery loop. Report blockers honestly rather than extending the phase
into a general high-availability or distributed execution redesign.
