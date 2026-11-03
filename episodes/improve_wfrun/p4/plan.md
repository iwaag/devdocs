# improve_wfrun p4: agent-directed execution coordination by warden

## Purpose and agreed direction

Let independent runs execute concurrently on the same PC when an agent judges that doing so is
useful. Extend warden from recovery of interrupted work to helping the system operate reliably and
keep work moving. Execution coordination is the first addition to that broader responsibility;
existing recovery remains part of it.

The Developer explicitly prefers flexibility to perfect admission decisions. Start with agent
judgment, tolerate mistaken judgments, and improve from observed outcomes. Avoid replacing the
current one-attempt limit with resource-name locks, compatibility matrices, fixed load thresholds or
workflow-category rules. CPU, memory, GPU, display and shared-path facts are evidence for judgment,
not automatic admission policies. Unnecessary waiting is an undesirable outcome too.

Reference: [opinion.md](opinion.md). Its account of the blocked recovery motivates this phase; its
proposal for deterministic resource admission is superseded by the subsequent discussion and the
Developer's agreement. The historical incident has not been reverified for this plan.

## Roles and scope

- **warden** judges how queued work should proceed: start on the current node, place on another
  available node, wait, investigate, or arrange a practical change that makes concurrent work viable.
  It may revise its decision as work changes or interference appears.
- **autolab** executes the chosen attempts and reports their lifecycle. It remains the execution
  mechanism, rather than becoming another independent scheduling agent.
- **WFE** owns run and attempt state, records decisions and applies their effects consistently.
  It checks identity and lifecycle correctness, not semantic resource compatibility.
- **ponderer and the working agents** provide intent, urgency and facts about the work when useful.
  They need no mandatory resource declaration or consultation at every graph node.
- **cagent** supplies additional environment knowledge or changes when warden needs them. Selecting
  another already usable execution node is within warden's coordination role and does not require
  a cagent run on every placement decision.
- **Observer** continues observation and incident detection. Coordination waits and failed decisions
  become observable without making Observer a second admission-policy engine.

Deliver same-node concurrency on both coordinator-local and remote workers, agent-directed placement
between existing workers, useful decision visibility, and an operational warden coordination path.
Live migration of running processes, a general resource reservation system, mandatory resource
declarations, comprehensive process/filesystem auditing and automatic infrastructure expansion are
outside this phase. Environment changes may use existing cagent workflows when actually needed.
The unrelated no-node-record and progress-reporting issues in opinion.md remain separate, except
where they prevent demonstrating this phase's behavior.

## Starting point

The current code and contracts have more than one single-execution assumption:

- WFE claims queued jobs oldest-first within the assigned worker/lane and refuses claims at capacity
  (`agdevworld/workflow_editor/server/exec.ts`, `docs/workers.md`).
- autolab has one Executor loop per lane, each owning one attempt
  (`agautolab/src/agautolab/wfexec.py`). Raising a configured capacity alone does not add execution.
- Remote worker bookkeeping in `state.json` assumes a single executor owner; extra lanes are refused.
- Assignment changes occur between attempts. A session on another worker is replaced through the
  existing recorded session mechanism, not transferred as a live process.
- warden currently offers recovery, and its guide explicitly assumes the recovered run waits until
  the recovery attempt releases the execution slot.

Confirm deployed revisions and these assumptions before implementation. Keep host identities,
addresses, configuration paths and operational evidence in ignored local records. Use portable
worker identities in contracts and remappable test configuration.

## Step 1: establish warden's coordination contract and execution path

Update warden's maintained entry and role guides to describe its expanded purpose, current recovery
and execution-coordination capabilities, readers, available tools and limits. Keep shared facts in
one place and tool invocation details in tool help. The role should encourage trying plausible
concurrency, observing the result and adapting, rather than requiring certainty before starting.

Give coordination a service path that remains runnable while ordinary work occupies the workers.
Prefer a dedicated warden coordination workflow served independently of ordinary work, retaining
normal run records. Its own start must not require a coordination decision about itself. Existing
conversation traffic must not be the sole route to this service. Do not make an extended recovery
or cagent task hold the coordination path while it waits for another run.

