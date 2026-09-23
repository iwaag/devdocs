# Robust workflow p1: Proposed solutions and work approach

This is an AI-authored proposal based on [braindump.md](braindump.md) and the discussion that followed. It does not describe an implemented specification or confirmed root causes.

## Desired outcome

After a request reaches Front, work progresses without the Omni Agent having to locate stalls and prompt agents to continue. If work cannot proceed, the requester learns what happened, what is being awaited, and who needs to decide or act next. Even after recovery, the failure and the recovery history remain available to inform further improvements.

To achieve this, I support the braindump's primary approach: simplify the system and reduce both error-prone steps and guide rules. Develop Observer's detection, recovery, and incident recording alongside those changes so that work can continue while the workflow is still being developed.

## What the existing evidence shows

The [adventure_game p3 step 2 report](../../milestones/adventure_game/p3/report2.md) records the following cases.

| Case | What the record establishes | What still needs investigation |
|---|---|---|
| A workplan was resolved immediately after being opened, and another topic was created | autolab had already started processing the original conversation, leaving two conversations | The input and tool results that led to the resolve, and the available recovery operations |
| Front reported starting task 4, but no start post existed, causing a 24-minute stall | The report did not match the actual action, and Front did not notice the stall | Whether the action was omitted, failed, or its result was misread |
| Front waited for task 5's result without starting it | Responsibility for triggering the next operation was not fulfilled | What was visible at the handoff and whose action was expected |
| Task 4's content was worked on in task 3's conversation, but task 4 still required a start post | The boundaries between actual work, task initiation, and task records were unclear | How responsibility was divided between tasks and whether unnecessary relays can be removed |

These observations come from reports; they are not root-cause findings from a fresh examination of all execution logs. The [phase report](../../milestones/adventure_game/p3/report.md) already describes guide additions and proposes another trial without Omni Agent prompting. This episode should investigate both whether those additions work and the structure that made them necessary.

## Proposed solutions

### 1. Examine the actual task given to the agent before attributing failure to the model

Before concluding that the model lacks capability, examine the input, guides, available tools, permissions, execution environment, and operation results. The Omni Agent's ability to perform an operation does not establish that the in-system agent can perform it.

What looks to a human like "just start the next task" may require the agent to coordinate multiple topics, approvals, start conditions, return destinations, and completion conditions. Keep the model fixed and measure the improvement from reducing that burden. There is no need to rule out the model's contribution entirely, but changing models should not be the first remedy.

### 2. Clarify responsibility for handoffs and reduce relays that require no judgment

Add "distributed responsibility" and "poor visibility into handoff state" to the candidate causes.

First, use actual conversations to map who handles planning, approval, initiation, result review, and progression to the next task across Front, autolab, and forge. In particular, if autolab can advance through its own tasks within an approved scope, reconsider whether Front needs to relay a start post for every task.

Distinguish boundaries that involve decisions such as accepting a delivery, incurring additional costs, or changing scope. Work requiring human acceptance must not be automatically accepted merely to reduce steps. Carry forward what an existing approval authorizes so that the same permission does not need to be obtained again.

Decide specific ownership changes after investigation. The aim is to reduce coordination work that offers little value for agent judgment, rather than turn every activity into a fixed procedure.

### 3. Make the facts of an operation inspectable

A natural-language report saying "started" is not evidence that execution began. A post ID alone is also insufficient to establish that the recipient started executing.

At a minimum, it should be possible to trace whether a request was posted, accepted by its recipient, is being executed, produced a result, and delivered that result to the requester. Legitimate waiting for a human and an inability to observe the state must also be distinguishable.

Prefer existing message IDs, rootchat notes, task records, and the serving journal. Avoid adding steps that manually copy the same facts into another ledger. First provide tools that connect and read the existing records, then add only the minimum records needed at boundaries that remain unclear. Failures and uncertain results returned by tools should be traceable from the same request as successful outcomes.

Approach this as Tool Giving. Make the necessary state and evidence available through fewer operations, rather than adding guides that require a specific sequence of commands in every situation.

### 4. Make Observer's coverage independent of Front remembering to register a watch

If every watch depends on Front explicitly registering it, Front can omit the watch just as it can omit a start post. Explore ways to discover observation targets from ongoing requests and incomplete handoffs.

Observer should compare evidence with what is expected to happen next, rather than infer a stall from elapsed time alone. It needs to distinguish work that has not started, work in progress, pending approval, execution failure, missing result delivery, and an unobservable state. Use tools to obtain facts that records establish directly, and agent judgment where the meaning of a conversation needs interpretation.

The current observe role is limited to observation and judgment. Assigning recovery and incident recording is an expansion of its role, not something to treat as a one-sentence guide addition. Use the following division of responsibility as a starting point.

- Observer identifies the evidence of the stall, the affected request, the preceding operation, and the expected next action.
- The original responsible agent reads the conversation's authorization scope and current state, then resumes the work.
- Recovery that requires no semantic judgment, such as redelivery, uses the existing delivery and retry mechanisms.
- If recovery is not possible, the requester receives the reason and the decision needed.
- Observer connects detection, the recovery request, and the recovery outcome in one incident record, confirms recovery, and ends the watch.

