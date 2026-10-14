# Observer p3 — workflow observation and recovery

## Objective and agreed direction

Implement [braindump.md](braindump.md) using workflow records, Git-backed
work and executor evidence as the basis of observation. Prove an entire
interruption-to-recovery cycle, rather than stopping at detection or notification.
The motivating failure is documented in
[project_migration p2 report3](../../project_migration/p2/report3.md).
Account for the proposed session policies in
[agent_session p1](../../agent_session/p1/braindump.md).

The Developer explicitly permits breaking the old agent system. Front, forge
and the other existing agents will be replaced by workflow-driven execution;
their existing operation is not a compatibility requirement. Zulip becomes
inter-agent communication, not the authority for work or its lifecycle.
Do not maintain parallel legacy and workflow monitoring paths, adapters that
pretend runs are Front conversations, or old listener behavior merely to keep
retiring agents operational. Preserve useful implementation, evidence and
working data; permission to break compatibility is not a reason to erase them.

This plan covers implementation, verification and deployment. Writing the plan
does not start implementation, resume existing work or change services.

## Starting evidence and scope

- The study attempt left a retry command in the background and ended its turn.
  The process was lost and the run became `interrupted`; a later manually
  resumed attempt completed. The executor already detects this exit condition.
  The run guide has since gained an evidence-backed foreground-wait explanation.
- The experiment's intentional hold blocked other runs under the old workspace
  ownership rule. Workflow editor p9 introduced isolated run copies and removed
  that reservation. Observer must respect the hold, not resume it as a repair.
- `agobserver.monitor` discovers Front conversations and uses conversation
  traces and legacy serving probes. Those assumptions are replaced in p3.
- WFE already owns run transitions, receipts, sequence checks, attempt fencing,
  queues, reconciliation, checkpoints and integration. Its `exec status` and
  activity reader expose useful facts. Extend these boundaries rather than
  creating another executor or another authority for run state.

In scope: workflow discovery, observation, incidents, communication to a working
recovery recipient, recovery execution, verification, and Observer health.
Replacing every legacy agent, implementing general delegate nodes, introducing
session reuse, and building a general external-job platform are outside p3.
The minimal recovery recipient and its wakeup path are in scope; the phase must
not depend on a future Front replacement to finish its acceptance test.

## Authority and responsibility

| Subject | Authority and owner |
| --- | --- |
| Request, plan, questions, answers, instructions, hold and completion | Workflow records and referenced documents |
| Work products, checkpoints and integrated changes | Run working copies, Git commits and publication/integration journals |
| Attempts, process identity, queue ownership and activity | Executor observations and execution records |
| Incident evidence, recovery assignment and verified outcome | Durable Observer records linked to the workflow identity |
| Agent-to-agent requests and replies | Zulip transport, carrying references to the records above |

Workflow documents and Git are the durable work record. Uncommitted work in the
authoritative run copy is still current work; a shared-branch copy of `run.json`
is a historical snapshot after p9, not a second live authority. Process facts
and transport journals remain host-local operational data. No commit frequency,
topic resolution, acknowledgement or conversational promise proves progress.

Observer detects and coordinates; a recovery agent investigates and decides
what work is needed; WFE validates and performs lifecycle transitions; the
executor owns process management. Existing plan agreement and result acceptance
remain separate from permission to investigate or resume an interrupted run.

## 1. Define the observation and recovery contracts

Inspect current WFE, executor and Observer code and document the smallest shared
contract before implementation. Use repository-relative references in tracked
documents. For local services, consult Nautobot or nctl as required by the
workspace instructions and read the ignored development-environment notes.

- Identify a run independently of its directory, branch label or Zulip topic.
  Resolve ambiguity across registrations explicitly; include provenance needed
  to distinguish different creations with the same display identifiers.
- Define a versioned observation result with observation time, source freshness,
  run sequence, current attempt, execution/queue status, hold intent, waiting
  reason and responsible party, outstanding inputs, activity evidence, and
  checkpoint/integration problems. Unavailable evidence is explicit.
- Keep a snapshot consistent enough to make a decision: detect changing run
  sequences or attempts while collecting evidence and re-read when necessary.
  Mutations must still revalidate the current state at the WFE boundary.
- Define incident identity from the run and the interruption episode, including
  its triggering attempt or record sequence. A different diagnostic description
  of the same episode must not replenish retry budgets or create duplicate work.
- Define recovery records: evidence references, assignment, acknowledgement,
  proposed/performed action, actual actor, attempts, first suspicion, next review,
  outcome and links to the target run. Keep workflow state authoritative.
- Specify the initial recovery authority: inspection and continuation of already
  authorized work are available; explicit human holds and cancellation remain
  binding. Scope changes or unresolved external effects become a recorded
  question. An interrupted attempt does not itself grant permission to replay
  side effects or accept a result.

Inventory reusable code with a short keep/adapt/remove rationale. Candidates
include durable delivery/read-back, incident persistence, bounded asynchronous
triage, freshness checks, process probes and the independent watchdog. Preserve
these where useful without carrying over Front/rootchat/serving assumptions.

