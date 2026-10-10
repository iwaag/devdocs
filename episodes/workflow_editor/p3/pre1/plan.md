# Workflow editor p3/pre1 — prepare observable workflow execution

## Goal and scope

Prepare the existing editor for a person to give an Omni Agent a braindump in
VS Code, have it execute a workflow, and follow the work in the browser.
Implementation stays in `pj-agdev/experiments/workflow_editor/`.

The input is `../braindump.md` and the implementation proposal discussed with
the person. The discussion explicitly accepted a version-controlled workflow
snapshot together with execution state: each Git commit preserves which workflow
the run followed and where execution stood, while timestamped history preserves
transitions between commits.

Pre1 supplies the record contract, tools, UI, IDE guidance, and a readiness
rehearsal. P3 is the subsequent human-led trial using the person's actual
braindump. A synthetic rehearsal does not complete that trial.

The Omni Agent performs the work under human supervision in VS Code. Repository
access and command restrictions are self-checked declarations in this phase.
Automatic agent launching, crash recovery, and unattended execution are not
required. Writers continue to take turns, as in p2.

## Starting point

The editor already has Git-backed projects, `devdocs/workflows/*.yaml`, stable
workflow and node IDs, DAG validation, repository bindings, definition approvals,
and `study`, `do`, `talk`, and `delegate` nodes. An edge makes its target wait
for its source; a join waits for all predecessors; a delegate waits for another
workflow. These semantics currently describe definitions only.

The CLI and browser share operations. The service observes files and sends SSE
notifications, with definition polling around one second and separate Git-state
observation. Browser reconnection re-reads current state. Extend these mechanisms
for runs without replacing them with a scheduler or a database.

## Run files and their authority

Use this layout inside each project:

```text
devdocs/<workflow-id>/runs/run-001/
  braindump.md
  plan.md
  run.json
  definition/
    <workflow-id>.yaml
    <delegated-workflow-id>.yaml
  report*.md
```

An agent-originated request uses `request.md` instead of `braindump.md`.
Run directory names may be sequential or descriptive (`run-<name>`); creating an
existing run must not overwrite it. Define validation and collision behavior in
the contract and command help.

- `braindump.md` preserves the person's own input. Saving their words through an
  agent does not turn them into an agent-authored request.
- `request.md` contains a request an agent constructs on the person's behalf.
  Record the requester and a reference to the entrusting input or parent run;
  do not fabricate human authorship. Keep the original input available when the
  agent produces a derived request.
- `plan.md` describes the work. The agent chooses useful report names and contents;
  the machine does not infer execution state by parsing their prose.
- `definition/` is the fixed definition bundle for this run.
- `run.json` is the authority for recorded execution state, questions, relations,
  artifact references, and timestamped changes. UI and CLI read the same record.

All of these are version-controlled devdocs content, including `run.json` and
`definition/`. A save updates the browser even before a commit. Saving does not
implicitly commit or push; ordinary Git operations publish meaningful milestones.
Document committing devdocs and, when publishing the project, its updated gitlink.
Keep machine-specific absolute paths, credentials, and local process details out
of these files; local execution settings belong under ignored `.local/` paths.

### Definition snapshot

At run creation, validate and copy the saved workflow and its transitive delegate
targets into `definition/`, indexed by their stable IDs. Preserve the actual
definition content, including the layout and approval records. Record IDs and
definition digests; a digest alone cannot reconstruct the executed workflow and
the existing approval digest does not include the workflow ID.

The bundle is fixed for the lifetime of the run. Later edits, renames, or deletion
of the source workflow do not change the run graph. The UI distinguishes the
snapshot from the current editable definition. A changed execution definition
belongs to a new run, optionally referencing its predecessor.

Delegated runs use the relevant definitions from the parent's fixed bundle, not
whatever source YAML exists when the delegate node is reached. Keep their bundles
self-contained so historical child runs remain readable. Record parent run/node
and child run references explicitly, including the workflow IDs needed to locate
them. Different invocations of one workflow have distinct run IDs.

Definition approval remains a statement about a definition, not permission to
execute or acceptance of a result. Preserve its status without inventing another
approval gate. Snapshotting workflow definitions does not snapshot repository
contents or promise reproducible execution; record available repository revisions
and dirty-state facts as context, without claiming a commit includes dirty files.

### Record shape and updates

Define a versioned run schema, proposed as `ag.workflow-run.v1`, separate from the
existing record of a single agent/harness execution. A workflow run may span many
agent turns and long human waits.

