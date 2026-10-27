# Tidy p2 — reminders at reporting and result assessment

Status: planned. Date: 2026-10-27 (JST).

## Objective

Help producing agents report maintained, usable results with evidence, and help receiving agents check that evidence before accepting or summarizing another project's work. Apply this to delegation generally, including ponderer; do not make it a ponderer-only rule.

Success means a receiving agent independently notices a relevant gap, returns it to its owner, and verifies the correction. Displaying a reminder or completing a tidy node is not sufficient evidence of success.

Inputs: `../p1/report.md`, `../p1/report3.md`, `../p1/report4.md`, `../p1/problem.md`, the current devpolicy standard, WFE's reminder/assessment contracts and `localrule.md`.

## Evidence motivating the change

- In the p1 trial, ponderer accepted a nonexistent protoprey report and a lowerpoly README containing machine-specific instructions. Omni Agent had to name both defects before revisions were requested.
- The new `document-missing` assessment was already visible before the intervention, but ponderer did not use it to reassess the result.
- A subsequent protoprey task added a test outside the test class; the reported passing count did not include it. Omni discovered the ineffective test.
- The agsfx adoption pointed to a producer's local run copy, not a published shared output. Protoprey's tidy read the adoption record without verifying acquisition/status and missed the portability defect.
- After correcting two children, ponderer reported only one correction. The existing `conversation-unanswered` check did not catch this because an earlier reply already existed.

These observations support reminders about concrete evidence at work boundaries. They do not justify mandatory tidying after every action, a fixed checklist for every project, or a new approval process.

## Execution policy

- Follow `localrule.md`: files in English, user-facing communication in Japanese. Implement, test, commit, push and deploy the authorized work without approval gates. Backward compatibility is not required.
- Use published revisions and current run records; p1 findings may have been corrected by later work. Preserve unrelated and concurrent changes.
- Consult Nautobot/nctl before relying on or changing local services, following `pj-clusterintent/nctl/README.md`. Store machine-specific addresses and deployment evidence in ignored notes.
- Leave the mechanism deployed and its trials easy to inspect. Any review service is reachable from the LAN. Do not ask the Developer to do setup the agent can do.
- Record interventions honestly. Never attribute synthetic test requests, decisions or reviews to the Developer.

## Design

### Producer boundary: reporting a result

At an existing result-reporting boundary, provide a short prompt to consider whether the change needs tidying and whether the report supports its claims. Relevant boundaries include a run's result/completion and a result posted into a conversation; map the actual paths before implementation.

For ordinary development, the message asks whether the maintained entry, guide and acquisition/setup instructions still match the delivered change. If tidying is needed, perform it through the existing activity rather than merely asserting completion.

For work that already performed tidy, ask for evidence rather than another tidy pass:

- Which files or records changed, and at which revision?
- Which checks actually ran, with what results?
- Are the report and referenced artifacts present at the stated locations?
- What remains uncertain, deliberately unchanged or unresolved?

Keep optional documentation semantics: an unrequested report is not made mandatory. If an agent claims that a report exists, or completes a documentation step, the existing missing-document checks still apply.

### Receiver boundary: judging delegated work

Deliver the guidance alongside the child's result assessment in the receiving run's brief/take-up/show output, before the receiver makes its disposition. Do not rely exclusively on the agent voluntarily opening an extra help command. Reuse the same assessment rendering at the existing acceptance/reporting boundaries where appropriate.

The message asks the receiver to compare the outcome claim with the actual documents, artifacts and verification records. State known missing facts prominently. Where the system cannot assess correctness, explicitly ask the agent to inspect rather than declaring a defect.

If the result is incomplete, the receiver returns a concrete correction to the owner through the existing same-child revision mechanism when applicable, then checks the revised result. A prior judgement does not establish that a newer result has been checked.

### Contextual prompts, not one large checklist

| Work | Relevant questions |
| --- | --- |
| Documentation or tidy | Does the current entry agree with implementation and adopted deliveries? Do references resolve? Are run history and current state distinguishable? Are portable requirements retained after removing local host details? |
| Artifact delivery or integration | Is the output published and identified by hash? Can a fresh consumer obtain it without the producer's working copy? Which installation/runtime checks were performed? |
| Cache removal or file moves | Can the needed files be regenerated? Are original assets and evidence retained? Have references and consumers been checked? |
| Tests or verification changes | Did the named checks actually execute, including newly added tests? Do counts and results support the report? |
| Several children or corrections | Does the final summary account for each commissioned result, its disposition and remaining limitations? An earlier acknowledgement is not a final account of all outcomes. |

Select prompts from available request, node/template and artifact context. A project name, filename or keyword alone must not establish that a problem exists. When applicability is uncertain, use the short general prompt and let the agent choose relevant checks. Do not introduce an LLM classifier or mandatory structured audit form merely to select reminder text.

### Authority, limits and repetition

- Keep shared principles in devpolicy, tool usage in help, and runtime facts in assessment output. Avoid independently maintained copies of the standard in every project guide.
- Preserve the existing distinction between verified facts, missing records and agent judgement. A tidy node's presence/completion is not proof of quality; its absence is not proof of a problem.
- Prefer advisory guidance in the result/brief for judgement questions. Use the existing reminder/override behavior for factual omissions where appropriate; no new approval state or permanent gate.
- Keep messages short and combine overlapping existing reminders. Do not issue the full producer and receiver checklist on every command.
- Bind result guidance to the actual delegation/result event and current evidence. Corrected results must receive fresh assessment; avoid a generic acknowledgement that suppresses later defects.
- Do not create a new tidy task solely because a tidy result was received. Stop when the relevant evidence supports the result, or record the remaining limitation honestly.

## Step 1 — Inspect current boundaries and establish the baseline