**Completion evidence:** a documented ownership map and fixtures describing the
motivating interruption, intentional hold, normal wait and unavailable source.

## 2. Expose workflow observations through WFE

Add a read-only CLI/API surface (suggested shape: `wfe observe --json`) using
existing run resolution, execution views and activity parsing. Observer should
not learn SQLite schemas or scan working-copy layouts itself.

- Enumerate registered runs, including isolated copies, runs absent from the
  shared branch, and completed runs with unresolved publication/integration.
  Keep active incidents observable even if normal run discovery changes.
- Resolve live records through `runSpace`; Git supplies change and publication
  evidence, not a replacement for the live state. Missing copies or registrations
  are observation problems, not proof that work finished.
- Distinguish executor heartbeat, attempt process liveness, harness activity,
  tool/child activity and workflow progress. A healthy daemon does not prove a
  healthy attempt. Extend process evidence only where existing facts cannot
  distinguish the required failure cases.
- Expose named waits, deadlines and externally observable progress where known.
  A live PID or a log write alone is insufficient proof of useful progress;
  remote or uninspectable work remains explicitly uncertain.
- Keep reads bounded and incremental where practical. Observation must not
  reconcile, enqueue, create copies, mutate run records or post to Zulip.

**Completion evidence:** CLI/API agree on live run identity and state, including
p9 copies, stale/missing sources, long commands and concurrent state changes.

## 3. Replace conversation-based monitoring

Implement workflow discovery and monitoring as Observer's main execution model.
No per-run watch registration or Front-origin message is needed to detect a
stopped workflow. Retain tracking until a recorded terminal outcome or explicit
incident disposition, independently of run age and recent chat activity.

| Observation | Response |
| --- | --- |
| Explicit human hold or unanswered human question | Respect the recorded wait; no automatic resumption |
| `interrupted` or `unknown` without an existing disposition | Open or update an incident and arrange investigation |
| Ready work or received answer without a valid execution/wait owner | Check queue and executor; account for healthy work ahead |
| Live attempt with insufficient activity evidence | Probe and, when necessary, ask the local triage model; retain uncertainty |
| Missing executor heartbeat | Record infrastructure failure and use an independent recovery/notification route |
| Failed/stalled checkpoint or accepted integration | Track the outstanding operation separately from execution completion |
| Stale or unreadable source | Report observation failure; never infer rescue from absence |

Mechanical facts should be handled in code. The local model receives a compact,
consistent evidence packet only for ambiguous investigation or wait judgments.
Do not build an exhaustive recovery taxonomy or force the agent through a fixed
command recipe. Preserve evidence references and the exact basis of judgments.
Invalidate judgments when their evidence becomes stale.

Define and test detection/review/escalation bounds. Repeated uncertainty must
not reset the first-suspicion time; a legitimate-wait judgment schedules another
look rather than permanently suppressing monitoring. A failed component shared
by many runs should produce one actionable infrastructure incident with the
affected runs, not a flood of equivalent repairs.

**Completion evidence:** required cases are classified from workflow/executor
evidence with no dependency on Front history, topic names or legacy serving IDs.

## 4. Complete the recovery path

Provide a minimal workflow-driven recovery recipient, its discoverable contract,
and a durable intake/wakeup path. Use Zulip for the agent-to-agent request and
reply, linked to the durable incident and target run. Do not rely on a generic
post to a legacy listener to start recovery.

- Route work-content investigation to an agent with the target run's records,
  guides and retained files. Route executor/service problems to a recipient
  that can run independently of the failed execution path; if none is available,
  make that limitation visible and escalate through an independent path.
- Communicate evidence and the question to resolve. The agent decides whether
  to continue, repair the execution environment, or ask the person, using tools
  whose help owns their invocation details.
- Journal assignment and delivery before sending. Correlate replies and outcomes
  by incident identity; recover interrupted delivery and lost acknowledgements
  without creating a second recovery task. Zulip failure remains visible and
  does not stop observation or erase an assignment.
- Investigate residual processes, dirty files, checkpoints and external effects
  before replaying work. Git history cannot prove that an external operation did
  not happen. Preserve the target working copy and its uncommitted work.
- Add a WFE recovery operation if needed instead of impersonating a person's
  `run resume`. Record the recovering agent and incident, with sequence/attempt
  guards, durable receipts, bounded retry count and backoff. The guide and run
  view must reflect this new authority instead of still claiming that every
  interrupted run can only be resumed manually.
- Serialize target-run writers. A recovery agent must not modify the same copy
  while an old attempt remains alive. Fresh human holds, cancellation, answers
  or manual recovery invalidate stale actions. Investigation must not consume
  the only execution slot while waiting for a continuation queued behind itself.
- Bound automatic recovery of recovery work. Repeated failure escalates with
  evidence rather than recursively creating recovery runs forever.

Record restoration only after later observations confirm useful work or a valid
handoff such as a recorded question. Assignment, acknowledgement, a new PID or a
promise is not restoration. Cancellation, a deliberate hold, an erroneous alarm
and successful completion have distinct outcomes.

