# Agent session p1 — session policies for workflow runs

## Objective and scope

Implement [braindump.md](braindump.md): compare fresh-session execution with
one retained session across a workflow run, using the RTS project for bounded
experiments. Make the two policies selectable in Detail, regardless of whether
session reuse proves cheaper. Measure cost, cache use, execution quality and
recovery behavior before choosing the operating default.

Build on [Observer p3](../../observer/p3/report.md) and workflow editor p9's
isolated run copies. Workflow records and Git remain authoritative. A retained
conversation is useful context, not a second work record. Zulip remains
inter-agent communication. Legacy Front/forge behavior and backward compatibility
are not requirements; preserve useful shared harness and execution code.

This phase covers the single-agent workflow executor, the two policy choices,
session persistence/resumption, observation, comparison and deployment. General
delegate nodes, cross-agent conversation sharing, automatic model routing,
advanced session-splitting policies and durable background-job infrastructure
are outside scope. Future delegated work must be able to occur between execution
intervals without requiring a parent process to stay alive.

Creating this plan does not start the experiment or alter running services.

## Starting implementation and design direction

- `agautolab/src/agautolab/wfexec.py` launches a new harness process per claimed
  attempt, provides the full contract/project guides/brief, and releases the
  execution slot when the process ends. Its default time budget is four hours
  per attempt. A talk question can end an attempt normally.
- `pyagag/src/agag/harness.py` owns harness-specific invocation and result
  parsing. Its current invocation interface has no explicit workflow session
  lifecycle. Inspect the installed harness behavior rather than inferring
  resume support from an existing command name or generic extra arguments.
- WFE records attempts, answers, holds, checkpoints, recovery and integration.
  Observer uses these records plus per-attempt process/log evidence. Session IDs
  currently come from logs; Observer already tests separate incidents for two
  attempts sharing one session ID.
- Observer does not yet automate stopping a live hung process or repairing an
  unavailable executor. Those known limits do not block this comparison.

Prefer **retained conversation, new process per execution interval**. End the
attempt while waiting for a person, release the slot, and resume the exact
session in a new attempt after the answer is recorded. Do not keep a process
alive merely to preserve a conversation. If the selected harness cannot support
this, report the limitation and assess an alternative explicitly; do not silently
equate a persistent process with session reuse.

| Identity | Meaning |
| --- | --- |
| run | One workflow execution and its retained working copy |
| session | Harness conversation state, scoped to that run and execution configuration |
| attempt | One active execution interval, with its own process, budget and reports |

## 1. Verify harness capabilities and establish the baseline

Read the current workflow/executor/Observer contracts and ignored environment
notes. Consult Nautobot or nctl before relying on local services, as required
by the workspace instructions. Inspect the actual RTS workflow execution profile
and installed harness version; begin with that profile rather than requiring
every supported harness to implement resumption at once.

- Use installed CLI help and current official harness documentation to verify
  explicit session creation/resumption, persistent storage, stream event fields,
  compaction, output usage and missing-session behavior. Record versions and
  references. Keep machine-specific paths in ignored notes.
- Run a small disposable capability probe: create a conversation, terminate its
  process normally, resume that exact conversation in the same run copy, and
  demonstrate retained context. Prove that another run does not inherit it.
- Check whether resumption appends or replaces prompts, replays old tool calls,
  changes model/settings, or produces cumulative versus per-invocation usage.
  Inspect how a session ID becomes known before a crash and how it is recovered.
- Establish the present fresh-session baseline, including full prompt size,
  cached/uncached input, output, elapsed time and actual reported cost where
  available. Existing fresh sessions may already receive prompt-cache hits.

**Completion evidence:** a capability table for the selected profile, a working
explicit resume probe, and a measurable fresh-session baseline. Unsupported
harnesses must be identified rather than silently presented as resumable.

## 2. Define and expose the two session policies

Use two initial choices (final names may follow the existing editor vocabulary):

- `fresh`: each attempt starts a new session from the run records and files.
- `run`: subsequent attempts normally resume the run's retained session.

Put the policy beside the workflow's execution settings in Detail and freeze
the selected policy into the created run, following the existing snapshot model.
Workflow edits affect future runs. Show the run's effective policy and any
session replacement in its execution details. Keep Simple uncluttered.

- Carry the setting through definition validation, serialization, CLI/API,
  claim, attempt settings and the executor. Both choices must have real behavior.