Include run identity, input kind and reference, requester, executor identity,
workflow snapshot references, creation/start/end times, node records, parent/child
relations, questions/answers, and project-relative artifact references. Identify
timestamp format and stable IDs in the contract. Keep backend/model information
separate from agent identity when it is known; unavailable facts remain unknown.

Each state-changing operation updates current state and appends a timestamped,
ordered history entry in the same atomic replacement of `run.json`. The history
records the actor, affected run/node/question, change, and relevant reason or
evidence reference. Define one reducer/validator shared by CLI and service so
current state and history cannot silently disagree. This is an operational record,
not authenticated or tamper-proof evidence.

Validate reads and writes. Missing, malformed, or inconsistent records must be
visible errors, never an empty or successful run. Preserve the last valid browser
view with its error where possible; do not overwrite invalid files automatically.
Keep sequential-writer semantics explicit rather than claiming concurrent safety.

## Execution and waiting semantics

Track nodes individually; a run has no single `current_node` because several
branches may be active or waiting.

| Node state | Meaning |
| --- | --- |
| `pending` | Work has not started |
| `running` | The executor recorded that work started |
| `waiting` | Work awaits an identified answer, child run, or external result |
| `completed` | The executor recorded completion with an outcome reference |
| `failed` | A recorded problem prevents continuation |
| `cancelled` | Work was explicitly discontinued |

Derive readiness from successful completion of every predecessor. Failure or
cancellation does not satisfy a dependency. Enforce these graph facts through
shared operations, while leaving how a ready node is performed to the agent.
Independent branches may be performed sequentially in this phase; no automatic
parallel worker pool is needed.

For waiting nodes, record the reason, who or what holds the next move, and the
question, child run, or external-result reference. Questions have stable IDs;
answers identify which question they address. An answer being recorded is distinct
from the agent taking it up, completing the node, or accepting the result. Allow
explicit withdrawal of a question without manufacturing an answer.

A delegate node links to a child run and waits for it. Parent completion requires
the child's recorded successful completion; missing, failed, or cancelled children
remain distinguishable. The agent uses the same tools to record the parent taking
up a child's result; no background workflow runner is implied.

Derive the run summary from its nodes and any explicit cancellation, retaining
mixed facts: one waiting branch does not hide another running branch, and an
independent failure remains visible. Specify the summary rules in the contract.
All nodes must complete for execution success. Result acceptance, where requested,
is a separate decision with actor and evidence; neither an agent's completion
record nor an answer to an unrelated question invents that acceptance.

Long waits and long work have no automatic expiry. Display last-recorded update
times and explicitly describe `running` as reported progress, not proof of a live
process. No-update duration and an SSE connection do not establish executor health.
Do not add heartbeat quotas, timeout-based failures, or automatic resumption.

## Tools and browser experience

Add shared run-domain operations and expose them through `wfe` and appropriate
browser actions. Proposed CLI capabilities, with final syntax to be documented:

- Create, list, and inspect runs, including their fixed definitions and ready nodes.
- Start nodes; record waits, completion, failure, and explicit cancellation.
- Record questions, answers, their uptake or withdrawal, and outcome references.
- Create a delegated run with its parent relation and inspect both directions.
- Attach reports/artifacts and inspect execution outcome and any recorded acceptance.

Keep the CLI independent of the running browser service, as it is today. Help
states each operation's inputs, reads/writes, and results; offer structured output
for agents and readable output for people. File formats remain documented, and
file editing remains available without requiring handwritten JSON for routine use.

Add a run list and a run view using the existing graph renderer. Show the fixed
graph, node status with text as well as color, ready nodes, all active branches,
waiting reasons, questions, answers, artifacts, parent/child links, and update
times. Keep current-definition editing distinct from execution-state updates.

People and agents have documented routes to the same operations through browser,
CLI, or VS Code/Git. Provide a browser question/answer record interface, but do not
imply that submitting an answer launches or wakes the VS Code agent. VS Code is
the conversational entrance; the agent can also record the relevant exchange from
there. The browser stores the answer and the person continues the IDE session.
This phase does not need a general-purpose embedded chat system.

Extend file observation and SSE with targeted run refreshes. Start with the
existing one-second interval. Observe relevant run metadata without rereading all
historical report bodies on each tick. Progress changes must not discard unsaved
definition edits or rebuild unrelated views. Reconnection must recover changes
made while disconnected; expose disconnection and read errors visibly.

## IDE handoff and access declarations

Extend the existing setup templates and help with a short execution entry point.
Identify the working directory, selected project/workflow, input file, run folder,
CLI invocation, and browser link. Link to the canonical contract rather than
copying its contents into the agent guide. Preserve existing authoring workspaces
when refreshing setup.

