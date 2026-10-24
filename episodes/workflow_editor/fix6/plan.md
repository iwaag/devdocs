# Workflow editor fix6 — delegated result review and correction

## Goal and evidence

Make delegated work reach the person in a form they can inspect, and let
the parent return an incomplete result to the same child run for correction.
Implement the direction agreed after [preresearch.md](preresearch.md).

The motivating case is conversation `20261024-160217-forusr`:
misc/freework/run-001 through run-004 completed without a presentation;
run-005 presented only after an explicit request. Some children saw only
the tail of fix5's reminder, one used untemplated work nodes and received
none, and the parent accepted results without a presentation reminder of
its own. The parent did act on clear discussion-activity reminders. The
hypothesis is that evidence and a concrete correction operation at the
parent's receiving point will improve delivery. The trial must test that
hypothesis rather than assume it.

## Working conventions and scope

- Follow `localrule.md`, `devpolicy/styles.md`, `devpolicy/terms.md` and
  `devdocs/README_DEV.md`. Generated files are English; communication with
  the Developer is Japanese. Keep machine-specific details in ignored
  local notes, including `pj-agdev/.local/devenv.md`.
- Execute implementation, checks, commit/push as appropriate, deployment
  and deployed verification without an approval milestone. The agent may
  perform the necessary local and LAN operations. Human review of an
  in-system result is an observed workflow fact, not permission to develop
  or deploy fix6.
- Backward compatibility is not required. Preserve historical evidence;
  use explicit migrations and matching recovery backups when necessary.
- Keep fix5's advisory convention: applicable reminders refuse the proposed
  operation before effects, and the same operation with `--yes` may proceed.
  This is not a human approval gate. Correctness errors remain errors.
- Include parent review, same-child correction, dispatch/brief information,
  warning wording, and coverage of substantive untemplated work.
- Do not add a general natural-language policy engine, automatic correction
  work, mandatory presentation for every result, or a new approval process.
  Do not change integration into an operation gated by the parent's
  disposition or human review.

## 1. Establish the evidence model and shared assessment

Trace the existing operation boundaries before editing: delegation
dispatch/take-up/disposition, child completion relay and result manifests,
conversation result/synthesis posts and combined `--completing` operations,
run adjustment/reopening, executor continuation and integration. Check
CLI, HTTP and remote-worker routes, not just CLI formatting.

Extend the shared reminder assessment to consume a child's effective
definition, frozen catalog, run records and result evidence. Separate
observable facts from the catalog's expectations and the agent's judgment.
Catalog expectations are prose today; do not claim to compile arbitrary
expectations into executable rules. Implement the following bounded checks
and show the relevant frozen expectations alongside their evidence.

| Check | Evidence and interpretation |
| --- | --- |
| Presentation missing | Completed substantive work or registered viewable artifact candidates, with no relevant presentation or planned presentation. Identify actual paths and links; a filename extension is a signal, not proof that a presentation is required. |
| Person's review not recorded | `accepts: person`, but no traceable answer from the person concerning this result. A confirm node or decision alone is insufficient. Follow parent answer references where the parent relayed the person's words; report missing evidence as unknown, not a proven violation. |
| Presentation still serving | Recorded serving state, review state, cleanup plan and any recorded request to remain available. Distinguish unfinished review or requested availability from forgotten cleanup. Never stop a page merely because this check ran. |

Retain the child's checks. Expand missing-presentation coverage beyond the
`work` template using artifact and structured work evidence; do not classify
every untemplated `do` node as work. Reporting, documentation, consultation
and coordination must not acquire presentation obligations merely from
their node type. Keep presentation coverage tied to relevant work/result
revisions, rather than assuming any earlier presentation covers all output.

Bind each assessment to the delegation, completed event, manifest digest,
child record position, definition and catalog snapshot used. Inspect current
state for later repairs or a stopped presentation, but keep it distinct from
the facts frozen with the delivered result. Expose unavailable evidence
explicitly. Revalidate the result identity at mutation time; a stale result
must not be disposed as if it were current. Choose snapshot fields and
cross-run concurrency handling after tracing the existing locks and relay.

## 2. Put the assessment where the parent receives and reports

