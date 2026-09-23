# Robust workflow p2 — Durable tracking and evidence-backed recovery

Sources: [p1 report](../p1/report.md), [p1 trials](../p1/report5.md), and the follow-up review. Original desire: [braindump](../p1/braindump.md).

## Goal and working approach

Keep unfinished requests traceable through renames, long waits and observation failures. Report recovery only when evidence establishes the expected transition. Make a stopped or ineffective Observer monitor visible without relying on that monitor to report itself.

- This is a private experimental environment and a breaking phase. Backward compatibility is unnecessary. Implementers may replace interfaces, code and disposable state; choose the smallest coherent design rather than preserve p1's structure.
- Favor shared operation semantics and useful tools over more agent instructions. Preserve human acceptance and cancellation decisions, and carry existing authorization forward.
- Keep secrets and machine-specific details in ignored files. Routine implementation choices need no additional approval; no new security framework is required.
- Focus p2 on identity, observation and recovery correctness. Wider discovery for routines, argues and Project Room, mission-close simplification and incident UI can follow unless a small related change is necessary here.

## step1 — Reproduce the gaps and define observable outcomes

- Read p1's reports and inspect the current implementation and deployed consumers. Check service state and observation freshness through Nautobot or nctl.
- Reproduce candidate disappearance after an unreadable target or a change of failure kind; a request aging out of discovery; an origin renamed or resolved while work remains; and an unrelated home reply after a delegated answer arrives.
- Map identity handling across trace, continuation, callback routing, progress/result writes and Observer discovery, incident keys and recovery destinations. Mark remaining name-based joins and duplicated resolution logic.
- Define the evidence needed to distinguish recovery, cancellation, observation failure, another unresolved blockage and escalation. Record which concerns are reproduced defects and which remain hypotheses.

Done: `report1.md` contains small reproductions, the affected paths and concrete acceptance criteria for the changes.

Hints: `agobserver.monitor.tick()` currently closes an incident when its candidate key disappears. `origins()` selects unresolved Front topics active within 12 hours. `agag.trace.Candidate.key` includes a topic name; `_taken_up()` accepts later speech at home as evidence that an answer was handled. These are code-review findings, not previously observed production failures. p1 also found repeated identity defects and missed its six-minute detection target once.

## step2 — Use stable identity throughout the affected paths

- Give each tracked request, conversation and incident a stable identity based on existing anchors. Resolve its current location when reading, writing or dispatching recovery; keep names for display and initial discovery.
- Consolidate identity resolution where it removes inconsistent behavior. Cover ordinary rename, resolve/unresolve, replacement, deleted anchors and reuse of an old name. A replacement remains distinct work with an explicit relationship to its predecessor.
- Apply the same identity to incident deduplication and retry accounting so a rename does not create a new recovery allowance or redirect an old incident.
- Exercise callbacks, progress posts and prepared reply delivery through these changes. Preserve p1's automatic task progression and existing delivery recovery.

Done: the affected request and its incident survive rename/restart, and reuse of the old name neither captures its replies nor causes duplicate work.

Hints: start with pyagag's existing `Conversation`, `locate`, rootchat and replacement relations. p1's fixes span `agag.trace`, `agag.listen`, Front continuation and autolab progress writes; inspect the complete paths rather than patch only the monitor. No general identifier migration framework is required.

## step3 — Retain unfinished work and verify recovery positively

- Separate recent-request discovery from retention of work already being tracked. Keep unfinished requests and open incidents across inactivity, origin closure and restart until their outcome is established. On restart, rediscover outstanding work beyond the recent window where existing records permit it.
- Require fresh, sufficiently complete observations before declaring recovery. Confirm the relevant transition: work actually started, the specific answer was processed, or an explicit terminal decision was recorded. Candidate disappearance alone is insufficient.
- Keep cancellation, loss of visibility, a different blockage and successful recovery distinct. A resolved origin is not by itself evidence that its dependent work finished or was cancelled; report unresolved cases through an available route.
- Tie answer handling to the actual input boundary or an explicit receipt for that answer. Later speech at home must not consume an answer that the serving never received.
- Preserve bounded recovery attempts across state changes and restarts. Report exhausted or unobservable cases without repeatedly restarting work whose outcome is unknown.

Done: no tested loss of evidence produces `rescued`, no unfinished tracked request silently expires, and concurrent replies are accounted for by the input actually processed.

