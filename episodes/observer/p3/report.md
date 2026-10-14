# observer p3 — workflow observation and recovery: final report (2026-10-14)

Observer now watches workflow runs from WFE's records, Git and executor evidence. An interruption is detected, investigated by a recovery agent through Zulip, continued through a validated WFE operation, and recorded as restored only when later observations show work. The motivating failure (project_migration p2 step 3a: a background retry killed with the session, the run held until someone resumed it by hand) now recovers with nobody resuming it by hand — shown in a private trial on the real realm and once in production. Step reports: [report1](report1.md) … [report7](report7.md).

## What changed

**WFE (agdevworld `314c98b`, deployed)**

- `wfe observe` / `GET /api/observe` (`ag.wfe-observation.v1`, `docs/observation.md`): read-only. It covers every run with a uid that tells creations apart, source state and consistent reads, situation and holds (human or not), attempts, the process group's CPU, log facts (session id, an open tool call with its declared timeout, the harness's end of the session, background tasks stopped with it), queue position and the work ahead, waits with their responsible party, the agent's last report, recoveries, and publication and integration problems, plus the executor's heartbeat and slot.
- `run.recover` (`wfe run recover`): a recovering agent continues an `interrupted`/`unknown` hold or a continuation nothing queued.
  - Refused: a person's stop, cancellation, an open attempt or a live process, a stale attempt or sequence, a queued job, and more than 2 recoveries since the person's last instruction.
  - Recorded with the incident, the ended attempt and the recovering run's own agent and attempt.
  - The next attempt's brief says it was continued by recovery.
  - Not offered on the HTTP ops route.
- `intake.agents`: named agents may open requests in named workflows. Observer opens `agops/recover`.
- `wfe run wait --deadline`: external waits declare when they become doubtful.
- The run view and docs say an interrupted run may be continued by recovery.

**Executor (agautolab)** — unchanged code. The `wfrun` guide says an interruption is held until a recovery agent or the person looks, and that a continued attempt's brief names the recovery.

**Observer (pj-agdev/agobserver)** — `wfmonitor`, `wfclassify`, `wfstore`, `wfpost`, `wfrecover`, `wftriage`, `watchdog`, `retire_legacy`; processes `com.agdev.agobserver` and `com.agdev.agobserver-watchdog`.

**Recovery recipient** — the `agops` project (`developer/pj-agops`) with `recover` (the recovery agent's workflow and guide) and `check` (a disposable recovery-path check).

## Kept, adapted, removed

| Implementation | Fate |
| --- | --- |
| WFE `runSpace`, `runRefs`, `readRun`, `ExecStore.runView`/`executorState`, `activityOf` | kept; reused by `wfe observe` |
| WFE `activity.ts` | adapted: `logFacts` (session, open tool call, harness end, background tasks) |
| WFE intake, `ExecStore.resume` | adapted: agent requesters; `recover` beside the person's `resume` |
| Observer `monitor.py` (2227 lines: `#front` discovery, traces, serving probes, Front asks, conversation rescue rules) with `triage`, `health`, `review`, `review_status`, `hold`, `disposition`, `withdraw`, `trial` and 11 test files | removed |
| Observer `triage` (bounded local-model judgment, prompt/input kept beside the transcript) | adapted into `wftriage` |
| Observer `notify` read-back / `destination` three-valued resolution | pattern reused in `wfpost` (journal before send, marker read-back) |
| Observer watch entrance (`listener`, `intake`, `worker`, `observe`, `notify`, `anchor`, `destination`, `store`), `argue` | kept as a library, not started |
| agentroom `watchdog.py` | relocated to `agobserver.watchdog`, its own process; removed from the relay |
| `monitor-health.json` | replaced by `workflow-health.json` (`agobserver.workflow-health.v1`) |
| Old incidents (7) and watches (16) | archived with an inventory; none open, nothing replayed |

## Trial evidence

| Trial | Where | Result |
| --- | --- | --- |
| Lost background retry (+ lost post answer, duplicate intake delivery, Observer restart) | matrix S1 | one incident, one assignment, one recovery run; `restored` in 10 s (stub agents) |
| Long foreground command, external wait, queue wait | S2 | no incident |
| Person's hold and unanswered question while another run proceeds | S3 | no incident |
| Live-but-silent attempt, then process gone | S4 | `uncertain`, one local judgment (`stalled`), no writer while alive; `restored` after the kill |
| Answer queued behind healthy work vs lost queue entry | S2/S5 | the first waits; the second is `continuation` → `restored` |
| Executor outage | S6 | one incident naming the affected run, owners told once, `resumed` |
| Checkpoint failure | S7 | `publication` → recovery retried publishing, failed, handed off; no re-run, no acceptance; published after the remote returned |
| Person resumes / cancels during investigation | S9 | WFE refused the stale recovery (`recover-owned`, `run-cancelled`); `manual`, `cancelled` |
| Missing copy; observation command failing | S10 | told to the owners, never a rescue; health `source: failed`, nothing concluded |
| Real recovery agent, real Zulip delivery and wakeup | private area on the realm | restored in 3 min 28 s; the agent read the session log and concluded the backgrounded `curl` never ran; $0.69 |
| Deployed path | production `agops/check/run-001` | restored 3 min after a deliberate SIGKILL; $0.49 |

Model calls in the matrix: 1. False positives: 0. Interventions: only the injected faults and the simulated person's acts; none in the two real-path trials after the request. Defects the trials found and fixed:

- a lost queue entry had no recovery act;
- an escalated uncertain attempt confirmed lost did not move to recovery;
- an unreadable copy's incident never closed;
- a lost attempt was requested before its hold existed, naming no attempt;
- a completed recovery run was invisible to acknowledgement;
- posts waited a look;
- a bare node start counted as restoration;
- a test fake predated freework channels (pre-existing).

## Recovery limits

- **Executor and service failures have no automatic recipient.** A recovery run needs the executor. Observer and the watchdog tell the owners by DM, with the affected runs and where to look.
- **A live but silent attempt is never given a second writer.** Recovery waits until the process is gone. Stopping a hung process stays a person's act (`wfe run stop`).
- **Budget.** Two recoveries per person-originated control, then the person decides. Two assignments per incident, then the owners. A recovery run is never itself recovered.
- **Restoration is a later report by the run's agent.** A process may still have caused external effects the recovery agent could not see. Its reason is recorded as its finding, not as a fact.
- **Observer looks every 60 s.** Detection of a hold takes up to a minute; the uncertain path takes 15–30 min by design.
- Zulip outages delay requests (they are journaled and found by marker) but never double them.

## Follow-up for agent_session p1

- Keep the attempt (execution interval) as the unit of budget and of incident identity. Two intervals of one session are two attempts (tested).
- Record the resumed session id in `attempt.begin`, so the observation reads it from the record rather than from the log.
- Keep wait deadlines on the waits (`--deadline`), not on the session.
- A retained session does not keep background jobs alive; the `wfrun` foreground-wait rule stands until the job-lifetime contract changes.
- Delegation will add parent/child run references. Observer infers none meanwhile.

## Open

- The executor's own stop of a hung process and a recipient for infrastructure repair are not automated (by design in p3).
- `agentroom/progress.py` and the old ops board still read the archived `monitor-health.json` for the legacy progress panel, which now shows Observer's old monitor as missing. That panel belongs to the agents being retired.
- Front and the other legacy agents are untouched and unnecessary to this path. Their retirement is outside p3.