- Define omitted-setting behavior explicitly; use `fresh` initially to preserve
  the comparison baseline, not as a legacy compatibility layer. Choose the
  final default from the evidence in step 6.
- Expose capability failures clearly. An unsupported profile with `run` must
  not silently execute every attempt fresh and appear to satisfy the policy.
- Keep session policy separate from model/profile, workflow node type and
  documentation policy. Do not add per-node heuristics in this phase.

**Completion evidence:** a Detail selection survives save/reload and run creation,
is frozen on that run, and reaches its claimed attempt consistently via CLI/API.

## 3. Implement session ownership, durable identity and continuation

Extend the shared harness seam with explicit new/resume semantics for the
selected harness. Keep provider-specific arguments, stream parsing and storage
behavior there; keep workflow ownership and lifecycle in WFE/the executor.

- Bind sessions to a stable run identity, harness/profile and working copy.
  Always resume an explicit session ID; never use the most recent conversation
  from a directory or account. A recovery agent owns its own session.
- Persist the intended resume ID in `attempt.begin` when known and record the
  actual ID as soon as the harness supplies it through a guarded operation.
  First creation may not have an ID at claim time; represent that state rather
  than inventing an ID or relying solely on a successful final result event.
- Keep harness storage paths/transcripts host-local. Store portable identifiers
  and replacement provenance with the run where appropriate. Git commits alone
  do not preserve the harness's conversation store.
- Journal session creation/attachment sufficiently to handle executor restart
  after launch but before recording the ID. Reconcile without launching another
  writer. Ambiguous attachment is explicit, never guessed from account history.
- Bootstrap a new session with the common contract, project guides and current
  run facts. Resume with the new answer/instruction, current attempt identity,
  relevant changes and the command to read the authoritative run state. Avoid
  re-appending the entire unchanged bootstrap context on every continuation.
- Revalidate the effective execution configuration and guide revisions at each
  claim. Record when a configuration change requires session replacement; never
  assume an old conversation has learned new instructions automatically.
- Renew `WFE_ATTEMPT`, process identity, log boundaries and time budget for every
  interval. Old conversational references to an attempt cannot authorize writes
  under that old attempt. Retain current WFE fencing and receipts.

Keep the agent guides factual: the session may remember previous work, but
current records determine what was answered, accepted, held or completed.
Records must remain sufficient for a fresh session to continue the work.

**Completion evidence:** one run performs several attempts in the same explicit
session, another run has an independent session, and fresh policy uses a new
session each time. Restart and early-crash cases preserve single-writer behavior.

## 4. Preserve waits, recovery and Observer semantics

At a question or intentional hold, end the active attempt and release the slot.
An answer queues the next interval through the existing mechanism. A retained
session consumes no execution slot while another run works. Apply the existing
attempt time budget only to active execution, not to the retained session's age.

- Observer obtains recorded session identity alongside attempt identity, using
  logs as supporting evidence. Keep activity, open tools, end events and usage
  scoped to the current attempt even if a harness transcript spans sessions or
  includes replayed history. Session creation time is not a stall timer.
- Exercise answer waits, intentional holds, completion, cancellation, queued
  continuation and Observer-triggered recovery under both policies. Existing
  recovery limits do not reset when a session is replaced.
- Under `run`, resume the retained session after an interruption when its state
  is usable. Supply recovery findings and current record state; do not blindly
  execute remembered commands. Inspect possible external effects as in p3.
- Define an explicit, recorded replacement path for a missing, corrupt or
  incompatible session. Start fresh only once it is established that the old
  execution has ended; preserve work and record the reason and old/new identity.
  If resume failed after possibly executing tools, first treat that as an
  interrupted attempt, not as permission for an immediate fresh retry.
- Do not silently turn an unsupported harness into repeated fresh execution.
  Distinguish deliberate capability refusal from recovery of lost session data.
- Retain foreground-wait guidance: session persistence does not keep child
  processes alive. Future external/delegate waits retain their own identities
  and deadlines independently of a parent conversation.

A persistent-process implementation would require additional changes: explicit
idle/wait states, slot release, input delivery and Observer precedence (currently
an open attempt is classified before human waits). It is not the default scope
of this phase and must not be smuggled into a resume-flag change.

**Completion evidence:** wait A, run B, answer/resume A in its retained session;
then repeat with an interruption and with missing session storage. No duplicate
writer, lost answer, false stall or bypass of a human hold occurs.

