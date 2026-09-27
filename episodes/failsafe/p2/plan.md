# Failsafe p2 — diagnose early and learn from recoveries

## Goal and scope

Extend p1's Front → autolab recovery path with task-specific health checks
within minutes, and a durable developer review for every recovered incident.
Prove that an unreported worker exit reaches Front within five minutes of
the exit, without disturbing healthy quiet work.

Keep this phase focused on autolab and its current harness. Expose a small
health interface whose implementation can vary by harness; participation in
the wider system does not require it. Cross-host reassignment, replication,
and recovery of Zulip remain outside this phase.

This is a private experimental environment. Backward compatibility is
unnecessary; replace unsuitable code, state formats, and fixtures freely.
Choose storage, transport, and diagnostic tools during implementation.
Avoid adding production security or preservation requirements unrelated to
the behavior being proved.

## step1 — define timing and separate resumption from acceptance

- Read `../p1/report.md` and `../p1/report4.md`, then inspect the current
  trace, monitor, serving journal, and autolab close-out path.
- Fix the p1 finding that "continue and report" could count as acceptance.
  A recovery request authorizes resumption; closing a task still needs
  evidence of the requester's acceptance of its result. Test both a resume
  that must leave the task open and a subsequent acceptance that closes it.
- Define separate facts for process liveness, work progress, waiting reason,
  observation freshness, and recovery state. Unknown is a useful result.
- Define timestamps for failure onset, last confirmed progress, first
  suspicion, health check, request to Front, and developer escalation.
  Repeated claims or verdicts on unchanged evidence do not reset uncertainty.

Start with these operational targets, then tune them with recorded evidence:

| Condition | Action / target |
|---|---|
| Confirmed ended or failed execution with unfinished work | Enter recovery on the next monitor cycle. |
| No confirmed progress for 2–3 minutes | Run a task-specific health check. |
| Continued uncertainty | Ask Front to investigate within five minutes of first suspicion. |
| Still unresolved after ten minutes of uncertainty | Escalate to the developer with facts and unknowns. |

The separate end-to-end target for a worker exit is five minutes from the
actual injected exit to the request reaching Front, including discovery,
probe, queue, and judgment time. A time threshold triggers investigation,
not a requirement that agents post progress at fixed intervals.

Hints: `pyagag/src/agag/trace.py` has `silent=2700`, `quiet=1800`, and
`unheld=300` thresholds. `pj-agdev/agobserver/src/agobserver/monitor.py`
also has retry, rejudgment, judgment-deadline, and escalation delays.
Review the whole path: lowering only `silent` leaves other long waits.
Autolab's task close-out is in `agautolab/src/agautolab/zulip_listener.py`.

## step2 — expose task-specific execution health

- Map the request/conversation and serving identity to the actual harness
  execution. Inspect process existence/exit, recent execution events, and
  active tool or child-task waits where available.
- Return compact facts with observation times, source identity, and unknowns.
  A live listener or heartbeat alone does not establish task health; lack
  of output alone does not establish failure.
- Observe known wait targets directly when practical. Support a legitimate
  quiet wait without demanding periodic model-generated conversation posts.
- Bound probe duration and resource use. A dead host, failed probe, or
  unavailable adapter returns uncertainty promptly rather than blocking
  monitoring. Prevent stale evidence from a previous serving being applied
  to its replacement.

Hints: start from autolab listener/run records and
`pyagag/src/agag/serving.py`. Keep harness-specific inspection behind the
health interface. A file, local service, or existing tool may be sufficient;
an elaborate universal telemetry platform is not a p2 prerequisite.

## step3 — use early diagnosis in the recovery loop

- Run cheap checks before the former long silence threshold, including
  unfinished work misclassified as awaiting a requester or answered.
- Route confirmed stopped work promptly to the existing recovery path.
  Pass evidence and uncertainty to Front for ambiguous cases; retain agent
  discretion over diagnosis and the next action.
- Keep confirmed healthy waits under review. Do not start competing work
  merely because a quiet timer expires. Process liveness without progress
  or an explained wait remains eligible for further investigation.
