# project_migration p5/pre1 — Step 2 report: destination fit and design alternatives

Status: complete (2026-10-16). Deliverable: [options.md](options.md).

## What was examined

- The installed WFE contracts (`pj-agdev/agdevworld/workflow_editor/docs/`: contract, runs, storage, intake, io, integration, delegation, access, sessions, observation) and the executor side (`pj-agdev/agautolab` `wfexec`, the `wfrun` role grant, pyagag's harness argv), at the revisions recorded in report1.
- The forge (p3, p3 ex1) and archsage (p4) migrations: what moved, what retired, what was left to later, which evidence was accepted.
- report1's inventory of cagent and the backend.

## Findings that shaped the options

1. **Neither system enforces authority below the conversation.** The executor runs every `claude_code` attempt with permissions skipped and an advisory tool list; bindings are self-checks; cagent's own doors have no permission engine either. Authority therefore has to be placed where it is real: who may open which workflow (intake and delegation configuration), plan agreement before effects (a reducer-enforced step), and a project tool that performs effects and records them.
2. **External effects are precedent, not contract.** WFE has an external wait, hold-and-recover wording and forge's record-before-act convention, but no effects ledger, approval gate, target lock, scheduler or undo. The options put an effects ledger in the run folder and verification by fresh observation, which maps onto nctl's existing operation records.
3. **Reports have no home anywhere.** Nautobot stores purpose only on the desired side; nodeutils' self-report fields are never populated under nctl; the owner/purpose columns were deleted as unused. The options propose project-owned report records with provenance, verified by read-only reads, and leave a Nautobot-side model to a later phase.
4. **The only scheduler is the launchd refresh job**, and it runs a bounded `reconcile --yes`. The options keep it as observation only and treat a WFE scheduler as deferred.
5. **Every previous migration left Front's routing to Front's episode** and leaked host details into published records at least once. Both are named in the migration section so they are decided, not repeated by default.
6. **The state platform question is separable.** All of Nautobot, nctl, nodeutils and Ansible have responsibilities that would need a new home; the braindump already expects replacement to be its own phase. The options keep the stack and wrap it.

## What options.md contains

- The destination in one paragraph, for the Developer who should not have to read the contracts.
- Nine decisions (boundaries, state ownership, desired versus one-time, authority, lifecycle, failure and concurrency, self-management, stack disposition, migration) each with a smallest proposal, an alternative, evidence, tradeoffs, a recommendation and assumptions.
- A capability and gap matrix separating cagent-specific work, reusable facilities, missing common capabilities and deferred work.
- Six paper walkthroughs (a delegated report with a duplicate, a question with stale data, a one-time change, a maintain request, a change on the command node, an interruption mid-effect).
- Ten prioritized questions.

## Not done, by design

No prototype, no new project repository, no change to any service. Step 3 is the discussion; the design is written only after it.
