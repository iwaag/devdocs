# project_migration p5/pre1 — final report: the cagent migration design

Status: complete (2026-10-16). This phase investigated, proposed, discussed and wrote the design; it implemented and deployed nothing. Operational claims below are inventory reads at one moment, not verification.

Documents: [report1.md](report1.md) (implementation trace), [options.md](options.md) + [report2.md](report2.md) (alternatives), [decisions.md](decisions.md) (D1–D14), [design.md](design.md), [../plan.md](../plan.md) (implementation handoff). Host facts and raw evidence: ignored `pj-agdev/.local/project-migration-p5-pre1/`.

## Findings

- **cagent's authority story was prose.** The destroy-class guard documented for every door exists only on agcode profiles; the deployed node and human doors run claude_code with an unscoped Bash. Nothing in cagent or the destination enforces authority below the conversation.
- **cagent was idle.** No HTTP request since 2026-08-28, no open change request, no live node caller; the last servings were argue mentions in September. Its only autonomous cluster writes were August trials.
- **The backend's real executor is a person at a shell plus a six-hourly launchd job** that runs a bounded `nctl reconcile --yes` under the name "observation refresh". nctl alone owns inventory generation, SSH trust and operation history; Nautobot owns both ledgers; Ansible performs every host change and the collection itself.
- **Installation reports have no ingestion path and no actual-side storage.** Purpose lives only on the desired side; the actual-side owner/purpose fields were deleted as unused; the planned agent-to-cagent report path (refine_cluster_management p2) was never built.
- **The destination offers** isolated run copies, plan agreement, result confirmation, holds and Observer recovery, delegation with per-run IO channels, cross-project reads, and a credential pattern (ignored `.local` env files, redacted IO). It lacks an effects ledger, an approval gate before effects, a per-target lock, a scheduler and undo.

## Decisions (all with the Developer, 2026-10-16)

| Decision | Outcome |
| --- | --- |
| D1 boundaries | four workflows by authority (`consult`, `report`, `scan`, `change`) plus freework |
| D2 reports | project-owned Git records with provenance, verified by read-only reads; Nautobot-side model deferred |
| D3 drift | record and propose only; refresh job becomes observation only |
| D4 restrictions | destroy-class human-only; no effect before the Developer agrees the plan; approver = Developer (provisional) |
| D5 stack | keep Nautobot/nintent, nctl, nodeutils/nauto, Ansible; retire executor, HTTP doors, CA/ledger, listener, unused roles, webhook; state platform later |
| D6–D8 lifecycle | effects ledger by the project tool; verification by fresh drift; recovery reads ledger → `nctl ops show` → drift; unknown never re-issued; target lock deferred, prerequisites re-read |
| D9 self-dependency | person-operated with a prepared command |
| D10–D13 migration | staged trials ending in a disposable guest create/retire; argue role retires; `cluster:fetch` reads the scan snapshot; Front routing is Front's follow-up |
| D14 | design approved |

## Design in brief

`pj-cagent` with five workflows, `records/` (reports, scans, changes) and `desired/batches/` as its state about the cluster, Nautobot unchanged, and a small tool `cagentctl` that wraps nctl, writes an effects ledger per external action, refuses destroy-class and self-dependency mutations, and requires an attempt and the agreed plan hash for any effect. One backend change: an observation-only `nctl observe` command, which the refresh job then uses. Eight scenario walkthroughs in design §9 say who acts, what proves a result and what stays uncertain; five are checkable from code and tests, the rest need the bounded trials in the plan.

## Unresolved and deferred

- Nothing blocks implementation.
- Deferred to later phases: state-platform replacement, nctl's long-term fate, continuous convergence, per-target lock enforcement, a WFE scheduler, a second execution host, Nautobot-side report records.
- Follow-ups outside this migration: Front's guides naming `wf-cagent.*`; the argue host opening `consult` requests.

## Handoff

[../plan.md](../plan.md): nine steps separating common-system checks, the backend change, the project tool, guides and definitions, trial preparation, seven trials with their evidence, deployment proof, legacy retirement, and the final report. Acceptance evidence is read from records (`wfe run show`, `integrate --status`, `nctl ops show`), not from agent claims.

## Omni notes

- Investigation used three parallel read-only source audits (cagent, backend, destination) whose findings are kept in the ignored notes folder; their consequential claims (the missing guard, the refresh job's `--yes`, the absent observe-only entry point) were re-verified in source by hand.
- No Deus Ex Machina note is due: nothing was done for an in-system agent.
