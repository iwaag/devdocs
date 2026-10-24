# Workflow editor fix5 — reminders at operation boundaries and retrospective nodes

## Goal and evidence

Fulfil [braindump.md](braindump.md): help an agent notice omitted
presentation, cleanup and node records while it can still correct them,
and let it record work already performed without pretending to execute it
again.

fix4 improved the catalog, freework skeleton and wfrun guide. Its trial,
`gimmickimg/freework/run-009`, performed presentation and cleanup. The
later `run-010` recorded several experiment nodes but completed without a
presentation. A separate, explicitly requested `run-011` then presented
run-010's result, received feedback and stopped the page. These records
show that the capability exists, but the agent still omits it unless
reminded. Earlier runs 005–008 also demonstrate work hidden inside talk
and presentations left serving; see [fix4's report](../fix4/report.md).

## Agreed behavior

- Do not add an optional `wfe run check-in`. Put reminders on the existing
  operations an agent uses to ask, report, complete work and handle feedback.
- Teach ordinary commands without `--yes` in the guide and normal examples.
  When a warning applies, return it **before** recording the operation,
  publishing its message or triggering its downstream effects.
- The warning explains the observed facts, the uncertainty, concrete repair
  commands and how to repeat the original command with `--yes` if the agent
  judges continuation appropriate. No human approval is required.
- After repair, the ordinary command works when its warning conditions no
  longer apply. A generic self-check that cannot be resolved from records
  may still require `--yes` after the agent has considered it.
- `--yes` bypasses these advisory checks only. Invalid arguments, stale
  revisions, attempt ownership, invalid graphs and other correctness errors
  remain errors. It grants no persistent exemption to later operations.
- Accept the known limitation that an agent may learn to use `--yes`
  immediately on later calls. Do not add mechanisms to conceal the flag,
  enforce reflection or require approval to compensate for that limitation.
- A reminder is not proof of an omission. WFE cannot observe every shell
  action or determine whether every result needs a presentation.

## 1. Define shared warning evaluation and operation coverage

Implement one evaluator over the current run, effective graph, catalog
snapshot, presentation records and proposed operation. Return structured
warnings with stable codes, supporting references, explanation and repair
advice. Use the same evaluator for CLI, HTTP and remote-worker operations;
do not implement this solely as CLI output formatting.

Map the actual ask, conversation result/report, completion and combined
adjustment/completion entry points before editing. In particular, cover
`adjust --completing` and the transition that actually completes authored
work, rather than relying on `_final-report`: fix4 recorded runs already
shown as completed while that document node remained pending.

| Boundary | Reminder |
| --- | --- |
| Completion of substantive work | If no relevant presentation is recorded or planned, ask whether the implementation, change or experiment should be presented through its own node. |
| Presentation registration | When the presentation is being registered inside unrelated work, or has no cleanup planned, explain the missing separation or follow-up. |
| Resumption after a presentation answer, and answer take-up | Identify the presentation and feedback; remind the agent to decide whether more work is needed or review is finished and cleanup is due. |
| Ask, outward result/report, and run-completing operations | Ask whether catalog activities were performed without their own nodes, and identify any known presentation or cleanup omissions. |

Use template provenance, node meaning available in structured records and
actual presentation links. Do not assume that a node must literally be
named `present` or `cleanup`. Account for planned but unfinished nodes and
multiple presentation revisions. Keep the initial rules narrow enough that
ordinary consultation and clarification questions do not receive irrelevant
presentation warnings. At outward reporting boundaries, give a concise
self-check for unrecorded catalog activities where work may have occurred;
do not claim that their absence can be mechanically established.

Feedback arrival alone is not permission to stop a presentation: the answer
may request changes or continued availability. Include the reminder in the
resumed attempt's context, and apply the pre-operation warning to relevant
agent actions. Never reject or delay recording a person's incoming answer.
Do not automatically stop pages or add work merely because a warning exists.

## 2. Make warning responses recoverable and consistent

Reuse the existing completion-warning convention where appropriate, moving
the new checks into the common operation boundary. Evaluate against the
same current revision used by the mutation, under the existing concurrency
control. A warned operation records no question, completion, revision or
publication and queues no work. Check combined operations before any part
of them is committed.

Provide equivalent confirmation input for HTTP callers and forward it from
remote workers. User-facing clients that perform these operations must show
the warning and support explicit continuation; they must not silently set
confirmation. Preserve existing response data and explain that nothing was
recorded. Retrying must not duplicate questions, posts or operation events.

Consolidate simultaneous warnings into one response. Fixing a missing node
or stopping a presentation removes that warning. Advisory reminders in
state/continuation output should not repeat continuously without relevant
new state; explicit risky operations are still evaluated each time.

## 3. Add an atomic retrospective recording operation

Provide a dedicated operation, provisionally `wfe run record-done`, for
adding work that actually happened as a completed node. Its help owns the
full usage; the wfrun guide points to that help. Support a catalog template,
node identity and graph placement, outcome, reason for retrospective
recording, and available artifacts or existing record references. Final
argument names follow the existing CLI conventions.

The operation must:

- Create an immutable definition revision and the retrospective completion
  together, with normal graph validation, revision/attempt checks and retry
  deduplication. No partially added pending node is exposed to scheduling.
- Record the author, recording time, reason, outcome and evidence. Optional
  actual work times are separately identified as reported times; unknown
  times remain unknown. Do not synthesize ordinary start/complete events
  suggesting the work just ran.
- Preserve the original talk outcome and historical events. Show the added
  node as completed with a visible retrospective marker in the graph,
  details, history and replay. It appears from the recording revision,
  not retroactively in older graphs.
- Perform no work, start no process, dispatch no delegation and invent no
  answer or approval. Handle completed runs without accidentally scheduling
  a new attempt or generating a second completion notification for old work.
- Allow an existing presentation to be linked to its retrospectively added
  node through an additional provenance record, retaining its original
  registration event. Validate that the referenced evidence exists.

If the activity never happened, add ordinary pending work with `adjust`
instead. A missing presentation cannot be repaired by declaring it done.
Starting a genuinely new task after retrospective recording remains an
ordinary workflow revision, with ordinary execution semantics.

## 4. Update guidance and documentation

Update the wfrun guide, command help and WFE documentation for runs,
adjustments and presentations. Ordinary examples omit `--yes`; the warning
response explains it at the point of need. Explain the three available
responses: perform omitted work, retrospectively record work already done,
or consciously continue despite the warning.

Keep tool usage in tool help and shared guidance in its existing shared
location. Cite run-010/run-011 and fix4 alongside the guidance they justify.
Do not add another blanket instruction to every project or change frozen
catalog snapshots implicitly.

Primary implementation areas are `agdevworld/workflow_editor` shared run
models, server operations, executor continuation, CLI/API and run views,
plus `agautolab/agent/guides/wfrun/guide.md`. Extend catalog metadata only
if needed for reliable identification; avoid a general rules engine for
this initial set of reminders.

## 5. Verification

Add focused behavioral checks and run the affected projects' required
checks:

- Missing presentation at a work/report boundary produces a warning before
  any mutation or outward post. Repair permits retry; `--yes` permits
  continuation and does not suppress correctness errors.
- CLI, direct API, remote worker and combined adjustment/completion paths
  have the same behavior. Concurrent state changes and repeated requests
  do not duplicate effects or apply a stale revision.
- A presentation answer reaches the agent normally. A request for changes
  does not stop the page; completed review prompts cleanup, and verified
  stopping removes that warning. Planned cleanup and multiple revisions
  are handled without misleading claims.
- A retrospective node is added and completed atomically, marked honestly
  in history/replay, linked to real evidence, and never executed. A retry,
  including on an already completed run, causes no duplicate work or result
  delivery.
- Custom node IDs and catalog-derived nodes work. Consultation-only flows
  do not accumulate irrelevant warnings. Reporting self-checks acknowledge
  that unrecorded work is unknown rather than asserting an omission.

After deployment, run a small real freework trial whose request asks for
work without telling the agent to add presentation/cleanup nodes. Observe
whether the warning leads it to add them, present an accessible result and
clean up after review. Separately exercise retrospective recording with
known already-performed fixture work. Use a clearly identified check actor
for any agent-authored test feedback; do not impersonate the Developer.
Record actual behavior, including immediate `--yes` use if it occurs.

## 6. Deployment and report

Before inspecting or changing local running services, read their state
through `pj-clusterintent` Nautobot or `nctl`, following
`pj-clusterintent/nctl/README.md`. Deploy WFE and executor guidance through
the established release paths and update applicable worker pins. Verify
the deployed behavior, including access from another LAN host where the
changed UI or presentation path is involved. Leave the Developer a result
that can be checked with minimal operations.

If the event schema changes, prepare the necessary store backup and a
matching code/data recovery procedure; backward compatibility is not a
requirement. Keep hostnames, addresses, credentials, deployment details and
machine-specific evidence paths in ignored local files.

Write `report.md` here with changes, validation, trial run references,
remaining limitations and recovery references. Add a concise fix5 entry to
`devdocs/README_DEV.md` and update ignored local development notes where
needed. Do not rewrite the historical runs merely to make the new checks
pass.

## Execution policy

Follow `localrule.md`: generated files are English, chat is Japanese,
backward compatibility is unnecessary, and implementation/deployment needs
no additional human permission. The warning-and-`--yes` behavior above is
the explicitly agreed product interaction, not a human approval gate.
The current task creates this plan only; implementation and deployment
steps describe subsequent work.
