# project_migration p8 ex1 — Durable delegation waits and clear run status

## Objective

Implement [braindump.md](braindump.md) and the subsequent discussion: normal waits must have an accurate state and a short, useful reason. Delegation lasting hours must not depend on a live harness process. Persist the parent's wait, end its attempt, and start a new attempt when a child event requires attention. Repair missed continuations independently through reconciliation and Observer.

This is an implementation plan, not a deployment report. Creating this plan does not execute its steps.

## Working policy

Follow `devpolicy/styles.md`, `devpolicy/terms.md`, `devdocs/README_DEV.md`, applicable repository instructions, ignored environment notes and `localrule.md`.

- Generated files are English; communication with the Developer is Japanese.
- Backward compatibility is not required. Update affected consumers together rather than retaining misleading state aliases.
- Implementation, deployment and verification do not require a new human approval checkpoint. Preserve unrelated working changes; commit/push intended changes and update submodule pins as appropriate.
- Consult Nautobot or nctl before operational work and distinguish observation timestamps from fresh verification.
- Keep host addresses, credentials, machine-specific paths and raw operational evidence in ignored local files. Use portable references in tracked documents.
- Finish with the deployed experience and direct review links requiring little effort from the Developer.

## Evidence and starting points

The initial read-only investigation found the following:

- `front/converse/run-027`, delegating to `jevlab/study/run-002`, ended with a delegate node waiting but a talk node still running. Two attempt endings produced an interrupted hold. Another Front run resumed it once; a recovery agent subsequently recovered it. It ultimately completed.
- `workflow_editor/shared/run.ts` derives an interrupted hold when an attempt exits with active nodes. Ready work alongside a recorded wait already has explicit deferral support. Do not remove the active-work check merely because some child is running.
- The executor already has child-event continuation and reconciliation paths. Extend and repair these rather than introduce a second scheduler.
- `situation()` currently maps all recorded waits to `awaiting-person`, including delegation and external waits.
- `src/runActivity.ts` receives a hold kind but not its reason; `src/runSide.ts` combines the reason with lengthy generic recovery explanations.

Reconfirm these facts against the current source and deployed release before implementation. Relevant code lives under `pj-agdev/agdevworld/workflow_editor/`, with worker behavior in agautolab, observation/recovery in agobserver and its managed recovery workflow, and Front definitions in the managed Front project.

## Behavioral contract

### Wait ownership and process lifetime

The run record owns progress and waits. A harness process executes one attempt; a retained harness session is optional conversation continuity, not a running process required to preserve the wait.

1. Record the delegation identity and the parent's wait before ending the parent attempt.
2. Leave no active parent work falsely recorded as running. A talk node whose purpose ends at delegation completes there; continuing work explicitly waits, and later result handling is represented in the graph.
3. End the attempt normally and release its worker slot. No harness process or polling loop remains solely to wait for the child.
4. Persist relevant child events and enqueue continuation of the parent. A new attempt may resume a retained harness session where supported, or reconstruct its context from durable records through the existing session policy.
5. The parent takes up the event, judges the result or answers the question, and either continues, completes, or records another wait.

Apply this to short and long delegations initially. A bounded in-process wait is deferred until measured launch overhead justifies it; it must never become the authoritative wait or prevent a child obtaining its execution slot.

### Events and continuation

- Child questions, completion, failure and cancellation are structured events. Natural-language claims of completion do not trigger lifecycle transitions.
- A completed child with an inadequate report still wakes the parent, which judges the report. A child that exits without recording its outcome is an observation/recovery problem.
- Use durable event identities and existing take-up records. Repeated delivery and reconciliation must not enqueue duplicate effective continuations, execute concurrent attempts for one parent, or dispatch duplicate children.
- Event delivery is retryable; do not claim exactly-once external effects. Recovery checks recorded effects before retrying work whose outcome is uncertain.
- An event arriving while the parent runs remains available. If it is not handled in that attempt, continuation is scheduled after the attempt ends, including the race around wait registration and process exit.
- For multiple children, one event can wake the parent while other children remain pending. Waiting for all children must not suppress an intermediate question or failure.
- Intentional stop/cancellation retains its meaning. Events remain recorded, but do not silently override these controls. Interrupted or unknown runs use the recovery path rather than being relabeled as normal waits.

### Reconciliation and Observer

Use two levels of repair:

| Condition | Owner and response |
| --- | --- |
| Child has a durable event missing from the parent's event stream | Existing delegation reconciliation reconstructs/delivers it with stable identity |
| Parent has an actionable unhandled event, no attempt, no queued continuation and no hold | Executor reconciliation enqueues continuation |
| Child process disappeared or its work record conflicts with the observed execution outcome | Observer records an incident and invokes recovery to inspect evidence |
| Parent exits without correctly recording its wait or taking up an event | Show the specific interruption; Observer invokes recovery with parent/child references |
| Delivery repeatedly fails or a parent remains stranded | Expose the transport/queue failure and escalate through existing Observer incident handling |

A long elapsed wait alone is not failure. Distinguish a running child, a queued child, unavailable observations, an explicit deadline and a missing continuation. Preserve evidence freshness and existing incident deduplication. Recovery may continue a run after diagnosing it; it must not repeatedly clear a hold without addressing its cause.

### User-visible status

Derive status from authoritative work, control and execution facts in one shared interpretation used by API and UI consumers. Avoid a second mutable status ledger.

