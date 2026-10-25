# Workflow editor p17 ex1 — cleanup ownership and continuation after approval

## Goal

Make approval finish review cleanup and continue any remaining workflow work
without recovery-agent intervention. A person approves once; cleanup and
downstream work proceed through their normal owners, survive restarts, and
do not require another approval or manual resume.

This follows [p17/report7.md](../report7.md). The chat review list is already
deployed; this episode addresses the separate cleanup stall it exposed.

## Confirmed cause and evidence

Conversation `20261025-013759-jz8ebz`, ponderer/conduct/run-042 delegating to
gimmickimg/freework/run-013. The child had `present -> review -> cleanup`;
the final report was already complete before the person's review.

| Child history | What happened |
| --- | --- |
| seq 19, `review.prepare` | Child prepared rv1 and later ended its attempt normally while awaiting the person. Its presentation was already serving. |
| seq 25, `review.decide`, 2026-10-25 10:39:47 JST | The person approved rv1 in the browser. |
| seq 26, `review.cleanup` | WFE recorded `status: left`: the instance was reused and its owning run was unfinished, so that run should stop it. The review became complete and its node completed, making the ordinary cleanup node ready. |
| seq 27, `run.recover`, 10:44:21 JST | Recovery-agent found ready work with no execution and explicitly resumed the run. |
| seq 31–33 | The child started cleanup, stopped the presentation and completed. |

The host cleanup did run; it deliberately handed off stopping the page.
The stalled step was the agent-owned cleanup after that handoff. There is
no evidence here of an agent ignoring an approval, a failed stop operation
or a worker process crashing.

Two implementation gaps explain the sequence:

1. `server/reviews.ts`, `ReviewHost.cleanup`, treats every reused instance
   whose owning run is unfinished as someone else's responsibility. It
   does not distinguish the review's own run from a different unfinished
   owner. In this case the review was part of that same run. The reducer
   accepts `left` as resolved review cleanup, so the round becomes complete
   even though the page still serves.
2. Approval has no continuation path for work unlocked by review completion.
   `ReviewHost.decide` calls the continuation callback only for requested
   changes. `ExecStore.afterReview`, the attempt-finished path and reconcile
   likewise cover changes-requested continuations, not completed approved
   reviews with ready successors. After `cleanup.state = done`, `hostOwes`
   is false, so host cleanup retries cannot repair that missing execution
   handoff. Observation correctly reports ready work without a job, and
   recovery-agent eventually compensates.

Existing tests cover a final review completing the run and reuse of pages
belonging to completed child runs. They do not cover this combination:
same-run reuse, an explicit successor cleanup, and approval after the agent
attempt has ended.

## Conventions and scope

Follow `localrule.md`, the development policies and the p17 contracts. Write
generated files in English and communicate with the Developer in Japanese.
Implementation, appropriate commit/push, deployment and deployed checks need
no human approval milestone. Human approval in a test is an attributed
result-review fact, not permission to perform this development.

Keep historical runs unchanged. Backward compatibility is not required;
prepare matching backups/recovery if events or stores change. Keep local
addresses, credentials and deployment/evidence paths in ignored notes.
Read Nautobot/nctl state before inspecting/changing running services, using
`pj-clusterintent/nctl/README.md` and local environment notes.

Fix the specific lifecycle and scheduling gaps. Do not generally auto-start
every run with ready nodes, suppress Observer's incidents, shorten recovery
polling, force a new agent instruction, or change integration permissions.

## 1. Reproduce both gaps before changing behavior

Add an end-to-end regression fixture modeled on the child's actual graph:
work/present, own Human review, explicit successor cleanup and final report.
Start the page before preparing review, end the attempt, start the review
as an identified check actor and approve it. Capture round state, process
ownership, ready nodes and queued attempts at each boundary.

Demonstrate that the old behavior leaves the page to its unfinished owner
and creates no ordinary continuation. Keep this fixture independent of
recovery-agent; it must prove that the repaired normal path succeeds.
Also reproduce approval while the preceding attempt is still open, to
cover the race with attempt completion.

## 2. Resolve cleanup by actual ownership

Review the existing presentation usages, instance identity, retained-page
records and cleanup results. Decide whether stopping is owed using those
facts and the relationship between the round and owning run. `reused`
describes how the page was obtained, not an exemption from cleanup.

- For a reused page owned by the same run as the approved round, stop it
  through the existing Presenter lifecycle when no other active usage or
  requested retention owns it. Record verified stopping on the owning run.
- Respect another active review and explicit requested retention, including
  visible ownership. Do not stop an unrelated run's page merely because
  the person approved a review that references it.
- Where an unfinished different run still owns serving the page, preserve
  a truthful handoff with the owner and any outstanding work visible. A
  `left` result must not imply that the process stopped. Check whether the
  existing records express this sufficiently; extend them only as needed.