A recovery request can itself trigger another execution, so repeated nudges alone are insufficient. Make duplicate notifications and duplicate starts for the same failure identifiable, and define retry limits and a reporting destination when recovery fails. If the return destination is lost, a display or reporting route must still make the stall visible outside the original conversation.

### 5. Consolidate guides after correcting implementation, tools, and responsibilities

Look beyond guide length to differences in how roles explain the same concept. For example, Front's guide describes resolve as a reversible rename, while the Desk guide describes a resolved conversation as finished. Recovery decisions can become inconsistent if ordinary completion and accidental resolution are not distinguished.

This does not establish that both guides are supplied as conflicting instructions in the same execution. Inspect the guides and generated context actually supplied to each role, and align their explanations with implementation behavior.

For each instruction, determine whether it is a contract that is still needed, a restriction from a past implementation, or something the tools already guarantee. Remove obsolete instructions and maintain shared contracts in one place. Guide changes remain a valid remedy: evaluate the smallest explanation that addresses an observed cause through another trial.

## Proposed work sequence

### Step 1: Reconstruct failures and establish a baseline

For the three stalls or erroneous actions in adventure_game p3, correlate conversations, execution transcripts, tool results, and listener records. Distinguish the guides in effect at the time from the current guides, and separate missing inputs, incorrect explanations, tool defects, insufficient permissions, omitted actions, and inaccurate reports.

At the same time, map responsibilities and the current successful path from Front's request through result delivery. Use Nautobot or nctl to understand the running environment, and assess cluster health separately from conversation workflow health.

Deliver a failure analysis with evidence references, a handoff inventory, unresolved questions, and baseline counts for steps, stall duration, and interventions. Keep local environment details and secrets out of shared documents.

### Step 2: Correct mismatches between the execution environment and available operations

Check whether instructed operations can be performed using the target agent's credentials, tool grants, working directory, environment variables, and deployed versions. Provide necessary operations to the responsible agent or move responsibility to an agent that can perform them, without unnecessary permission expansion.

Fix defects with a minimal reproducing case. If an alternative method completes the work, preserve the failure of the original operation and the fact that a substitute was used.

### Step 3: Simplify one boundary responsible for frequent handoff errors

The first candidate is the boundary between an approved plan, starting its individual tasks, and advancing to the next task. Evidence from Step 1 may change that priority.

Compare consolidating ownership, combining related operations, and making operation results clearer. Choose the option that removes the most unnecessary relays. Update the corresponding guides and introductions for other agents together, removing obsolete procedures.

### Step 4: Enable Observer to detect these failures and coordinate recovery

Initially limit the scope to failures confirmed in Step 1. Connect target discovery, evidence collection, recovery requests to the responsible agent, recovery verification, and incident recording.

During rollout, also check that legitimate long-running work and waiting for a human are not misclassified as stalls. Treat unreadable records as an inability to observe; do not assume the work is healthy or has not started and rerun it on that basis.

### Step 5: Verify without Omni Agent prompting

Test both the normal path and deliberately introduced failures. Use small verification requests that do not damage real project deliverables to check the following.

| Case | Expected outcome |
|---|---|
| An ordinary request with several sequential tasks | Work progresses within the approved scope, and results reach the requester |
| A start operation is omitted | The missing start is detected, and the original responsible agent resumes the work |
| A reply or its delivery fails after posting | Delivery is recovered or failure is reported without duplicating existing work |
| A required command or permission is unavailable | The problem remains recorded with evidence and is not reported as success |
| A topic is renamed or accidentally resolved | The link to the original request is preserved, and recovery or reporting occurs without creating duplicate work |
| Legitimate long-running work or waiting for a human | No unnecessary reruns or nudges occur |
| Observer cannot read its target | The inability to observe is explicit, and no unsupported recovery action is taken |

Measure completion rate, time to detection, time to recovery, false positives, duplicate execution, Omni Agent interventions, and additional execution cost. Set timing targets and trial counts after establishing the baseline and before starting verification. One successful trial is not sufficient to establish reliability.

The Omni Agent prepares and observes the tests, while in-system agents locate stalls, issue prompts, and perform recovery during each trial. Count any necessary intervention as a failure and record whose work was performed on their behalf as a handoff candidate. Count ordinary human approvals and evaluations separately from rescue interventions.

## Completion criteria for p1

Representative failures have evidence-supported explanations, at least one handoff boundary has been simplified, and instructions made unnecessary by that change have been removed. In addition, the normal path and agreed failure cases must either complete or recover without Omni Agent rescue, or deliver a stop report to the requester that identifies the reason and the next responsible party.

Record successful recovery separately from elimination of the cause. Defects that Observer merely rescued remain improvement candidates. Use recurring incidents to prioritize changes to design, tools, and guides. This cycle advances Easier Next Time while keeping work productive.