| Situation | Example primary text |
| --- | --- |
| Human answer pending | `Waiting · Your answer` |
| Delegated work pending | `Waiting · jevlab research result` |
| External result pending | `Waiting · <named result>` |
| Continuation queued | `Queued · Waiting for an execution slot` |
| Intentional pause | `Paused · <recorded reason>` |
| Interrupted attempt | `Interrupted · Talk node left running at exit` |
| Lost process/outcome | `Unknown · Process lost; outcome unconfirmed` |

Use `Watching` only if an actual registered observation/notification mechanism supports the claim. This episode does not add a general monitoring engine.

Carry a concise reason, target reference and continuation condition where known. Prefer structured cause data for new interruptions; retain the full diagnostic text in details. Multiple waits should show a useful summary with the individual targets accessible. A child failure or pause must be visible rather than hidden behind a generic healthy wait.

Show the status and reason prominently in run details, run lists and Front Desk. Move generic explanations about preserved work and recovery into details. Keep actual holds visible even when a question is also pending; never imply that answering alone will resume a held run.

## Implementation sequence

### Step 1 — trace and reproduce

Inspect source, deployed versions, current working changes and service state. Trace delegation dispatch, event persistence/relay, attempt ending, continuation queuing, reconciliation, Observer incidents and UI derivation. Reproduce the run-027 failure in a controlled fixture and identify which Front graph/guide behavior left the talk node active. Inventory consumers of `situation()` before changing its vocabulary.

Deliverable: `report1.md` with the reproduction, current ownership and concrete changes needed. Keep live evidence in ignored storage.

### Step 2 — make normal waits durable and resumable

Repair Front's workflow/guide and the shared wait operation where necessary so delegation leaves an accurate parent graph. Reuse existing CLI operations; add a cohesive operation only if current operations cannot reliably express the wait. Document semantics in tool help and the common run contract, without duplicating procedural rules across agent guides.

Repair event persistence, continuation scheduling and reconciliation gaps found in Step 1. Verify that process exit releases the slot and retained session support is independent of waiting. Preserve detection of genuinely unfinished active work.

Deliverable: implementation and focused lifecycle tests, documented in `report2.md`.

### Step 3 — close recovery gaps

Make deterministic reconciliation repair missing delivery/queue records from durable evidence. Extend Observer's existing observations and recovery context for ambiguous execution, malformed work records and repeated continuation failures. Include the parent, child, event and attempt identities and the precise reason. Keep recovery incident handling idempotent and avoid retry loops that dispatch or repeat child work.

Deliverable: recovery behavior and failure-injection evidence in `report3.md`.

### Step 4 — unify status and concise reasons

Separate human, delegation and external waits in shared status interpretation. Expose structured interruption reasons and update API consumers, run activity, run focus, project lists, delegation views and Front Desk. Audit notification and Observer consumers affected by changed state names. Show queued continuation distinctly from waiting for a child.

Deliverable: UI/API changes, representative screenshots and status checks in `report4.md`.

### Step 5 — verify, deploy and demonstrate

Run the focused tests below and the repository's required checks. Inspect Nautobot/nctl state before deployment. Back up affected stores, record deployed revisions and define a rollback covering code and any changed records. Deploy the required coordinator, worker, Observer and managed-project changes in a compatible order; restart only affected services.

Inspect outstanding waits/holds through supported operations. Repair only those whose evidence establishes the cause; do not bulk relabel holds or rewrite history. Perform a controlled deployed Front delegation, confirm that its parent process and slot are released, and verify automatic return of its result to the originating conversation. Check the same status through the LAN-accessible UI. Use an explicitly identified trial actor for any authorized test messages.

Deliverable: `report5.md` with test results, deployed revisions, remaining limits, recovery/rollback notes and direct review links in ignored local environment notes. Update developer documentation and relevant project pins. No additional approval checkpoint is part of this sequence.

## Verification and completion criteria

- Reproduce the original active-talk-node failure and prove the corrected Front records a normal wait without a hold.
- Prove genuine active work abandoned at exit still produces an interruption with a concise reason.
- While a child remains active, verify no parent harness process occupies a worker slot and another request can execute.
- Simulate an hours-long wait through durable state, elapsed timestamps and coordinator/worker restarts; include a controlled deployed wait across a restart. Do not require a harness to sleep for hours to prove persistence.
- Verify child question, completion, failure and cancellation each reaches the parent; inadequate report content does not prevent wake-up.
- Exercise duplicate events, multiple children, event arrival before/during/after parent exit, and crashes between event persistence, queue insertion and event take-up.
- Inject missing delivery and missing queue records; deterministic reconciliation repairs each without duplicate child work.
- Exercise lost child process, interrupted parent and unavailable observation; Observer explains and recovers the relevant incident without treating a healthy long wait as failure.
- Verify stop/cancellation prevents automatic continuation while preserving incoming evidence.
- Verify each representative status has a short reason consistently in the run view, list and Front Desk, with detailed evidence still accessible.
- Complete deployment and the controlled end-to-end trial. Record what was tested versus inferred; leave no claim that a deployed behavior was verified solely by a fixture.

## Out of scope

- A separate Front execution engine or scheduler.
- A new general monitoring service or mandatory periodic harness polling.
- Automatic inference of lifecycle transitions from natural-language reports.
- Short-delegation in-process wait optimization before measured need.
- Broad workflow redesign unrelated to correct waiting, continuation and explanation.
