# Failsafe p6 ex1 — explicit work relations and shared retirement decisions

## Goal and approach

Close p6's two remaining consistency gaps: a long conversation can turn a
citation into adopted work, and Observer's retirement decision can disagree
with Front Room. Make work ownership and outstanding obligations stable across
history limits, cleanup comments, and restarts.

Read `../report.md`, `../report1.md`, and `../report6.md`. This is a private
experimental, breaking-change phase. Backward compatibility is unnecessary;
implementers choose record formats, commands, and refactoring scope. Give agents
usable operations and evidence, with minimal additional restrictions. Existing
records can be migrated once rather than supported through permanent heuristics.

## step1 — reproduce the gaps and define the decisions

- Inspect current code and deployment. Before service work, use Nautobot or
  `nctl status` / `nctl drift` and the ignored environment notes; see
  `pj-clusterintent/nctl/README.md`.
- Reproduce the citation case at 199, 200, and 201 posts, and with a bounded
  history missing its beginning. In `pyagag/src/agag/trace.py`, `_reference`
  returns false when `len(messages) >= HISTORY`; the surrounding traversal
  then treats the relation as adopted work. Raising the limit only postpones
  the problem.
- Trace o11450, o11522, and o14251 against Observer's `retired.json`, their
  actual conversations, and the panel. P6 reports an unanswered forge plan,
  a routine without an end record, and a stray post. Establish what each
  retirement meant and whether any real repository work remains.
- Correct the baseline description: agentroom reads `retired.json` and passes
  retirement metadata, but progress does not interpret it as a disposition.
  The gap is shared meaning, not simply a missing file read.

Starting points: `agag/{trace,selfnote,chat,progress,holds}.py`, root-note writers
and readers, Front's `agrun adopt` and callback routing; Observer's
`hold.py`, `retired_now`, and tracking; agentroom's progress and completion paths.
Follow current file ownership if these entry points have moved.

## step2 — record what a conversation relation means

- Represent delegation/adoption separately from citation. Keep the return
  address for a reply distinguishable from ownership of the referenced work:
  a cleanup comment may need an answer without adopting the entire request.
- Have ordinary delegation tools record the intended work relation. Comment
  and reference operations record references; deliberate adoption/move tools
  change ownership explicitly. Choose defaults that fit normal agent usage
  and document how an agent inspects or corrects a relation.
- Use stable message anchors and shared writers/readers. Renaming, resolving,
  archiving, or reusing a topic name should not change which request is meant.
  Apply the relation consistently to trace traversal, callbacks, acceptance
  scope, completion scope, Observer, and progress aggregation.
- For existing ambiguous records, obtain the actual conversation beginning
  from the mirror or a targeted read and record the resolved relation. If the
  evidence is unavailable, expose the relation as unknown with a way to resolve
  it. Missing history must not manufacture an ownership edge or silently erase
  established work.
- Remove the history-length heuristic after migration. Preserve explicit
  adoption and genuine delegations while dropping incidental citations such
  as p6's cleanup link #15357.

## step3 — separate monitoring suppression from ending a request

- Replace the ambiguous retirement meaning with two explicit dispositions:
  **monitoring suppressed** (work can remain open) and **request ended**
  (completed, cancelled, or withdrawn, according to the decision and evidence).
  Reuse p6's conversation-record and hold mechanisms where appropriate.
- Record the affected request/work, decision maker, reason, supporting evidence,
  and scope. Trace, Observer, and Front Room should consume the same record.
  Suppressed unfinished work stays visible as such; ended work leaves the
  active queue without claiming successful completion when it was abandoned.
- Provide agent/operator operations to inspect, apply, and reverse these
  dispositions. Use the existing completion/cancellation paths to reconcile
  dependent work and routine endings. Describe any remaining child obligation.
- Define what new substantive activity does: a later request or result must
  not inherit an old blanket exemption. A restart or bookkeeping-only receipt
  should not accidentally undo the decision. Choose an explicit scope or
  activity boundary and test it.
- Make repeated or interrupted operations converge. Replace the private-file
  retirement path once its decisions are represented by the shared records.

## step4 — migrate and settle the existing cases

- Reconcile o11450, o11522, and o14251 using their recorded intent and the
  new tools. The user has delegated cleanup of completed/disposable test work;
  use that authorization for routine dispositions. A genuinely new decision
  about substantive unfinished work should be identified specifically.
- Record success only where the requested outcome was reached; use cancellation
  or withdrawal for abandoned trials. Check for unfinished repository changes
  before declaring a case settled.
- Migrate affected relation records, including long conversations, and replay
  the Front requests to detect changed ownership, callback destinations, and
  completion scopes. Explain each meaningful change.
- Preserve p6's repaired receipts and scoped holds. o8512's missing historical
  hold backfill is documentation debt, not an open approval: document it without
  inventing an old user statement or creating a new live hold.
- Expose any unresolved evidence gap explicitly. Record repairs in `report.md`
  so future occurrences can use the same tools rather than manual file edits.

## step5 — validate, deploy, and report

Use focused tests and transcript replay, then short live trials. No long study
run or full replay of every failsafe phase is needed.

| Scenario | Required result |
|---|---|
| Same citation below/at/above 200 posts, or truncated history | No accidental adoption; missing evidence is explicit. |
| Delegation, citation with a reply, and deliberate adoption | Correct work owner, callback destination, and completion scope for each. |
| Rename, resolve, archive, or reused topic name | Relation still identifies the intended request. |
| Monitoring suppressed on unfinished work | Panel explains the suppression; work remains open; Observer follows its scope. |
| Completed, cancelled, or withdrawn request | All readers agree on the actual outcome and remaining child obligations. |
| New substantive activity after a disposition | New obligations are visible and receive appropriate monitoring. |
| Repeated operation or restart during recording | Same final disposition, without duplicate execution or a lost decision. |

- Run affected shared-library and consumer regressions. Retain p6 checks for
  unreceived results, result-bound acceptance, receipt recovery, and human holds.
- Demonstrate a cross-request comment with a reply and a legitimate delegation
  through Front. Exercise both disposition types and subsequent new activity.
  Compare trace, Observer, and fresh Front Room data before and after restart.
- Include a live archived-source case using an account permitted to perform
  the operation; keep a focused listener regression for the timing boundary.
  Record actual environment limitations separately from product restrictions.
- Deploy the required pins to intended consumers, checking the VM deployment
  explicitly. Document deliberate exclusions and their impact; a shared-library
  push alone does not update running services.
- Remove obsolete paths/guidance and trial faults. Write `report.md` with the
  final relation/disposition semantics, migrations, validation, interventions,
  deployment coverage, and limitations. Update development docs and commit/push
  changes in their owning repositories.

Completion requires the table to pass, the three legacy retirements to have
explicit dispositions, and the readers to agree after restart. Clearly label
developer intervention in live trials rather than counting it as autonomous
agent recovery.