## 5. Run a bounded comparison in RTS

Prepare disposable run copies from the same committed RTS baseline, with the
same model/profile, workflow, guides and task constraints. Use a small useful
task with multiple natural question/answer boundaries, a correction and a final
report. Keep outputs on run branches until independently accepted; do not
integrate both experimental implementations into the shared project.

- Record a modest cost/run budget before starting. Begin with one matched pair;
  repeat enough to identify variability, alternating policy order where useful.
  Report small-sample uncertainty instead of claiming a universal saving.
- Use equivalent scripted human inputs for disposable comparison trials, clearly
  identified as trial inputs, and no fabricated acceptance for ordinary work.
  The agent should perform real task work with the actual harness.
- Compare immediate continuations and a realistic delayed answer. Record wait
  duration, cache conditions and task order; session retention does not guarantee
  that provider-side cached prefixes remain available.
- Capture per-attempt input/output, cache-read/cache-write usage where exposed,
  harness-reported cost, prompt size, latency, total run cost, session identity,
  compaction/replacement and completion/repair outcome. Deduplicate cumulative
  usage reports and replayed events. Label unavailable metrics as unavailable.
- Keep reported cost separate from estimates and subscription/quota usage. If
  estimating from token counts, record the current official price source and
  calculation; never treat a flat subscription as measured zero execution cost.
- Judge equivalent output quality, adherence to the agreed plan, correct uptake
  of new answers and freedom from stale assumptions. Savings that cause extra
  repair attempts must include those attempts and recovery costs.

**Completion evidence:** a comparison table with raw evidence references, both
successful continuation chains, cost/quality limits, and an evidence-based
recommendation for the default. A negative cost result is a valid result.

## 6. Verify, deploy and report

Run focused tests at the changed boundaries and the relevant existing suites:

| Area | Required checks |
| --- | --- |
| Policy | validation, Detail save/reload, run snapshot, CLI/API/claim agreement, unsupported profile |
| Identity | explicit resume, independent runs, fresh replacement, actual ID differs/missing, executor restart |
| Context | changed answer/guide, stale attempt references, compaction and fresh reconstruction |
| Lifecycle | answer wait frees slot, manual hold/cancel, interrupted resume, missing session, ambiguous launch failure |
| Observation | same session across attempts, attempt-local activity, no false wait alarm, recovery budget preserved |
| Measurement | cumulative/replayed usage deduplicated, cost labels correct, repair costs included |

Run WFE `npm run check`, affected agautolab and shared-harness tests, and Observer
monitor/recovery tests. Include browser verification of the policy control and
the run's effective settings. Use real harness trials for behavior that a fake
resume flag cannot prove. Do not widen testing solely to preserve retired agents.

Commit and push component changes and required dependency/submodule pointers.
Before activation, inspect current services via Nautobot/nctl and current WFE
runs; protect active writers and retained run/session data. Deploy matching WFE,
executor, harness dependency and Observer versions. Any schema transition must
be explicit and must not resume a held run. Newly started services follow the
LAN-access requirement in localrule.md.

Verify deployed identities, run one disposable multi-attempt session sequence,
and leave the Detail setting and trial evidence ready for human inspection.
Complete agent-operable setup before requesting human checks. Do not change the
person's held experiment as part of verification.

Write `report.md` with capability findings, actual semantics, comparison results,
selected default, source/deployed versions, failure-path evidence and remaining
limits. Update workflow/harness/Observer guides and developer documentation;
keep host paths, credentials, session stores and bulky logs in ignored local
notes. Commit/push final reports and pointer changes as required by localrule.md.

## Definition of done

- Detail offers fresh-per-attempt and retained-per-run policies, both functioning
  for the selected harness and explicitly handling unsupported profiles.
- Retained context survives question waits and process/executor restarts while
  each active interval remains a separate fenced attempt.
- Session loss has a recorded recovery path from authoritative records without
  replaying uncertain work automatically or overriding a human hold.
- Observer distinguishes retained context from active execution and continues
  to detect and verify recovery correctly.
- RTS evidence compares total cost and quality, including cache behavior and
  repairs, and supports the chosen default even if reuse yields no savings.
- The tested behavior is deployed, documented and committed/pushed; future
  delegate/session policies have clear boundaries without being implemented here.