- Existing explicit cleanup nodes remain ordinary workflow work. They
  must still execute to completion, even if the host already stopped the
  page. An already-stopped presentation is an idempotent cleanup outcome.
- Failed stops remain owed and retryable. Do not complete required host
  cleanup or ask for approval again because a stop failed.

Update review status/help where necessary so review complete, retained or
handed-off serving, and remaining workflow work are distinguishable.
Avoid solving this only by instructing agents not to add cleanup nodes.

## 3. Continue work unlocked by approval durably

Add an executor-owned continuation for approved review completion, separate
from the existing changes-requested correction continuation. Trigger it
after `review.cleanup` actually completes the review node, rather than
immediately at the approval click while host work remains pending.

Use the completed review event/round and its record identity as durable
evidence. Queue an ordinary attempt only when there is relevant ready work
left and the run is executable. Respect executor ownership, execution-node
assignment, holds, cancellation and an already-open attempt. If the review
completes the whole run, launch no needless agent turn; preserve the existing
publication/integration behavior for host-driven completion.

Provide all three delivery opportunities through one shared eligibility
rule:

1. Normal path: cleanup completion requests the continuation promptly.
2. Attempt-end path: if approval/cleanup happened while an attempt was
   running, reassess when that attempt ends instead of launching another
   concurrently or losing the event.
3. Reconcile path: recover a missed callback or queue write from the durable
   run facts, even though the round's host work is already marked done.

Bind receipts to the review completion and run creation; deduplicate jobs
and do not recycle an old continuation for a later review round. Establish
how an attempt consumes the completion handoff so repeated reconcile does
not keep launching work after it was handled. Re-read the run after host
reconciliation changes it; do not schedule from its earlier snapshot.
Serialize competing callbacks/queue writes using the existing ownership and
concurrency mechanisms. Preserve ordinary interruption handling if a new
attempt fails to do or defer its work.

Review the coordinated and remote-worker routes: a browser operates on the
coordinator, while execution may belong to a different worker node. Route
the continuation to the run's assigned node through the existing job
mechanism. A held run remains held; ordinary resume can pick up its ready
work. This continuation does not reopen the person's decision.

## 4. Focused verification and required checks

- Exact incident shape: same-run reused presentation, idle execution,
  approval, verified stop, successor cleanup automatically claimed once,
  final completion and parent result delivery. No `run.recover` or extra
  person action is used.
- A final review with no successors stops its page and completes without
  an extra agent attempt. Keep the existing deferred parent-review case.
- Approval during an open attempt, including before its end is recorded,
  produces one later attempt when needed and no concurrent execution.
- Crash/restart after review completion but before enqueue is repaired by
  reconcile. A duplicate callback or repeated reconcile queues no duplicate
  work. Cover multiple rounds, unrelated ready work, holds and cancellation.
- Stop failure retains the recorded approval, retries stopping and only
  then releases successor work. A retained page or another review's usage
  is preserved and shown honestly.
- A page owned by a different unfinished run is not incorrectly stopped;
  ownership remains visible. Cleanup of an already-stopped page succeeds.
- Exercise assigned remote execution and parent/child identity checks.
  Requested-changes continuations, same-child follow-up and existing
  cancellation cleanup retain their behavior.

Run affected tests during implementation, then the required WFE checks.
Avoid overlapping duplicate full suites; the p17 follow-up full check took
several minutes and the release path runs its own checks. Run executor or
UI checks only if those sources change. Cover browser/API and CLI routes
that initiate the relevant lifecycle operations.

## 5. Deploy and prove the ordinary path

Obtain current service/worker state through Nautobot/nctl and inspect local
release notes. Back up affected stores/configuration as required. Deploy
through established release paths and update worker pins if their runtime
changes. Verify the coordinator and applicable worker versions. Keep code
and data rollback instructions together where new events require them.

On the deployed release, run an identified-actor fixture of the incident
shape through the real review UI. Verify that approval stops the page and
queues/finishes the explicit cleanup successor without Observer recovery.
Check the page from another LAN host while it serves and verify stopping
afterward. Separately exercise the lost-callback/restart repair and the
already-running-attempt race under controlled conditions. Clearly separate
these fixtures from autonomous agent behavior.

Do not alter or replay the Developer's completed waterfall review. Leave a
small safe example awaiting review, stopped and restartable, if needed for
minimal-click human inspection; check actor approval must already have
proven the completed path. Leave no unowned disposable serving instances.

Write `report.md` here with root cause, implemented ownership/scheduling
contract, checks, deployed run/event references, limitations and recovery
references. Update the p17 follow-up report and local development notes.
Completion requires the normal path and restart repair to work without
recovery-agent intervention; a successful eventual recovery alone is not
proof of the fix.
