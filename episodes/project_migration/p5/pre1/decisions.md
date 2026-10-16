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

Presented as four points on 2026-10-16. Developer response: "ok、提案を承認" — later clarified ("元の４提案を承認したんです") as approving these four points, not the stack question asked in between.

### D6. Effects ledger and verification

- Question: how are a run and an external operation linked, and what proves a result?
- Evidence: options §5; forge's record-before-act precedent; WFE has no effects concept.
- Alternatives: ledger entries written by `cagentctl` before and after each effect, verification by fresh drift, acceptance as a record not an authorisation (S); every effect as a WFE external wait (A, kept for long operations only).
- Outcome: **agreed**.
- Consequences: `effects/<n>.json` per effect in the run folder; `verify` node reads fresh host-scoped drift; a Git rollback never claims to undo an effect; long operations use `wait --external … --deadline`.

### D7. Concurrency and changed prerequisites

- Question: a per-target lock now, or nctl's lock plus WFE's slot?
- Evidence: no recorded interleaving incident (hypothetical); the LXC incident and "no error is not proof" for prerequisite checks.
- Outcome: **agreed** — target lock index designed but **deferred** until an interleaving is seen; the execute node re-reads drift and asks when the plan's assumptions changed.

### D8. Interruption, partial success, cancellation

- Question: what does recovery read, and when may an effect be retried?
- Evidence: options §6; Observer p3 recovery contract.
- Outcome: **agreed** — an entry without `ended` means unknown; recovery reads the ledger, then `nctl ops show`, then fresh drift; an unknown effect is never re-issued; retry only when the ledger says it never started or drift shows the pre-state; a cancellation after an effect names the open entry and who owes verification.

### D9. Self-dependency targets

**Superseded on 2026-10-16:** the Developer instructed removal of this rule ("不要なルールなので消去してください。"). Dependency hosts now use the ordinary agreed-plan change path. [Amendment and implementation](../self-dependency-removal.md). The entry below records the original decision.

- Question: may a run change the host that carries its own path?
- Evidence: options §7; everything runs on one command node.
- Outcome: **agreed** — targets that host cagent's dependencies (the command node, Nautobot's containers, Zulip, the executor) are person-operated: the run prepares commands and a verification, a person runs them, the ledger records `performed-by: person`; observation stays a launchd job independent of WFE.

## Group 3 — migration scope

### D5. Stack disposition (raised early by the Developer, as a premise of group 2)

- Question: are Nautobot/nintent, nctl, nodeutils/nauto and Ansible carried over, or replaced as part of this migration?
- Evidence: report1 §5 (sole-owner responsibilities of nctl; every nctl read depends on Nautobot; Ansible performs all mutation and the collection itself); the p5 braindump expects a state-platform replacement to be its own phase.
- Alternatives: keep and wrap through a thin project tool, replacement deferred (S); replace selected responsibilities now, which would turn pre1 into a state-platform selection phase (A).
- Recommendation: S. Retire what has no caller: `executor/`, cagent's HTTP doors, CA, ledger, node client, Zulip listener and topic roles, the unused Ansible roles (`autolab_gateway`, `cagent_client`), the power webhook API.
- Developer response (2026-10-16): asked whether the base tools are assumed carried over ("前提として、nautobot、ansible、nctlといった基本ツールは引き継ぎという判断ですか？"); the reasoning above was given. The "承認" that followed was clarified as approving group 2, not this item. Asked again as group 3 item 1; response "採用" (adopt) for all five group 3 items.
- Outcome: **agreed** — keep Nautobot/nintent, nctl, nodeutils/nauto, Ansible for this phase; the listed retirements are in scope; state-platform replacement and nctl's long-term fate are **deferred** to a later phase.
- Consequences: `cagentctl` wraps nctl rather than re-implementing it; the observe-only refresh needs an nctl entry point or a wrapper limited to `observe_node`; retirement steps go into the migration plan with a Developer-applied Nautobot batch (as in p3/p4).

### D10. Migration order and trial target

- Question: staged or big-bang; what is the `change` trial target?
- Evidence: options §9; `change` is the only workflow with effects; no open legacy request exists.
- Outcome: **agreed** — build and register `pj-cagent`; trial `consult` and `report` (from the Developer and from one managed project's delegate node), then `scan`, then `change` as one create-and-retire round trip of a disposable guest on the Proxmox host with plan agreement; then cutover: stop `cagent-zulip`, retirement intro in `#agents`, keep `cagent-api` until `cluster:fetch` is re-pointed, refresh job to observation only, Nautobot retirement batch applied by the Developer, unused Ansible roles retired. Legacy records stay as history; nothing is migrated.

### D11. Argue participation

- Outcome: **agreed** — cagent's argue role retires; an argue that needs cluster facts opens a `consult` request (same direction as archsage).

### D12. `cluster:fetch` (agdevworld's cluster view)

- Evidence: last snapshot 2026-08-10; the script asks cagent's human door to run nctl reads and upload.
- Alternatives: read a snapshot the `scan` run publishes (recommended); read Nautobot directly; drop the view.
- Outcome: **agreed** on the recommendation — the `scan` run publishes the drift/actual/workspaces snapshot as a run artifact the view can read; `cagent-api` retires once the view reads it. If the view turns out unused, dropping it is a one-line follow-up, not a redesign.

### D13. Front routing

- Outcome: **agreed** — left to Front's episode, recorded in the migration plan as a named follow-up with the exact `wf-cagent.*` channel names and the agent-requester entry Front needs.


## Step 4 — design review

### D14. design.md approved

- Presented 2026-10-16 with the three points the design adds beyond D1–D13: the observation-only nctl command (§6.3), the `change` trial as a create-and-retire round trip of a disposable guest with the retire half person-performed, and `cagentctl`'s three refusal rules (§6.2).
- Developer response: "ok".
- Outcome: **agreed** — [design.md](design.md) at this revision is the basis for the implementation plan. No remaining blocking decision; deferred items are listed in design.md §10.