Hints: reuse serving journals, served marks and mirror records before adding storage. Distinguish trace computation time from source freshness. The current `MirrorReader` reads cached contents; verify how it exposes missing coverage or a stale feed. A failed read is not deletion. Retention can use a small persisted index reconstructed from existing records; it need not scan the whole realm every tick.

## step4 — Expose monitor progress and detect loss of coverage externally

- Publish a small monitor-health record: last completed monitoring cycle, source freshness, outstanding coverage/backlog, and the latest failure. Include enough information to distinguish idle, disabled, stalled and unable to observe.
- Have an existing independent process, such as the relay or the cluster observation path, evaluate this record and make failure visible to the operator. A heartbeat emitted by the monitor alone is not an alert mechanism.
- Ensure slow or stuck triage cannot silently prevent other requests from being checked. Choose bounded calls, scheduling or worker isolation as appropriate; measure the oldest unchecked request as well as cycle duration.
- Verify recovery of monitoring after interruption without resetting incident history or blindly replaying recovery requests.

Done: a stopped monitor thread, stalled judgment and stale mirror are distinguishable from healthy idle operation and become visible without the failed monitor sending a notification.

Hints: p1 measured local judgments at 43–91 seconds and executes them inside a monitor loop. Process liveness therefore does not establish monitoring progress. Nautobot reported Observer as `unobserved` in p1 and the follow-up check; inspect the status collection path before adding another service. Use deterministic health checks rather than another judging agent.

## step5 — Verify a fixed final revision under combined failures

- Deploy affected consumers and verify installed revisions and running processes. Keep models and configuration fixed for the acceptance run, recording the full version set.
- Run focused fixtures for the boundaries below, then three ordinary multi-task cycles and two live repetitions of the combined rename/recovery and monitor-failure scenarios. Include a forge-to-autolab handoff and concurrent independent requests.

| Scenario | Required result |
|---|---|
| Candidate becomes unreadable or changes failure kind | No false recovery; visibility or the remaining blockage is explicit |
| Origin and child rename/resolve; old name reused | Tracking and delivery stay with the original anchors; no duplicate incident or work |
| Inactivity exceeds the discovery window, then restart/resume | Unfinished work remains tracked and resumes once |
| Answer arrives during another serving or unrelated home reply | Only input actually processed is marked handled |
| Monitor thread stops while its process stays alive | Independent health reporting detects the loss of coverage |
| Triage stalls or mirror goes stale | Other requests remain covered or degraded coverage is explicit |
| Human wait, cancellation and ordinary completion | No unnecessary recovery; each terminal outcome has its own evidence |

- Use controlled clocks for long-window boundary fixtures. Also run one real long-duration job/wait scenario across the configured silence threshold; state clearly what was time-compressed and what ran live.
- Before acceptance testing, set detection and external-health reporting targets from measured cycle and judgment times. Keep p1's six-minute missing-start target as the starting point; justify any adjustment before the run, not after a miss.
- A code fix during acceptance starts a new final-revision run of the affected cases and normal cycles. Retain failed attempts in the report.

Done: the final version set passes the agreed trials with zero false recovery reports, duplicate work, lost tracked requests or Omni Agent rescue. Ordinary human decisions and operator fault injection are recorded separately from rescue. Fixture-only guarantees remain labeled as such.

## step6 — Complete rollout and report the reliability boundary

- Update affected dependencies, introductions and developer documentation. Remove instructions made obsolete by the shared identity and recovery behavior.
- Write `report.md` with versions, evidence, final-revision results, detection/recovery times, oldest-unchecked delays, additional calls and runs, and remaining limitations. Separate prevented failures, verified recoveries, cancellations and explicit stops.
- Record any work done for an in-system agent as `did X for agent Y — handoff candidate`. Identify the next useful expansion from the observed results, including whether wider discovery or mission-close simplification now has priority.
- Commit and push changes and necessary submodule references.

Done: the deployed system has durable request tracking, evidence-backed recovery and independently visible monitor health, with a repeatable acceptance record.

Hints: pyagag changes require consumer lock/environment updates and listener restarts, not just a library push. Most work is in pyagag and agobserver, with Front/autolab write paths and relay or pj-clusterintent observation changed as needed. Keep rollout evidence about versions actually installed and running.