1. Refresh p1 open findings, particularly agsfx publication and protoprey's adoption. Record their current published revisions and actual state; do not assume the defect still exists.
2. Trace producer result paths and receiver assessment/disposition paths, including conversation result/synthesis and ponderer's final summary. Inspect WFE `shared/reminders.ts`, `shared/assessment.ts` and their callers, plus existing guides and contracts.
3. Identify where existing `document-missing`, delegated-result and `conversation-unanswered` messages are already delivered, where they overlap, and where an agent can miss the relevant facts.
4. Specify the smallest shared rendering/data change needed to put the appropriate prompt at each boundary. Keep factual checks separate from advice to investigate.

Deliverable: concrete implementation points and a before/after account of what the sender and receiver will see.

## Step 2 — Implement and verify the reminders

1. Implement concise producer guidance and receiver evidence prompts through the existing machinery. Include links or identifiers for the actual run, result, report and artifacts when available.
2. Preserve the existing factual missing-document behavior. Improve presentation of outstanding findings so they are not buried under a success outcome line.
3. Make multi-child summaries explicitly consider every result and outstanding correction, without equating any prior reply with a complete summary. Start with available records and contextual guidance; introduce further state only if a demonstrated gap requires it.
4. Adjust the tidy template or shared guidance only where necessary, keeping one authority for the meaning. Existing run catalog snapshots remain historical; do not silently rewrite them.
5. Add focused tests for boundary delivery, known missing facts, uncertain applicability, revised results, unrelated small work, optional documentation and absence of reminder loops. Cover the normal override behavior and both CLI/API paths where they share the operation.
6. Run the relevant checks and required release suite. Deploy coordinator and affected workers using the existing release mechanism; verify the active versions and the actual rendered messages. Record recovery steps appropriate to any schema change; prefer no schema change if unnecessary.

Deliverable: tested and deployed guidance, without adding a mandatory tidy gate.

## Step 3 — Observe one unprompted end-to-end trial

Use a newly created ponderer-led request that asks for a concrete usable result and its final summary, without telling ponderer which defect to find or giving it a manual inspection script.

Preferred real case, if still open:

1. Obtain a portable agsfx delivery for protoprey's location sound effects.
2. Have the owning project publish it and describe the offer; have protoprey adopt the published reference and verify acquisition from a fresh copy.
3. Observe the sender's report, ponderer's assessment, any correction request and the summary across the involved projects.

If the real defect is already resolved, use another confirmed open finding or an isolated fixture with a deliberately incomplete result. Do not regress production data to manufacture a test. Keep fixture limitations known to the evaluator, and let the receiving agent discover them through the normal result path. Include a missing/contradictory evidence case and a sound result so blanket rejection is not mistaken for good assessment.

Record:

- The exact guidance each agent received and the relevant event/revision.
- Which documents/artifacts/check results the receiver actually inspected.
- Whether it discovered a gap before any Omni hint, returned it to the correct owner, and verified the corrected result.
- Whether the final summary covers every child and distinguishes completed work from unresolved or unverified claims.
- Whether unnecessary tidying, repeated warnings or circular correction occurred.

Do not intervene merely because the agent is taking time. If it wrongly accepts the defective result, record an independent failure before intervening. Improve the responsible mechanism and repeat the affected trial in a fresh context; do not count the assisted correction as independent success.

Deliverable: evidence of behavior, not just a screenshot showing that reminder text appeared.

## Step 4 — Verify the delivered work and assess the remaining gap

1. Independently check the trial artifacts, report existence, published revisions and claimed tests. For the agsfx case, verify hash-based acquisition from a clean consumer copy without the producer's run directory.
2. Verify any newly added test is discovered and executed, not merely present in the source.
3. Compare sender completeness and receiver assessment with p1's observed failures. One trial cannot establish reliability; state the sample size and any interventions.
4. If the reminder is visible but the receiver still accepts without inspection, treat this as an unresolved behavior problem. Consider a small evidence-backed guide/tool improvement and another trial, rather than escalating automatically to rigid gates or a new agent project.
5. Confirm ordinary, unrelated small work is not forced through a full tidy audit and a correct tidy result does not recursively trigger more tidy work.

## Step 5 — Publish and hand off

- Commit, push and confirm integration of all intended changes in their owning repositories. Preserve concurrent work and report exact published revisions.
- Update shared development documentation, p1 finding dispositions where resolved, and local deployment notes. Keep the scope of this improvement separate from a claim that all projects have been tidied.
- Write `report.md` here with boundary changes, evidence, test/deployment results, trial outcomes, interventions, remaining limitations and recovery information.
- Provide the Developer with the trial conversation and concise examples of the sender/receiver messages. Leave any required review artifact ready to inspect with few actions.

## Completion criteria

- Relevant producer and receiver boundaries deliver concise, contextual guidance through normal execution paths.
- Factual omissions and judgement questions are visibly distinguished; optional documentation and existing override semantics remain intact.
- An unprompted receiving-agent trial detects a real or isolated-fixture evidence gap, requests correction from its owner and verifies the correction, without treating all results as defective.
- Multi-project results are summarized with their individual outcomes and remaining limits.
- The actual trial output is usable and independently verified; any corrected publication is retrievable outside the producer's working copy.
- Focused tests and required release checks pass, affected deployments are updated, and no repetitive tidy/report cycle is introduced.
- Independent failures and unresolved behavior are explicitly recorded. Do not mark the behavioral objective achieved solely because the implementation is deployed.

## Out of scope

No dedicated tenderer, mandatory tidy node in every run, blanket directory validator, new human approval gate, automatic cleanup of arbitrary projects, or all-project migration. P2 improves reporting and assessment at observed failure points; it does not prove general autonomous review reliability.