Use one assessment for:

- `delegate show`, take-up output and the resumed attempt's brief;
- `delegate dispose --disposition satisfied`, before closing the delegation;
- the parent's outward result/synthesis post, including a post that also
  completes a node, whether or not the parent has work nodes of its own.

At report time, cover the relevant delegated results, including ones already
disposed as satisfied. Include delegation and result references so the agent
can tell which child needs attention. An earlier `--yes` is not a permanent
exemption, but a repaired omission must disappear on reassessment. Avoid
repeating identical informational notices without a relevant state change.

Merge the parent's own and delegated-result reminders into one response.
Evaluate before recording a disposition, posting a message, completing a
node or queuing downstream work. Combined operations must not leave partial
effects. Support equivalent HTTP input and forward it from remote workers;
user-facing clients display reminders and allow explicit continuation,
without silently setting `yes`.

Repair advice must be appropriate to the parent: inspect the child's
evidence, relay an actual answer, request same-child correction, or judge
continuation appropriate. Do not tell the parent to impersonate the child
or fabricate a review. Incoming human answers are always recorded normally.

## 3. Add an explicit same-child correction operation

Add a dedicated operation, provisionally:

```text
wfe delegate revise <parent-run> <delegation> --instruction "…"
```

Keep `dispose rejected` as closure/replacement; do not overload it with
reopening. The new operation records the result being returned, the reason
and instruction, and requests continuation of the existing child. The child
agent designs its additional pending nodes using the ordinary adjustment
mechanism. It does not repeat completed work merely to obtain a new result.

Define and implement the complete transition:

1. The parent returns the current completed result before closing the
   delegation. Validate attempt ownership, relation and result identity.
2. Record a durable correction request and move the parent to waiting.
   Deliver/reconcile it through the existing transport and receipt model.
3. Reopen the same completed child for an executable correction attempt.
   A stored instruction alone is insufficient today. Make the state between
   receiving the request and adding work explicit so the scheduler neither
   loses the continuation nor announces the old result again.
4. The child takes up the instruction, adds and performs the missing work,
   presents it, handles feedback and plans cleanup as appropriate.
5. The next completion produces a new result event and manifest. The parent
   takes it up and assesses that version before disposition/reporting.

The initial operation applies to an open delegation with a bound completed
child and a current result. Closed/replaced delegations, cancelled children,
and incompatible active attempts receive explicit state errors; do not
silently undo terminal decisions. A held parent does not launch its own
continuation until resumed. Parent cancellation must not implicitly cancel
the child; preserve the existing explicit cancellation contract.

Provide idempotent receipts and recovery across parent record, delivery,
child instruction/reopening, queue insertion and result relay. Retrying
after a crash must create neither duplicate correction attempts nor new
child runs. Make competing correction/disposition requests and stale
completion events deterministic. Extend record replay, observation and
views so waiting for correction is visible and owns a continuation.

Existing acceptances and integrations stay historical facts bound to their
original revisions. Display any prior integration in the assessment. Do not
undo it or imply the corrected result was previously reviewed. Changes to
the result follow the existing new-revision acceptance/integration rules;
test that an old acceptance does not cover corrected files. Changing when
integration is permitted is outside this episode.

## 4. Improve dispatch context and reminder wording

At dispatch, show the target's current catalog expectations, `accepts`, and
who relays questions/presentations to the person. This is information, not a
warning or permission requirement. The target catalog at dispatch is a
preview; assessment uses the snapshot actually supplied to the child.

Make the child's brief explicit about the person on whose behalf the parent
asked and the parent's relay role. A request from an agent does not imply
that presenting a result ends at a file-path report to that agent. Preserve
the original request and its identity; do not silently rewrite it.

Shorten reminder text. Put a compact summary of the observed omission and
the concrete repair at the end as well as near the beginning, so `tail -3`
still carries the reason to act. Remove the presentation-specific escape
sentence about "records that need no viewing"; keep the common explanation
of `--yes` and uncertainty. Preserve structured reminder data for clients.
Test full output and truncated output with multiple simultaneous reminders.