Specify and implement event delivery for queued work, attempt completion, changed worker availability
and explicitly requested reconsideration. A waiting decision records a reason and what should prompt
another look, including a wake time when appropriate. Events and wakeups survive restart and are
coalesced so a burst does not create competing coordinators or a storm of model calls. The agent
decides the work order and placement; event delivery does not encode a compatibility policy.

Use one authoritative coordination decision stream initially so concurrent requests are considered
together. The agent can dispatch several runs in one turn without waiting for them to complete.
Its own interruption, model failure or unavailable tools must leave a visible pending decision and
a retry/recovery path outside the blocked ordinary queue. Do not silently fall back to a hidden
resource policy or silently drop queued work. Preserve the existing owner-visible reporting of an
executor outage; this phase does not promise recovery when no execution service is alive.

## Step 2: give warden evidence and actionable choices

Provide a concise read view of queued and executing work across workers: stable identities,
requests, current nodes/progress, recent decisions, attempt/process state, worker capabilities,
observed load and observation times. Reuse WFE observation and worker contracts. Missing or stale
facts are labelled as such. Offer deeper logs and worker inspection on demand instead of stuffing
every session log into every coordination prompt.

Expose supported operations for recording and applying a decision to start selected queued work,
assign it to another usable worker between attempts, defer it with reconsideration, and request
additional information. Deliver relevant coordination notes to the affected working agent: for
example, why another run is sharing the node and an agreed separate temporary path. Reuse existing
communication/delegation and lifecycle operations for consultation and recovery where suitable.

Distinguish a default node preference from an explicit user placement constraint. Preserve the
person's stated constraints and holds. If existing records cannot express this distinction, add
the necessary provenance instead of treating all assignments as either immovable or freely movable.
For relocation, account for worker-held files, pending publication and session replacement before
the next attempt starts; a missing local dependency is a fact to resolve, not evidence of readiness.

Keep decision output small: affected runs, action/worker, reason, relevant observations and unresolved
assumptions, plus reconsideration information for a wait. Link it to the resulting attempts. The
record supports investigation of mistakes; no prescribed resource taxonomy, confidence score or
approval ceremony is required.

## Step 3: implement concurrent execution and consistent decision application

Replace the ordinary per-worker/per-lane single-attempt restriction with execution of warden's
selected jobs. Remove lane capacity as the semantic admission rule for ordinary work; account for
existing conversation-lane definitions so they do not retain an accidental fixed concurrency ceiling.
Keep the independent coordination path an execution mechanism, not a resource category system.

Retain one live attempt per run and idempotent request/launch receipts. Apply decisions against
identified queued jobs and the relevant observed dispatch state. If another dispatch or a lifecycle
change invalidates that basis, return the changed facts for reconsideration rather than inventing a
resource judgment in code. A metric sample changing alone need not invalidate every decision.
Record partial application and retries so a failed batch does not launch the successful members twice.

Generalize autolab to track several attempts independently: process ownership, stop requests, logs,
exit reports, sessions, owed publication and restart adoption. Heartbeat, intake, notifications and
reconciliation must continue while attempts launch or do expensive work. Stopping or losing one
attempt must not stop or report another as ended.

Generalize remote state, forwarded commands, log offsets, synchronization and checkpoint recovery
for multiple attempts. Serialize shared bookkeeping where necessary without serializing the work
itself. Inspect shared caches, copy preparation and publication paths for concrete races. Preserve
attempt identity and worker ownership across delayed messages, reconnects and executor restarts.
Unknown process state must not cause duplicate execution or reassignment of a still-running attempt.

These mechanisms maintain the meaning of an agent's decision. They do not decide that two Godot
runs, two GPU users or a measured load level are incompatible. Physical execution failures are
reported to warden with their actual cause and outcome.

## Step 4: make coordination visible and support revision

Show all active attempts on a worker, replacing default-slot-only summaries where misleading.
For queued work, show whether it awaits a decision or was deliberately deferred, with the latest
reason and next reconsideration. Show placement changes and the actual resulting worker. Keep
implementation identifiers and detailed observations in Detail rather than the ordinary user flow.

Allow working agents and recovery to report changed needs or observed interference and request
reconsideration. warden may arrange different paths/ports, wait, or use supported stop/resume and
between-attempt relocation when appropriate. Stopping is not assumed to be reversible or equivalent
to safely suspending arbitrary external work; retain the actual outcome and use existing recovery
evidence. Human-requested holds remain human intent.

