# Robust workflow p1 — Reliable handoffs with fewer steps

Sources: [braindump.md](braindump.md), [opinion1.md](opinion1.md), [opinion2.md](opinion2.md).
Initial evidence: [adventure_game p3 report](../../milestones/adventure_game/p3/report.md) and [step 2](../../milestones/adventure_game/p3/report2.md).

## Goal and working approach

A request given to Front progresses within its authorized scope without the Omni Agent locating stalls and prompting agents. Failures are detected, recovered where possible, and recorded. When progress is impossible, the requester receives the reason and the next responsible party.

- This is a private experimental environment and a breaking phase. Backward compatibility is unnecessary. Replace interfaces, code and disposable state where useful; implementers choose the design, tools and recovery mechanisms.
- Prefer removing unnecessary handoffs and providing useful tools over adding instructions. The steps below specify outcomes, not a mandatory runtime procedure for agents. Adjust priorities when evidence supports a better boundary to simplify.
- Preserve meaningful human decisions: permission to execute a plan is not acceptance of its deliverables. Carry existing authorization forward without asking for it again.
- Keep secrets and machine-specific details in ignored files. No additional security framework or broad non-destructive policy is required for this episode.
- Keep the execution model fixed for comparable trials. Record any necessary model or environment change. Use focused tests for actual failure modes.

## step1 — Reconstruct representative failures and establish the baseline

- Start with p3's accidental workplan resolve, task 4's claimed but absent start, and task 5's wait without a start. Correlate conversation IDs, execution transcripts, tool results, listener records and the guides actually supplied at the time.
- Separate omitted actions, failed operations, unavailable tools or permissions, misleading context and incorrect reports. Allow unknown or multiple causes; reports alone do not establish root causes.
- Map planning, approval, task initiation, review, progression and delivery across Front, autolab and forge. Identify relays that require no judgment and approvals that genuinely belong to a human.
- Inspect the affected roles' credentials, tool grants, working directories, environment and deployed dependency revisions. Check local services through Nautobot or nctl, including observation freshness; assess conversation workflow health separately.
- Consult p1/p2 and routine incidents to check recurrence, without making an exhaustive historical audit a prerequisite for implementation.

Done: `report1.md` records evidence, unresolved questions, the handoff map, and baseline counts for stalls, relays, rescue interventions, runs and cost where available.

Hints: p3 records a 24-minute stall and work for task 4 performed inside task 3's conversation. Front's current guide explicitly asks for a proposal and permission before delegation; distinguish this contract from unnecessary repeated approval. Planning-time nctl checks passed, but Observer liveness was unobserved: cluster convergence does not prove workflow health. The guides added after p3 are not evidence of what the failing runs saw.

## step2 — Make request progress inspectable from existing evidence

- Provide a shared read operation that follows an originating message ID through mission/task records, outgoing requests, recipient servings, results and delivery. Make it usable by Front, Observer and the existing views as needed.
- Distinguish not started, queued, executing, awaiting a human, failed, awaiting delivery and unobservable. Include evidence references and observation freshness; expose uncertainty when the available records cannot establish a state.
- Reuse existing records and add only missing evidence at actual operation boundaries. Capture operation failures, uncertain outcomes and substitute methods without requiring agents to copy facts into another ledger.
- Fix demonstrated tool/environment mismatches and defects with small reproducing cases. Verify access from the affected role's actual execution environment, not only from the Omni Agent's shell.

Done: representative requests can be inspected end to end, including their failure or visibility gaps, using tools available to the responsible agents.

Hints: `pyagag/src/agag/chat.py` already returns a message ID from `send` and supports conversation reads. `agag.serving` records received, acked, executed, prepared and delivered states; `agag.delivery` retries prepared replies without rerunning the model. Rootchat anchors, continuation context, mission/task notes and mirrors also exist. Extend or simplify these before introducing another store. A specification post is not a start request; a posted start request is not proof that execution began. An acknowledgement alone does not prove a worker remains active.

## step3 — Remove one unnecessary handoff boundary

- First consider assigning progression within an approved mission to autolab, eliminating Front's separate start post for every next task. Front retains request interpretation, coordination between agents and communication with the human.
- Preserve required review points. After the authorized acceptor accepts a task, autolab should be able to proceed to the next already-authorized task without another mechanical relay. Make explicit when progression must wait for a decision or changed scope.
- Choose the smallest coherent change supported by step1. Combining related operations or clarifying task ownership may be better than a general workflow engine. Make repeated progression requests recognize work already started or completed.
- Improve mistaken resolve and routing recovery where the evidence calls for it. Base checks on request identity and actual operation semantics, rather than a blanket prohibition on resolving recently created topics. Keep legitimate cancellation and correction practical.
- Update the affected guides and introductions with the implementation, removing obsolete procedures and incident-specific workarounds. Keep shared contracts in one place where practical.