Update command help, delegation/runs/presentation documentation, run views
and the shared wfrun guide. Keep command usage in help and shared facts in
one guide; cite this trial history beside guidance derived from it. Inspect
ponderer and Front guidance for contradictory reporting instructions and
change only what is necessary, without distributing duplicate checklists.

Primary code areas are `pj-agdev/agdevworld/workflow_editor` shared models,
reminders and reducers, server delegation/run/conversation operations,
continuation/relay, CLI/API, remote routing, views and behavioral checks;
executor integration and shared guidance are in `pj-agdev/agautolab`.

## 5. Verify behavior and failure recovery

Add focused tests for the changed contracts and run the affected projects'
required checks. Use fixtures representing the recorded failures rather
than modifying the historical runs to pass new checks.

- Work-template output, untemplated substantive work, screenshots/video,
  and a defensible report-only result; distinguish observations from
  judgment and avoid warnings caused solely by coordination nodes.
- Parent without work nodes receives the child's missing-presentation
  reminder at disposition and outward reporting. An already-satisfied
  delegation remains assessable; a repair removes its obsolete warning.
- A confirm node, an agent's own answer, a person's relayed answer and a
  decision with no supporting human answer are distinguished correctly.
- Presentation linked to the result, an older unrelated presentation,
  requested changes, requested continued availability and verified cleanup
  produce accurate assessments.
- Warned operations have no partial effects; repair and `--yes` work across
  CLI/API/remote-worker and combined completion paths. Stale result and
  ownership errors cannot be bypassed with `--yes`.
- Same-child correction survives retry and crashes at each handoff, resumes
  execution once, preserves completed work, produces a new result, rejects
  stale disposition and leaves parent/child waiting states observable.
- Exercise prior integration, concurrent disposition/correction, closed
  delegation, cancellation and held execution. Replay and recovery retain
  both original and corrected result identities.
- Warning tails retain concrete facts and a repair, rather than only the
  direction to use `--yes`.

## 6. Deploy and run the behavioral trial

Before inspecting or changing running services, obtain their state through
`pj-clusterintent` Nautobot or `nctl`, following
`pj-clusterintent/nctl/README.md`. Consult ignored local environment notes
for established release paths. Prepare backups and a matching code/data
recovery procedure for new journal events or store migrations. Deploy WFE,
executor changes/guidance and applicable worker pins; check coordinator and
remote-worker operation through the deployed versions.

Run a real facilitated conversation delegating a small viewable deliverable
without mentioning presentation or cleanup in the request. Observe the
child's actions, the parent's assessment, any correction request, the
presentation reaching the conversation, feedback and cleanup. If the child
presents on its first attempt, count that as successful delivery but do not
claim that it proves the correction path. Separately exercise a controlled
completed-child omission fixture through the deployed correction path.

Use an identified check actor for synthetic feedback; do not impersonate the
Developer. The Omni Agent must not perform the missing presentation on
behalf of the child to make the trial appear successful. If intervention is
necessary, record it and the handoff candidate under the existing policy.
Record immediate or repeated `--yes` use and failure to repair as results,
not as reasons to add approval gates. Improve demonstrably unclear repair
paths and repeat only the checks affected by those changes.

Verify the resulting presentation and changed views from another LAN host;
do not constrain them to localhost. Leave a concise entry point in the
conversation/run view so the Developer can inspect the deployed behavior
with minimal operations. A review page may remain available when explicitly
requested, with its cleanup ownership visible; disposable test pages must
not become forgotten services.

## Completion and report

Implementation is complete when the shared assessment, parent boundaries,
same-child correction, context and wording changes are deployed and their
behavior/recovery checks pass. Evaluate the behavioral hypothesis separately:
report whether the unprompted trial delivered a presentation and whether a
parent correction worked, without claiming reliable agent compliance from
one successful example or from a synthetic fixture alone.

Write `report.md` with changes, checks, deployed verification, trial and
result references, observed limitations and recovery references. Update the
relevant development documentation and ignored local environment notes.
Keep host addresses, credentials and machine-specific release/evidence paths
out of tracked documents. Do not leave an approval request as the final
implementation step; leave a deployed, inspectable result and an honest
account of any remaining failure.
