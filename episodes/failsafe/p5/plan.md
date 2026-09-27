# Failsafe p5 — wait correctly and finish delegated studies

## Goal and decisions

Make overlapping study requests reach their requested outcomes without human
nudges: recognize legitimate queues, accept reviewed work consistently, finish
routine runs and deliver their reports, and return sage refresh results to the
right request. Use the progress panel to observe the path and expose defects.

Read `../../progress_panel/p1/report.md` and `report5.md` in that directory,
plus `../p4/report.md`. Four study runs completed in progress_panel p1, but only
B growbox completed without a human step after the request. B worldtrend still
needed an acceptance and a nudge to end its run. These are the baseline cases.

This is a private experimental, breaking-change phase. Backward compatibility
is unnecessary. Implementers choose mechanisms, schemas, and refactoring scope;
remove superseded paths and guidance. Prefer shared evidence and useful tools
over prescribed agent scripts or extra approval machinery.

- Front may review and accept results of ordinary studies it was entrusted to
  execute. A request explicitly reserving final approval for the human waits
  for that approval. The initial execution request is not acceptance of a
  result that has not yet been shown.
- Acceptance depends on who holds the decision and what result they reviewed,
  not on which agent writes the record. Keep task acceptance, mission
  acceptance, routine completion, and final reporting distinguishable.
- Serial listeners are acceptable. Scheduler parallelization and universal
  execution-health instrumentation are outside the default scope.
- Older held requests remain separate decisions, not prerequisites for p5.

## step1 — reproduce the gaps and establish evidence

- Check current code and deployment through Nautobot or `nctl status` /
  `nctl drift`; see `pj-clusterintent/nctl/README.md` and ignored environment
  notes. Reconfirm which reported issues remain before changing them.