- Give probe and model failures bounded paths to investigation/escalation.
  Neither a slow judgment queue nor repeated `legit` verdicts on unchanged
  evidence may silently extend the new timing targets.
- Persist timing and attempts across restart; deduplicate requests and
  verify resumed work using fresh evidence, continuing through acceptance.

## step4 — hand recovered incidents to the developer

- On verified recovery, create a developer review linked to the existing
  incident and request. Track operational recovery separately from the
  status of the underlying problem's review or fix.
- Include the failure timeline, diagnostic evidence, recovery action and
  outcome, detection/recovery duration, confirmed cause versus hypotheses,
  and useful improvement candidates. An unknown cause does not prevent handoff.
- Use an existing developer-facing conversation or work-record mechanism.
  Notify on the first handoff; append related recurrences to the review,
  retaining each occurrence's evidence. Make uncertain grouping explicit
  rather than presenting a guessed common cause as established.
- Escalate repeated failures or unsuccessful recovery more visibly. Choose
  a small, documented recurrence policy; avoid a new issue-management system.
- Make review creation and notification recoverable after interruption,
  without duplicate reviews on repeated monitor passes. Developer acceptance
  of this follow-up is separate from acceptance of the original task.

## step5 — validate quickly, including real recovery

**Effort requirement: minimize implementation-test waiting without weakening
what the test proves.** Do not spend 30–45 minutes waiting for a threshold
that can be exercised with a controlled clock or shorter test configuration.

- Use injected clocks and replay for threshold boundaries, backoff,
  recurrence, and restart scheduling. Prefer explicit synchronization over
  arbitrary sleeps in process tests.
- Use short configurable intervals for repeated integration trials, while
  preserving event ordering, stale-evidence checks, real process termination,
  and the real recovery path. Allow enough time for real model/tool latency.
- Run at least one real silent-exit recovery using the intended operational
  configuration to verify the five-minute target. Stop measuring when the
  request reaches Front; no need to wait out the entire target window.
  Continue the mission to completion through the in-system agents.
- Distinguish accelerated test results from operational timing evidence.
  Record settings and justify any unavoidable long wait. Restore temporary
  test settings after trials.

| Trial | Required evidence |
|---|---|
| Harness process exits without a closing post | Health diagnosis reaches Front within five minutes of exit; work resumes and finishes without Omni Agent rescue. |
| Healthy quiet tool or child task exceeds the probe threshold | Additional checks run; no competing execution or false recovery; normal completion. |
| Process remains alive but progress/wait cannot be established | Investigation and escalation occur within configured bounds, rather than indefinite healthy classification. |
| Health probe fails or model judgment stalls | Other work remains monitored; uncertainty reaches Front/developer within bounds. |
| Observer restarts during diagnosis or review delivery | Timers and attempts survive; no duplicate recovery action or developer review. |
| A recovered failure recurs | Both occurrences retain evidence and link to the developer follow-up; recurrence policy is exercised. |
| Resume request followed by actual acceptance | Resume alone leaves the task open; acceptance closes the reviewed result. |

Use focused regression tests for mechanics and bounded live trials for
agent behavior. Before changing services, inspect Nautobot or `nctl status`
/ `nctl drift` using `pj-clusterintent/nctl/README.md` and local environment
notes. Developer fault injection is expected; count Omni Agent rescue as
an intervention, not autonomous success. Repeat affected trials after fixes.

## step6 — consolidate and report

- Update guides, affected dependency pins, and development documentation;
  remove superseded long-delay behavior on the covered path.
- Write `report.md` with the health contract, operational timings, trial
  evidence, false interventions, duplicate actions, human interventions,
  probe/model costs, developer handoffs, and remaining limitations.
- Keep machine-specific details in ignored local notes. Commit and push
  implementation and documentation in their owning repositories.

P2 is complete when the silent-exit target is demonstrated, healthy quiet
work survives the extra checks, ambiguous cases have bounded follow-up,
recovered incidents reach developer review, and resumption cannot substitute
for acceptance. Unsupported execution environments remain an explicit
limitation rather than an invitation to expand this phase indefinitely.
