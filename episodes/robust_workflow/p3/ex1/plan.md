# Robust workflow p3 ex1 — Isolated work and deliberate integration

Sources: [p3 report](../report.md), [trial findings](../report5.md), and the follow-up review.

## Goal and scope

Remove the known risk of concurrent or cancelled autolab missions contaminating another mission's changes. Establish a clear boundary between saving work, accepting it and integrating it, then resume ordinary project development with failures recorded as they occur.

- This is a private experimental environment and a breaking phase. Backward compatibility is unnecessary. Implementers may replace interfaces, code and disposable state; choose the smallest coherent design.
- Prefer structural isolation and explicit operation results over additional guide prohibitions. A mission worktree is a candidate, not a required implementation.
- Keep Observer on local. The [Sonnet comparison decision](../report6.md) remains deferred under its recorded triggers.
- Wider discovery, general queue identity refactoring and cosmetic reply cleanup are follow-up work unless this change demonstrates a dependency. Do not turn this extension into another broad workflow-hardening phase.
- Preserve meaningful requester decisions and keep secrets and machine-specific details in ignored files. Routine implementation choices need no additional approval.

## step1 — Reproduce contamination and define the acceptance boundary

- Inspect autolab's workspace selection, worker environment, task close-out, commits, pushes, mission acceptance and cancellation. Check relevant running services and observation freshness through Nautobot or nctl.
- Reproduce two missions editing the same file and cancellation leaving uncommitted edits for a later mission. Record which repositories and generated files are shared; include direction/devlog repositories where applicable.
- Define the working contract: workers may checkpoint changes in isolated branches before acceptance. Such a commit is not a claim of acceptance, integration, publication or completion.
- Identify the existing decisions that authorize integration and publication. Preserve task and mission review points without adding another approval round merely for implementation bookkeeping.

Done: `report1.md` names the contamination paths, the chosen isolation unit and the acceptance/integration contract.

Hints: p3 E∥F both edited `wordcount.py`; cancelled F's `--lower` edits remained until G discarded them. G, H and K committed before agreement. The p3 close-out guard prevents task closure, not commits. Do not simply ban all early commits: isolated checkpoints are useful. Make the restriction apply to the shared result and the actual authorization boundary.

## step2 — Give each independent mission its own working state

- Create and recover working state by stable mission identity, including its base revision and target branch. Sequential tasks in a mission may share that state; independent missions must not share mutable source files or an index.
- Route worker commands, tests, staging and commit operations to the correct workspace. Isolate conflicting generated outputs where needed; share immutable inputs and caches when practical.
- Make cancellation stop further automatic progression and integration for that mission. Retain or remove its isolated workspace according to a simple lifecycle; either choice must leave other missions and later requests unaffected.
- Handle existing dirty shared work explicitly during rollout. Attribute or set aside uncertain changes rather than silently assigning them to the next mission. No historical workspace migration framework is required.
- Make restart and repeated initialization recover the same mission's work instead of creating another competing checkout.

Done: concurrent missions and a cancelled mission can each leave changes without those changes appearing in another mission's working state or commits.

Hints: worktrees isolate files and indexes but still share repository refs. Check scripts that assume the project's canonical checkout, broad staging commands, fixed temporary paths and shared report outputs. Cancellation need not erase useful work; isolation is the requirement.

## step3 — Integrate accepted changes and record what actually happened

- Connect integration to the authorization identified in step1. Bind the accepted work to a concrete revision or change set and retain the accepting post as evidence.
- Coordinate updates to the shared target so concurrent integrations cannot overwrite each other. If the target moved, reconcile and verify the combined result; conflicts or material changes outside the acceptance return to the responsible agent or requester for a decision.
- Record distinct outcomes for checkpointed, accepted, integrated, published where applicable, and complete. Use the existing work record and receipts rather than a second manually maintained ledger.
- Make retries recognize already-integrated work. A crash after a merge or push but before the completion record must recover the record instead of repeating the operation or claiming failure without checking.
- Preserve p3's task progression, `agentchat accept` and mission completion paths. A task or mission must not report a promised shared result as complete while required integration or publication remains outstanding.
- Update autolab's introduction, guides and close-out messages together; remove explanations contradicted by the new contract.

Done: accepted changes reach the intended target with evidence, while cancelled or unaccepted work stays out. Repeated close-out and interrupted integration produce one coherent result.

Hints: branch names alone are not accepted content identities. Check the commit/tree actually being integrated. A small serialized integration operation may suffice; a general scheduler is unnecessary. Do not require fresh human approval for an unchanged, already-authorized result merely because an operation is retried.

## step4 — Verify the narrow change on a fixed revision

- Run focused fixtures for workspace ownership, cancellation, concurrent target updates and interruption. Then deploy and verify installed revisions and running consumers.
- On the final version, run a normal multi-task mission and the scenarios below through Front. Repeat the conflicting-mission case twice, changing which mission finishes first. Use disposable project work for deliberate faults.

| Scenario | Expected result |
|---|---|
| Two missions edit the same file | Each reviews its own change; integration preserves accepted work or exposes the conflict |
| One mission is cancelled with dirty files or checkpoints | Other and subsequent missions contain none of its changes |
| Worker checkpoints before acceptance | Work is saved in isolation; the shared target and completion record do not imply acceptance |
| Target moves after review | Integration checks the actual combined result and surfaces material conflicts |
| Restart after checkpoint or during close-out | Same workspace resumes; already-integrated work is recognized |
| Repeated acceptance/integration request | No duplicate work, extra merge or false completion |
| Task and mission explicitly accepted | Required integration finishes, mission reaches `done`, and Observer releases the request |

- Verify actual diffs, branch heads and work records, not only agent reports. Record any change discovered in the wrong scope, unnecessary approval round, mission-recording reminder or rescue intervention.
- Count fixtures and live evidence separately. After a fix, repeat affected cases and the normal cycle on the final version; retain failed attempts in the report.

Done: no cross-mission change contamination, cancelled work integrated, accepted work lost, duplicate integration or false completion in the agreed trials. P3's receipt and completion behavior still works.

## step5 — Hand back to ordinary project development

- Write `report.md` with the final contract, deployed versions, trial evidence, remaining limits and practical instructions for starting, cancelling and resuming missions.
- State whether same-project parallel development is ready. If a known correctness problem remains, name the narrow temporary operating limit rather than declaring the workflow generally stable.
- Resume the next real project request through Front when the requester supplies or selects it. This extension need not invent a production task to finish. During real work, record rescue follow-ups, missed completion records, change contamination, duplicate work and silent stalls as improvement candidates.
- Keep Sonnet testing and wider discovery deferred unless their recorded triggers or the next project's entry path justify them. A few passing trials establish readiness to practice, not unattended reliability at scale.
- Commit and push implementation, documentation and necessary submodule references.

Done: the known work-isolation hazard is addressed and the workflow is ready for real project practice, with a short list of evidence-driven follow-ups.