The guide explains the available tools, the fixed execution definition, reporting
locations, and the meaning of recording starts, waits, and outcomes. Give the
agent discretion over its plan and reports without prescribing every tool call.

Repository bindings remain declarations checked by the Omni Agent. The current
model has no allowed-command schema; document concrete restrictions in the run's
plan or execution context when supplied, rather than inventing a generic policy
engine. Explain that writing the run record to devdocs is a reporting capability
separate from permission to edit repositories used by a work node. This convention
does not grant general write access to a readonly binding.

## Future integration boundary

No Zulip or Observer dependency is needed for this experiment. Preserve stable
run/node/question IDs, parent-child relations, explicit next-move holders, and
evidence references so future adapters can connect them to conversations,
messages, and individual agent execution records.

In the future system, devdocs retains durable input, plans, definitions, and
results; Zulip supplies conversation and delivery evidence; Observer can combine
that evidence with actual execution health. Specify authority and synchronization
when those adapters are implemented rather than introducing duplicate state
authorities now. Workflow progress and process health stay separate concepts.

## Implementation sequence

1. **Contract and baseline.** Inspect existing run-related capabilities, finalize
   the schema, state/summary rules, snapshot closure, child-run identity, and
   operation matrix. Capture the current editor checks and update behavior on
   isolated data. Document the contract in the experiment.
2. **Run records and CLI.** Implement shared validation, snapshot creation,
   atomic state/history updates, graph readiness, questions, artifacts, and
   parent/child operations. Keep CLI and HTTP wiring thin.
3. **Run UI and observation.** Add run navigation, snapshot-based rendering,
   node details, question/answer recording, and targeted refresh/reconnection.
   Confirm that source-definition edits cannot rewrite historical run views.
4. **IDE handoff.** Update templates, command help, and start instructions.
   Verify both parties' operation routes and self-check access documentation.
5. **Readiness rehearsal and report.** Exercise the scenario below with synthetic
   input on separate data. Write `report.md` here with actual commands, results,
   update measurements, limitations, and concrete instructions for the human's
   p3 trial. Do not execute that trial on the person's behalf.

## Validation and completion criteria

Run focused tests of the new shared contract and operations, then the experiment's
type checks, test suite, and production build. Browser checks use an isolated
registry/repository tree, never reset the person's authoring data. If inspecting
existing local services, establish their state through Nautobot or nctl as directed
by the project environment instructions.

The rehearsal must establish:

1. Human input and delegated requests preserve their distinct authorship and
   provenance. Run-name collisions do not overwrite previous records.
2. A run captures its definition and transitive delegates. Editing, renaming, or
   deleting source definitions leaves that run and later-created child runs on
   their captured versions.
3. A graph with branching and joining shows all active/waiting branches and does
   not permit a join before every predecessor completes. Failure/cancellation do
   not appear as success or unblock dependent work.
4. A talk node retains its explicit question while waiting; an answer to another
   question leaves it waiting. Recording the intended answer, taking it up, and
   completing work remain distinguishable. Simulated old timestamps alone never
   trigger failure or completion; a multi-hour automated wait is unnecessary.
5. Delegated runs are navigable in both directions, use the fixed definition,
   and do not produce false parent completion on an absent or failed child.
6. State transitions and their history survive reload. Two meaningful Git
   commits can each reconstruct the fixed workflow and execution stage at that
   commit; history retains transitions made between those commits.
7. File saves, atomic replacements, burst updates, connection interruption,
   malformed-record recovery, and run switching produce the correct UI state
   without losing an unsaved definition draft or applying an obsolete response.
8. Ordinary progress saves normally appear within about two seconds under the
   representative local workload. Record repeat counts, latency distribution,
   workload size, idle/file-read/Git activity, and the observation interval used.
   Treat this as a measured usability target, not a universal timing guarantee.
9. The person can start from the documented directory, inspect progress, answer
   a question, find reports, and continue in VS Code without reading implementation
   code. The operation matrix has no unexplained capability asymmetry.

Pre1 is complete when these checks pass and the UI makes it clear what finished,
what is active, what waits for whom, and where results are recorded. The report
states which evidence is synthetic and leaves the actual human-led p3 run pending.

## Out of scope

Automatic scheduling or agent launch, unattended recovery, process-health probes,
Zulip/Observer/agdevworld integration, OS-level access enforcement, a general
allowed-command engine, concurrent writers, conditional branches, loops, automatic
retries, and comprehensive Git management in the browser.