Update Observer's evidence and classifications where the old slot model would misreport concurrency
or hide an abandoned scheduling decision. A documented wait is not proof that waiting remains useful;
an expired/missed reconsideration must remain visible. Keep policy judgment in warden.

## Step 5: verify mechanisms and evaluate real agent decisions

Use deterministic tests for execution correctness, and real-model trials for coordination judgment.
Do not claim that stubbed decision output proves the agent can make useful choices.

| Case | Evidence sought |
| --- | --- |
| Rendering plus research/recovery | Real overlap on one node; the short task progresses before the rendering attempt ends. |
| Godot plus Godot | Separate lightweight Godot tasks are considered on their actual work and demonstrated concurrently, without category-based rejection. |
| Shared-state conflict | A disposable fixture exposes a shared path, port or process-selection hazard; record whether warden investigates, separates work or waits, and the actual outcome. |
| Changing work | An initially light task becomes heavier or reports interference; reconsideration occurs and the chosen response is executed and recorded. |
| Alternative worker | warden selects another available worker; dependencies, files and session replacement are handled and the run completes there. |
| Busy system | Coordination remains responsive while ordinary attempts run; recovery does not wait solely because those attempts hold the former slots. |
| Incorrect decision | A controlled wrong decision produces traceable interference; warden can revise it and work can continue without Omni making the scheduling decision. |
| Deferral | The recorded event/time triggers reconsideration; unnecessary waiting is examined alongside collisions. |
| Multiple attempts and restart | Local and remote workers preserve per-attempt logs, stops, outcomes, owed publication and ownership through restart/reconnect. |
| Racing requests | Stale decisions, repeated receipts and partial application produce neither duplicate attempts nor lost queued work. |
| Judgment unavailable | Failed or interrupted coordination leaves visible pending work and recovers without depending on an ordinary slot becoming free. |

Use disposable projects for deliberate collisions. Include meaningful regression tests for two
attempts on the same worker, simultaneous remote forwarding/publication, stopping only one attempt,
and concurrent enqueue/decision application. Run required WFE and affected executor/Observer checks,
plus replay checks for changed persistent records. Verify conversation responsiveness and normal
intake/delegation behavior, not just the new coordination endpoint.

Measure queue delay, decision latency/model invocations, actual overlap, task completion, failures
and unnecessary waits. Compare with a comparable serial fixture and state limits of the comparison.
Inspect the evidence behind each judgment rather than expecting a specific wording. If a trial
reveals a failure, identify missing facts, tools or mistaken reasoning before adding guidance; do
not automatically convert that failure into a permanent resource prohibition.

## Step 6: deploy and demonstrate the ordinary user path

Deploy WFE, autolab, warden definitions/guides and any affected Observer changes through the existing
coordinator and managed-worker mechanisms. Verify installed revisions and service health on both
worker types. Handle active attempts and state conversion explicitly during deployment; backward
compatibility with older releases is not a requirement, but current work and records must be
accounted for. Do not make rollback infrastructure a separate project.

Exercise ordinary requests through the deployed interface with a named check actor. Demonstrate
same-node overlap, a responsive warden, an agent-selected alternative worker and an observable
deferral/reconsideration. Confirm completion and publication of the outputs, not merely simultaneous
process presence. Leave a readily inspectable view and linked records so the Developer can verify
the behavior with little setup. Attribute trial instructions and judgments to their actual actors.

## Deliverables and completion evidence

- warden's maintained role describes system stability support, with recovery and execution
  coordination implemented and their actual limits stated.
- Deployed agent-directed concurrency and placement work on local and remote workers, with an
  independent coordination path and durable decision/attempt correspondence.
- WFE views, CLI/API help, worker/run contracts, autolab documentation and development entry points
  reflect the resulting behavior. Host-specific setup and evidence stay in ignored local records.
- `report.md` in this folder names exact revisions, changes, checks actually run, live trial records,
  observed successes and mistakes, measured delays/overlap, and remaining limitations. Separate
  implemented mechanisms from what real-agent trials have demonstrated.

Completion means the agent can make and revise useful execution decisions through the deployed
system, and the machinery faithfully carries them out. It does not mean perfect conflict prediction.
Further warden responsibilities and any stronger scheduling policy should follow evidence from use.