Done: at least one error-prone relay is removed, its replacement works across interruption/retry, and the instructions it made unnecessary are deleted.

Hints: autolab's current introduction says each task requires its own post, tasks run in order, and the requester must accept completion. Ownership changes must update that public contract as well as code. `agentchat` already rejects sends to resolved topics, while resolve itself is a rename; its current error advice may encourage creating a replacement when correcting the original would be appropriate. Investigate the whole recovery path. Guide length is a secondary metric, not a target achieved by moving the same burden into help text.

## step4 — Connect Observer detection, recovery and incident recording

- Discover observation targets from active requests and incomplete handoffs so coverage does not depend on Front remembering to register a watch. Initially cover the failure classes demonstrated in step1.
- Extract mechanically established facts in code; use Observer judgment for conversational intent, expected next actions and ambiguous waits. Elapsed time identifies candidates, not proof of failure.
- Request recovery from the original responsible agent with the affected request, evidence and expected next action. Use existing listener delivery recovery for missing replies where applicable. Verify the result rather than treating a nudge as recovery.
- Identify repeated detections of the same incident, bound retries and provide a visible reporting route when recovery or the original destination is unavailable. Avoid duplicate work when an operation's outcome is uncertain.
- Keep detection, recovery requests, outcomes and evidence in one incident record linked to the original request. Record successful rescue separately from removal of the underlying cause.

Done: a demonstrated stall is discovered without manual watch registration, recovered or reported to the requester, and recorded without Omni Agent rescue.

Hints: the current `observe` role only looks and returns `met`, `not_met` or `unable`; recovery coordination needs an actual capability change. Keep its observation-only role or replace it deliberately. Ordinary posts can trigger paid runs, so choose notification destinations and dispatch carefully. Observer cannot directly read arbitrary remote-host paths. Missing evidence is unobservable, not proof that work never started. Existing mirrors avoid repeated realm sweeps; reuse them and measure added calls and evaluation cost.

## step5 — Verify normal operation and failure recovery without rescue

- Prepare a small multi-task request and a forge-to-autolab handoff, with concrete outputs and identifiable human review points. Exercise the ordinary entry and delivery paths.
- Test the cases below using fixtures and controlled live trials as appropriate. Include restart/retry at the changed boundaries and verify task identity through renames.

| Case | Expected result |
|---|---|
| Sequential authorized tasks | Work advances and results reach the requester without mechanical Front relays |
| Missing start | The responsible agent resumes the missing work after detection |
| Reply or delivery failure | Delivery recovers or failure is reported without repeating completed work |
| Missing command or permission | Evidence of the failure remains visible; no false success report |
| Accidental resolve or rename | The original request remains traceable; recovery creates no duplicate work |
| Long-running work or human approval wait | No unnecessary restart or recovery nudge |
| Unreadable target or stale observation | Visibility failure is explicit; no unsupported rerun |
| Repeated detection or listener restart | Recovery remains bounded and existing work is recognized |

- Initial trial budget: three normal cycles and two trials per failure case. After step1, set the final trial counts and detection/recovery time targets before verification; explain adjustments.
- Measure successful completion and explicit stop reports separately, plus detection/recovery time, false positives, duplicate execution, rescue interventions, runs, cost and additional API calls.
- The Omni Agent prepares and observes trials. Any intervention needed to locate a stall or advance the work counts as a failed autonomous trial; ordinary human approvals and quality evaluation are separate. Record substituted work as `did X for agent Y — handoff candidate`.

Done: the agreed trials meet their targets with no duplicate execution, false success reports or Omni Agent rescue. Unrecoverable cases deliver an actionable stop report. Record failed trials and fixes as well as successful repetitions; this sample is not proof of reliability at scale.

## step6 — Deploy the coherent change and report the remaining limits

- Apply the completed changes to affected consumers and verify the versions actually running. Deploy what is needed before live trials; this step closes any remaining rollout work.
- Update public introductions when behavior changes, align relevant developer documentation, and remove superseded guide text. Keep a short explanation of the new ownership and evidence paths.
- Write `report.md` with implemented changes, evidence, before/after handoff counts, trial results and remaining defects. Separate prevented failures, recovered failures and unresolved causes; use recurring incidents to select the next improvement.
- Commit and push changes and necessary submodule references under the repository workflow.

Done: one boundary is simpler, residual stalls are observable and recoverable in-system, and the result and its limits are reviewable.

Hints: pushing pyagag alone does not update consumers. Check their lockfiles, installed environments and running listeners; a separate deployment may retain an older revision. Most implementation belongs in pyagag, agfront, agautolab and agobserver, with forge and agdevworld updated where contracts change. Change pj-clusterintent when evidence shows an environment or shared-consumer need, rather than expanding its role into conversation orchestration.