**Completion evidence:** an interrupted disposable run is investigated and
continued through the new path without manual restart, with one incident,
bounded assignments and a verified outcome.

## 5. Separate sessions from execution and retire obsolete assumptions

Keep the observation contract compatible with future session policies without
implementing those policies in this episode:

- Run = the work; session = retained harness conversation; attempt = an active
  execution interval. Session identity is optional until supported by the
  executor, and multiple intervals may later reference the same session.
- An idle retained session can legitimately await an answer or external result.
  A session's existence does not prove an executing process. A process restart
  does not create a new obligation or reset incident age.
- The current executor's attempt time budget must remain distinct from session
  lifetime and wait deadlines. Document the current limit and the change needed
  when agent_session p1 implements reuse.
- Future delegation can provide parent/child run and result-wait references;
  do not fabricate conversation ancestry or implement general delegation now.
- Session reuse does not make background jobs durable. Retain the current
  evidence-backed waiting guidance unless the job-lifetime contract changes.

Remove the old Front discovery, conversation-derived work lifecycle, legacy
serving probes and incompatible recovery prompts from the active Observer path.
Retain reusable components and historical evidence. Explicitly inventory old
watches/incidents: archive or record their retirement without replaying them or
claiming the underlying work completed. General condition observation may remain
as reusable functionality, but an old Zulip watch entrance is not required.
Avoid deleting shared libraries still useful to the workflow implementation.

**Completion evidence:** Observer starts and operates without legacy agents;
tests model both a fresh session and multiple intervals sharing a session ID.

## 6. Verify failure, waiting and recovery end to end

Use disposable runs and repositories. Do not resume the person's held experiment
or inject faults into unrelated active work. Reproduce the failure mechanism
locally rather than depending on a real site's rate limit.

| Trial | Required outcome |
| --- | --- |
| Agent exits with running/ready work and a lost background retry | Interruption detected; recovery agent inspects and continues; useful progress verified |
| Normal long foreground command and explicit external wait | No false automatic restart; wait remains observable |
| Human hold and unanswered question | Remain untouched while another run proceeds |
| Process disappearance and live-but-unresponsive process | Confirmed loss distinguished from uncertainty; no concurrent writer |
| Answer queued behind healthy work, versus lost queue entry | Legitimate queue wait distinguished from stalled continuation |
| Executor outage | Independent observation and actionable escalation/recovery work without the failed executor |
| Checkpoint/integration failure | Work retained; recovery does not bypass acceptance or replay completed external effects |
| Observer restart, duplicate message, lost delivery reply | Same incident and bounded recovery assignment; no duplicate continuation |
| Hold/cancel/manual recovery during investigation | Stale action refused; new state respected |
| Missing/stale source and Observer loop failure | No false rescue; independent health reporting |

Record detection, assignment, execution and verified-progress times; false
positives; model calls; and every human/Omni intervention. One complete trial
must use the real recovery agent and normal Zulip delivery/wakeup path, not only
mocked decisions. Keep fixtures for deterministic failure boundaries and run
the affected WFE, executor, Observer and delivery/watchdog suites, plus relevant
type/build checks. Do not retain legacy tests solely as compatibility gates.

## 7. Deploy, verify and report

- Commit/push changed repositories and required dependency/submodule pointers.
  Build and deploy the committed versions as a coordinated WFE/executor/Observer
  change; account for the recovery recipient and its communication intake.
- Recheck local service state through Nautobot/nctl and live WFE state. Quiesce
  affected writers where needed, preserve existing work and evidence, and prevent
  old monitor processes from emitting obsolete recovery requests after cutover.
  No dual-monitor compatibility period is required.
- Extend or relocate the reusable independent watchdog so it reports Observer's
  workflow-observation freshness, loop failure and recovery backlog. Do not leave
  monitoring dependent on a legacy Front/relay process scheduled for retirement.
- Keep hostnames, absolute machine paths, credentials, deployment recovery data
  and bulky trial evidence in ignored local notes. Expose any newly started UI
  or service to the LAN as required by localrule.md.
- Verify the deployed release identities and perform a disposable recovery trial.
  Leave the run/incident view and evidence ready to open before asking a person
  to inspect behavior; complete agent-operable setup and checks first.
- Write `report.md` with kept/adapted/removed implementations, contract changes,
  trial evidence, deployed versions, recovery limits and agent_session follow-up.
  Update developer docs and ignored environment notes; commit/push final changes.

## Definition of done

- Workflow, Git and executor evidence determine monitoring; Front and legacy
  agents are unnecessary and may cease working without blocking this phase.
- Intentional holds and normal waits are respected, including across restarts.
- The motivating interruption is detected, investigated and recovered through
  the deployed communication/execution path without manual restart.
- Recovery preserves work, has explicit ownership and bounded retries, and is
  verified by subsequent evidence rather than a sent request.
- Observer and executor outages are visible through an independent path.
- Reusable implementation remains where useful; obsolete assumptions are removed.
- Session reuse can be added without redefining work identity or monitoring
  obligations, and unresolved limitations are explicitly reported.