- Replay progress_panel trial B: a healthy task blocks another request's plan
  for over five minutes, yet Observer raises `unacknowledged` (#13702).
- Reproduce the acceptance asymmetry: Front's own agreement is refused by
  `agentchat accept` but accepted when autolab records it (B growbox #13804).
- Trace B worldtrend's workplan opened beside its routine run, the correct
  `unheld` incident (#13707), and Front's ineffective self-post (#13715).
- Exercise an interrupted `agrunfinish` between end-record creation, report
  delivery, delivery receipt, and topic resolution. Code inspection suggests
  an existing end record can skip unfinished delivery on retry; this is a
  hypothesis to verify, not a demonstrated live failure.
- Capture the fixed-topic sage refresh returning to an earlier request and
  test whether two runs of the same study can borrow each other's refresh.

Starting points: `pyagag/src/agag/{trace,progress,acceptance}.py`, especially
`stall_candidates`, `queue_behind`, and `accept_mission`;
`pj-agdev/agobserver/src/agobserver/`; agautolab's `zulip_listener.py` and
`mission_done.py`; agfront's `runfinish.py`, listener, and desk/routine guides;
archsage's `sagesync` writer and the shared progress reader. The old plan's
reference paths are hints; follow current ownership if files have moved.

## step2 — share queue evidence with Observer

- Extract or reuse queue interpretation below the UI so the panel and Observer
  agree on which serving a request waits behind. Include evidence source and
  freshness; an unclosed conversation alone is not confirmed process health.
- Distinguish a healthy busy owner, an uncertain/stopped preceding serving,
  and an idle listener that has not picked up eligible work. Keep the original
  queued time as well as changes to the preceding work.
- Defer recovery of legitimate queued work while continuing to assess the
  serving ahead. Detect a stopped blocker, failure to advance after it ends,
  and prolonged lack of service despite activity elsewhere. Choose practical
  bounded checks; simply raising the five-minute threshold is insufficient.
- Keep recovery directed at the actual blocked/stopped work. Verify that an
  expired health observation cannot indefinitely excuse a queue or trigger
  competing execution.

## step3 — make acceptance independent of the recording route

- Represent the requester/decision holder separately from the recorder using
  existing mission relations and acceptance evidence where possible.
- Allow Front to record its own post-result decision when it is the delegated
  requester. Autolab recording the same decision must produce the same result.
  A human-reserved decision remains pending until that human gives it.
- Verify the evidence belongs to the relevant work and follows the reviewed
  result. Preserve p4's binding to the shown result/checkpoint and completed
  integration; a worker's own completion claim is not requester acceptance.
- Make retries finish the same acceptance record and update the CLI, agents'
  tools/guides, and completion UI to the same semantics. Avoid sending another
  paid planning turn merely to acknowledge an already-recorded decision.

## step4 — complete and resume routine close-out

- Make a study's work discoverable from its routine run and origin request.
  Address the observed sibling-workplan case through explicit relations and
  corrected tools/guidance; do not infer ownership from similar topic names.
- Give Front an effective way to continue its own routine when a callback or
  recovery requires it. Reuse the listener's durable serving/start mechanisms;
  a self-authored "Resuming" post alone must not be reported as resumption.
- Ensure Front can finish the run from whichever conversation receives the
  final evidence, without a human reminding it to use `agrunfinish`. Keep agent
  judgment over whether the requested outcome was achieved.
- Make end record, report delivery, receipt, and resolution resumable across
  interruption. An end record does not imply delivery. Reuse existing journal
  and delivery mechanisms to complete missing steps without duplicate reports
  or re-executing accepted work; verify the self-finish and external-finish paths.

## step5 — bind sage refresh to the intended run and revision

- Replace the study guide's fixed callback topic with a per-run delegation or
  another explicit return relation. Archsage's answer must reach the request
  that asked for this refresh, including after rename or restart.
- Record the originating run/request and the repository revision actually
  refreshed. Validate freshness against the result integrated for that run;
  choose how to recognize a newer revision that includes the required result.
- Update the panel and other readers to use this evidence. Matching only the
  project and a time after acceptance is insufficient for overlapping runs of
  the same study. A refresh completed elsewhere may satisfy the stage only
  when its recorded relation and revision establish that fact.
- Verify repeated refresh/delivery attempts preserve the association and do
  not wake an old request or satisfy an unrelated run.

## step6 — validate autonomous completion and consolidate

Use short fixtures, transcript replay, injected clocks, and interruption tests
for iteration. Run focused regressions in affected packages, then deploy the
needed pins and validate the real path. No full rerun of every failsafe phase
is required.

| Scenario | Required result |
|---|---|
| Healthy serial queue beyond five minutes | Panel explains the wait; Observer issues no false recovery request. |
| Blocker stops, or finishes but queued work is not picked up | Bounded detection names the actual problem; no indefinite exemption or competing run. |
| Front vs autolab records the same acceptance | Same decision and final record; repeated calls do not repeat work. |
| Human explicitly reserves approval | Work reaches that wait and stays open until the human agrees. |
| Routine continuation and interrupted close-out | Real serving resumes when needed; end and report delivery complete once without a human nudge. |
| Two runs of the same study | Refresh evidence and replies reach the correct run and satisfy the correct revision. |

- Run two small existing studies through Front in separate conversations with
  overlapping lifetimes and delegated acceptance. Both must finish through
  accepted/integrated research, required knowledge refresh, run end, and report
  delivery without human input after the initial requests. Serial execution
  is fine; record actual overlap and queue waits separately.
- Repeat one study to verify its callback does not return to the previous run.
  Cover same-study overlap/reordered refreshes with a focused test or short
  live trial. Repeat the concurrent pair after fixes to show the result is
  repeatable on the final code.
- Retain one confirmed-stop check on operational timing for a health-covered
  owner: stop to recovery request within the existing 300-second goal. Use
  accelerated checks for other failure permutations and label them as such.
- Record every failed attempt, human response, developer repair, false
  intervention, duplicate execution/delivery, and trial cost. An assisted run
  is diagnostic evidence, not a successful autonomous completion trial.
- Mark resolved progress_panel review occurrences with their fix and evidence;
  keep unresolved ones linked to concrete follow-up work. If a review requires
  the Developer's final acknowledgement, leave that last action explicit.
- Remove temporary faults and obsolete code/guidance, update documentation and
  ignored deployment notes, and commit/push in the owning repositories.

Write `report.md` with the adopted acceptance semantics, queue/recovery timing,
close-out interruption results, callback/revision evidence, repeated live trial
outcomes, interventions, and limitations. Completion requires the table above
and both concurrent-study trials on the final implementation to pass.

Older holds are a separate follow-up: m11741 can first present and recover its
existing results before deciding further research; m8519 can be accepted after
reviewing its already-shown task 2 result. Neither is automatically accepted,
resumed, or cancelled by completing this phase.
