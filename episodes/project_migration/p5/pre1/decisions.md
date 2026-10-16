# project_migration p5/pre1 — decisions

Record of the Step 3 discussion with the Developer, held in the ordinary human conversation on 2026-10-16. Each entry names the question, the evidence, the alternatives, the recommendation, the Developer's response, the outcome and its consequences. Status values: **agreed**, **provisional**, **deferred**, **unresolved**. A recommendation, silence or elapsed time is not agreement.

## Group 1 — purpose and responsibility

### D1. Workflow boundaries

- Question: one request workflow grown by adjustments, or several workflows split by authority?
- Evidence: report1 §2 (one door design left an unscoped shell on explanation doors); options §1.
- Alternatives: four workflows `consult` / `report` / `scan` / `change` plus freework (S); one grown `request` workflow (A).
- Recommendation: S.
- Developer response (2026-10-16): "提案を採用" (adopt the proposal), covering group 1 items 1–4 as presented.
- Outcome: **agreed** — four workflows plus freework with explicit readonly bindings.
- Consequences: tool access and plan agreement differ per workflow; `custom` documentation policy with `confirms: result` last; Front and delegating projects address the workflow by name.

### D2. Where reports live

- Question: project-owned records in Git, or Nautobot-side storage now?
- Evidence: report1 §3 (no ingestion path; purpose only on the desired side; actual-side fields deleted as unused); options §2.
- Alternatives: `records/reports/<id>.md` with provenance in the cagent project, verified by read-only reads (S); a nintent model or restored custom fields (A).
- Recommendation: S; A belongs to the state-platform phase.
- Developer response: adopted with group 1.
- Outcome: **agreed**.
- Consequences: reported / observed / desired are three statement kinds with three homes (options §2 table); purpose of a reported service that becomes managed is copied into the placement `config` by the `change` run; a Nautobot-side report model is **deferred** to the state-platform phase.

### D3. Drift policy and the scheduled refresh

- Question: record and propose only, or converge `maintain` placements unattended?
- Evidence: the LXC incident from an uncancelled turn; the six-hourly refresh runs a bounded `reconcile --yes` that nobody treats as an executor (report1 §7.5); no recorded failure requires repair within hours.
- Alternatives: record + notify + propose a `change` request (S); unattended convergence within a reconciler allow-list (A).
- Recommendation: S; the refresh job becomes observation only.
- Developer response: adopted with group 1.
- Outcome: **agreed**.
- Consequences: a `change` request declares `one-time` or `maintain`; scans never repair; the refresh job needs an observe-only entry point (nctl or a tool wrapper limited to `observe_node`) — an implementation item for the migration plan.

### D4. Restrictions and who may agree a plan

- Question: are plan agreement before effects and destroy-class human-only the whole restriction list; who besides the Developer may agree a `change` plan?
- Evidence: cluster_agent roadmap rule 1 (destroy human-only since 2026-08-09); the LXC incident; both systems enforce nothing below the conversation (report1 §2, options §4).
- Recommendation: those two restrictions only; everything else a guide fact.
- Developer response: adopted with group 1; no other approver named.
- Outcome: restriction list **agreed**; approver = the Developer only, **provisional** (no one else was named; the Omni Agent acts as the Developer's proxy under the existing people.toml rule when the Developer says so per episode).
- Consequences: the `change` workflow's plan talk addresses the Developer; `cagentctl` refuses destroy-class reconcilers outright; intake requesters for `change` are the Developer's id plus named agent requesters; delegation into `change` from other projects still ends in the Developer's plan agreement.

## Group 2 — operating model

Pending: effects ledger and external waits (options §5), concurrency and target lock (§6), self-dependency targets (§7).

## Group 3 — migration scope

Pending: stack disposition and retirements (§8), migration order and trial target (§9), argue participation, `cluster:fetch`, Front routing (§12 Q7–Q10).
