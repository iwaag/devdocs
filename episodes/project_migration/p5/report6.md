# project_migration p5 — Step 6: deployed workflow trials

Updated 2026-10-16 during execution takeover. Steps 1–5 had been implemented
and the initial trials had already run before takeover. This report checks
the actual records rather than treating report5's preparation status as
current. Raw evidence and costs are retained in ignored
`pj-agdev/.local/project-migration-p5/` notes.

## Trial evidence

| Trial | Record | Observed result |
| --- | --- | --- |
| T1 consultations | cagent `consult/run-001`, `consult/run-002` | Completed, accepted and integrated. Answers cite source observations and ages, distinguish unknown information, and perform no collection/change. Acceptances are attributed to Omni Agent in the record. |
| T2 reports and delegation | cagent `report/run-001..003`; agops `cluster-report-check/run-001` | Completed and accepted; integration recorded. Direct report, duplicate relation and delegated report all exist. The delegated child's decision is from the parent agent, whose received report records a satisfied disposition. |
| T3 scan | cagent `scan/run-001` | Completed, accepted and integrated. The unenrolled trial guest is unavailable; the artifact retains `complete: false`. Other hosts were observed; no repair was dispatched. |
| T4 guest creation | cagent `change/run-001`; agops `cluster-change-check/run-001` | Completed, accepted and integrated. The child's plan has the Developer's agreement, batch digest and exact host scope. The parent agent's result receipt is separate from that authority. |
| T5 interruption/recovery | cagent `change/run-001`; agops `recover/run-003`, `recover/run-004` | Prepared script killed only the attempt process group after the reconcile entry opened. Backend operation continued; recovery inspected the records and resumed verification. The single create operation has evidence; no duplicate creation is claimed. |
| T6 retirement/cleanup | cagent `change/run-002` | Developer agreed the revised plan and performed/confirmed guest destruction. Fresh observation verified removal. Takeover dry-prune was eligible; Omni Agent executed prune once, with operation evidence and correct performer attribution in `e0006`. Final drift/actual no longer contain the guest. Omni Agent reviewed the concrete result as trial requester; confirmation, completion and integration are recorded. |
| T7 dependency-host change | cagent `change/run-003` | After the Developer removed D9, the actual tool invocation follows the ordinary agreed scope. Fresh plan is empty; tool returns noop, dispatches no effect, and subsequent drift is converged. Omni Agent reviewed the concrete result as trial requester; confirmation, completion and integration are recorded. |

## Takeover corrections

- Report5's original "not started" statements were historical; retained
  evidence shows the initial trials and decisions already happened.
- Self-dependency refusal was removed at the Developer's instruction, with
  30 passing tool tests and current guides deployed to waiting run copies.
  The old prepared handoff is retained as unexecuted history.
- The retirement agent incorrectly inferred that missing `cagentctl prune`
  made all state pruning human-only, even after the Developer explicitly
  instructed execution. The approved design permits direct nctl use. The
  takeover completed authorized cleanup and recorded `performed_by: Omni
  Agent`, preserving the person's separate guest-destruction evidence.
  Existing unused person handoffs are marked not-started/superseded, not
  falsely person-confirmed. A guide fact now cites this observed failure.
- An explicit `expect: []` was incorrectly treated as no action constraint.
  The takeover changed the check to distinguish an empty list from an
  absent expectation, and added a regression proving new actions are
  refused before dispatch. The live dependency-host trial really had no
  actions; no unintended action was observed.
- The shared records index had already caused concurrent integration
  conflicts; the earlier executor removed it from tracking. It remains
  disposable/advisory. Initial manual integrations and the cleanup
  intervention are evidence of operator assistance, not autonomous success.

## Validation and close-out

Tool tests: **31 passed** (including the added empty-scope regression). Ansible conformance: **4 passed**. Retained legacy
TLS/client conformance, with its historical client source moved out of the
retired deployment role: **23 passed**. No live operation was repeated just
to test a refusal or unknown result.

Acceptances and integration status are read from WFE records. All eleven cagent runs now pass WFE record checks with no warnings; all
have recorded acceptance and integration. The new post-cleanup scan and
freework smoke check are covered in report7 and the final report. Costs use at most the last result per
attempt; killed/unfinished attempts without a final cost are not assumed
free.

Omni notes: completed authorized prune and repaired execution guidance for
cagent — handoff candidate. Earlier trial integrations also required Omni
intervention; see the private evidence and final report.

Final verification also ran the required clean Nautobot runtime gate:
**263 cases passed**, with isolated test-state cleanup. It changes no
production schema. Trial confirmations during takeover were recorded as
Omni Agent technical reviews under the execution handoff, preserving the
Developer's actual earlier plan approvals and destruction confirmation.
