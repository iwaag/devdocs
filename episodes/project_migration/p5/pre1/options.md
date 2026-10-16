# project_migration p5/pre1 — design options for migrating cagent into the managed-project system

Prepared 2026-10-16 from [report1.md](report1.md) (current implementation) and the destination survey of the workflow editor (WFE) and executor at agdevworld `47a365d0` / agautolab `3fbafad8`. Everything below is a proposal for discussion in Step 3; nothing here is agreed.

Reading guide: each decision has the smallest viable proposal (**S**), a credible alternative (**A**), evidence, tradeoffs, a recommendation and the assumptions it rests on. "Evidence" cites recorded failures or source facts; a hypothetical concern is labelled as such.

## 0. The destination in one paragraph

A managed project is a Git repository with `project.yaml`, workflows in `devdocs/workflows/*.yaml`, and local areas. A request (Zulip `request-…` topic in `wf-<project>.<workflow>`, `wfe request`, HTTP, or a delegation from another run) becomes a run that executes in its own working copy on the execution host, as the workflow's declared agent identity, on a `claude_code` profile with `--dangerously-skip-permissions` and an advisory `allowedTools` list. Bindings are path scopes the agent self-checks. A run talks to people through questions (the attempt ends while waiting), holds on stop or interruption, is recovered by Observer's `agops/recover` run, records a result decision by a person, and integrates accepted files into shared branches. WFE knows an *external wait* and tells recovery agents that "side effects may have happened", but has no effects ledger, no approval gate before an effect, no per-target lock, no scheduler and no undo. Precedent for external effects is forge: write a job record before acting, act idempotently, collect once after recovery.

## 1. Responsibility and workflow boundaries

**Question.** How do reporting, scanning, explanation, request registration and execution interact; which need separate runs, shared context or a schedule; what belongs to the requesting project?

**S — four workflows in one project `cagent`, split by authority, plus freework.**

| Workflow | Nodes (type) | Bindings | Effects | Who requests |
| --- | --- | --- | --- | --- |
| `consult` | answer (talk → confirms) | all readonly | none; reads Nautobot through the read-only `cagent` CLI and the project's records | people, Front (agent requester), any project's delegate node |
| `report` | record (do) → verify (do) → confirm (talk) | `records/` editable, rest readonly | none on hosts; writes a report record, verifies by read-only reads (`nctl actual/drift`), optionally proposes a desired-state batch as a file for a later `change` run | people, managed projects' runs (delegate node or agent requester) |
| `scan` | review (do) → confirm (talk) | `records/` editable | observation only (collection on hosts, ingest into Nautobot) through a project tool that cannot run other reconcilers | people; a scheduled opener (§7, §10) |
| `change` | register (do) → plan (talk: agreement) → execute (do) → verify (do) → confirm (talk → confirms result) | `records/` editable, `desired/` editable | desired-state writes, reconcile, playbooks, through a project tool that writes an effects ledger | people; other projects via delegate node; Front via agent requester |
| `freework` | talk | explicit readonly bindings (archsage pattern) | none | people |

Explanation (`consult`) and recording (`report`) are kept apart from execution (`change`) because their authority differs and the current code's one failure class (an explanation that was wrong, a cancelled turn that created a guest) came from mixing read and write in one door. The requesting project keeps: deciding what it needs, posting the report or request, answering the change run's questions, and accepting the result when it is the one that asked (delegation `accepts: parent`).

**A — one `request` workflow with a classifying talk** (like freework grown by adjustments). Fewer definitions; the agent decides mid-run whether it is reporting, explaining or changing.
Tradeoff: tool access and plan agreement cannot differ per kind of work, so every run would carry the change tool; adjustments would have to add nodes for each request; Front routing becomes simpler (one channel). Evidence against: report1 §2, the single-door design is exactly what left an unscoped shell on the explanation doors.

**Recommendation.** S. **Assumptions:** WFE's `custom` documentation policy is used so that `confirms: result` is the last node (the known integration trap); the `scan` and `change` tools are two commands of one project tool with different reconciler allow-lists (§4).

## 2. State meaning and ownership

**Question.** What is reported, observed or desired; where do purpose, owner, provenance and observation time live; how are disagreement, duplicates, absence and failed observation represented?

Facts: desired and actual state live in Nautobot (nintent models, core ledger); purpose exists only on the desired side; there is no installation-report storage; observation time is `last_seen` per Device and dump age per host; drift already distinguishes `converged / converging / drifting / unknown`.

**S — three kinds of statement, each with one home.**

| Kind | Meaning | Home | Provenance fields |
| --- | --- | --- | --- |
| **Reported** | "a run or person says X is installed on Y for purpose Z" | `records/reports/<id>.md` in the cagent project (Git, reviewable, integrated on acceptance) | reporter (run uid or person), source (post or delegation), time, claimed node/service/endpoint, purpose, owner, verification outcome and time |
| **Observed** | what a scan found | Nautobot actual ledger (unchanged), scan run's `records/scans/<run>.md` summary of what changed | `last_seen`, collector version, scan run uid |
| **Desired** | what should hold | Nautobot nintent models (unchanged); the batch documents that changed them kept in `desired/batches/<run>-<n>.yaml` | change run uid, the request, the approver's answer |

Disagreement: a report that a scan contradicts stays a report with `verification: not-found` and is surfaced in `consult` answers as "reported, not observed". Duplicate reports: the record names the earlier one (`supersedes`/`duplicate-of`), decided by the recording agent and stated in its reply. Absence: nothing is written for things nobody reported and no scan saw; `consult` says "unknown to cagent". Failed observation: the scan record lists hosts whose collection failed with the reason; drift already marks their targets `unknown` after 24 h.

Purpose for a *reported* service that becomes *managed* is written into the existing placement `config` JSON (a free field) by the `change` run that creates the placement, so the desired side carries it too; nothing new in Nautobot's schema.

**A — write reports into Nautobot** (restore the deleted owner/purpose fields or add a `ReportedInstallation` model in nintent).
Tradeoff: one state system instead of two, queryable by drift; costs a nintent model, a migration, an image rebuild (known cache trap) and a writer, for a record type the braindump expects to be lossy and advisory. Reports are also about things Nautobot has no object for yet (a service nobody placed).

**Recommendation.** S for this phase; A becomes natural if a later phase replaces the state platform. **Assumption:** the number of reports stays small enough for Markdown records plus an index file; if not, a sqlite index in the project is an implementation detail.

## 3. Desired state versus one-time work

**Question.** Which requests create a lasting maintenance responsibility; what happens when a later scan finds drift: record, notify, propose or repair; is continuous convergence wanted?

Facts: today the only continuous actor is the six-hourly refresh job, which runs a bounded `reconcile --yes` and nobody treats it as an executor (report1 §7.5). Observe-only profiles stop with `manual_intervention_required`. The LXC incident is the recorded cost of unattended mutation.

**S — the request says which, and drift is recorded and offered, never repaired on its own.**
- A `change` request declares `kind: one-time` (run a playbook, create a guest, stop a service once) or `kind: maintain` (a desired-state placement is created or updated so that the existing reconcile keeps it). The plan step must state the kind and what "done" means for each.
- A `scan` run that finds drift records it and posts a notice to the project's request topic (or the delegating project's topic) with a proposed `change` request text. Repair happens only through a `change` run that a person or the owning project opens and agrees to.
- The scheduled refresh is changed to observation only (no `--yes` reconcile); see §7 and §10.

**A — continuous convergence for `maintain` placements:** a scheduled `change` run reconciles drifting managed placements without a new agreement, within an allow-list of reconcilers (service_profile, dnsmasq_config, link_*).
Tradeoff: closer to what Ansible users expect; but every unattended run is a paid model run with skip-permissions, and the one unattended executor that exists today was not recognised as one. No recorded failure says drift must be repaired within hours.

**Recommendation.** S now; A only after `change` runs have produced enough evidence that their effect ledger and verification are trustworthy. **Assumption:** the Developer does not want self-healing yet (braindump: "record and operate", not "keep converged").

## 4. Identity and authority

**Question.** Who can request which work, where is that authority exercised, what do the harness permissions, execution user, inherited environment and credential access actually enforce?

Facts (destination survey): the executor runs as the developer's user on the command node; `claude_code` runs with `--dangerously-skip-permissions`; bindings and allowed tools are advisory; credentials reach a run as ignored `.local/*.env` files copied into the run copy and must be listed in `io.secretFiles` for redaction; the Gitea token is injected only by WFE's own Git operations; Zulip intake accepts configured person ids and per-workflow agent requesters; delegation posts are bound by delivery id. cagent today: no permission engine either (report1 §2).

So **no layer enforces authority at the OS or harness level, in either system.** Authority can be exercised at three real points:

1. **Who may open which workflow**: intake requesters and `intake.agents` per workflow; delegate nodes only from workflows that name `cagent/change`. This is enforced by the intake (bots not listed are ignored).
2. **Plan agreement before effects**: the `change` workflow's plan talk asks the Developer (or the person the Developer named) to agree to the exact batch document and reconcile scope; the execute node is refused by the agent's guide until `agreement` is recorded. The reducer enforces that `_plan` completion needs an agreement; the agent's compliance is a guide fact plus the claim check.
3. **The project tool** (`cagentctl`, a thin Python CLI in the project): the only documented route to `nctl desired apply --yes`, `nctl reconcile --yes` and playbooks from a run. It refuses destroy-class reconcilers outright (human-only, as the roadmap says), requires a run ticket (`WFE_ATTEMPT` plus the run's agreed plan hash) for any mutating call, writes the effects ledger (§5) and records the operation id. The agent can still reach `nctl` directly through `bash`; that is the Tool Giving stance already taken everywhere else in the system, and the IO channel plus the claim check make such a bypass visible.

Separation of secrets: the run needs the Nautobot token, the Ansible SSH key, the vault password and the known_hosts store; all are already on the command node for the developer's user. The project's `.local/cagent.env` names where they are (paths), never values; `io.secretFiles` lists it. User-facing secrets (passwords for people) stay out of cagent, as the braindump says.

**A — a second execution host or user** with only the cagent credentials, so that the harness bypass matters less.
Tradeoff: real isolation; but delegation requires the same host, there is no second host (p1 open item), and the command node is where Nautobot, the Ansible key and known_hosts live. Not for this phase.

**Recommendation.** S (the three points). Keep the restriction list short and tied to evidence: destroy-class human-only (roadmap, LXC incident), plan agreement before effects (same incident), everything else is a guide fact. **Assumption:** the Developer accepts that "cannot" means "not offered and visible if done", as it already does for every other project.

## 5. Run and operation lifecycle

**Question.** How are a request, a WFE run and an external operation linked; how are plan agreement, permission to perform effects, execution success, verification, acceptance and Git integration kept apart?

**S — an effects ledger in the run folder, written by the project tool, verified by fresh observation.**

| Moment | Record | Who writes |
| --- | --- | --- |
| request | `request.md`/`braindump.md` with source | intake |
| registration | `records/changes/<run>.md`: what is asked, target(s), kind, prerequisites observed (drift at registration) | agent (register node) |
| plan agreed | `plan.md` + `_plan` agreement question naming the batch document hash and reconcile scope | agent asks, person answers, reducer records |
| before each effect | `effects/<n>.json`: intent, command, targets, plan hash, started | `cagentctl` |
| after each effect | same file: nctl operation id, exit, artifacts path, ended | `cagentctl` |
| verification | `verify` node: fresh host-scoped drift for every target, compared to the plan's expected end state; written to `records/changes/<run>.md` | agent via read-only reads |
| result confirmation | `confirms: result` question presenting the change record and the effects ledger | agent asks, person decides |
| integration | records and batch documents reach the project's main branch | WFE |

Consequences made explicit: acceptance after an effect does not authorise it (authority was the agreement); a Git rollback removes the record, not the change, and the guide says so ("reversal names external effects, never undoes them" is already WFE's wording); an effect whose outcome is unknown stays `unknown` in the ledger until verification reads the cluster.

**A — model each effect as a WFE external wait** (`wait --external nctl:<operation id> --deadline`) so that Observer sees it.
Tradeoff: fits the contract and gives monitors a reference, but the attempt ends at every wait and nctl operations are seconds to minutes. Use it only for long operations (a guest creation, a baseline playbook).

**Recommendation.** S, with A for operations longer than an attempt should block. **Assumption:** `cagentctl` is small (a few hundred lines) and tested with the nctl fake patterns already in cagent's tests.

## 6. Failure and concurrency

**Question.** Partial success, connection loss, executor restart, duplicate delivery, concurrent work on one target, cancellation, changed prerequisites; when does evidence support retry?

Facts: WFE already handles duplicate delivery by receipt, interruption by holds and Observer recovery, cancellation as final. nctl already has a file lock around reconcile and a per-operation artifact trail. WFE's one slot serializes attempts on the host but a waiting run frees it, so two `change` runs can interleave.

**S.**
- **Partial success / connection loss / restart:** the ledger entry without an `ended` field is the signal. The recovery agent (Observer's recover run, or the resumed attempt) reads the ledger, then `nctl ops show <id>` where an id exists, then fresh drift; it never re-issues an effect whose outcome is unknown. Retry is allowed only when the ledger says the effect never started or when fresh drift shows the pre-state.
- **Concurrent targets:** `cagentctl` refuses a mutating call when another run's ledger holds an open effect on the same target (it lists `.local/wfe/runs/*/tree/devdocs/runs/change/*/effects/` on the host, or an index file the tool maintains in the project workspace's `.local/`). nctl's own lock still protects the reconcile itself.
- **Changed prerequisites:** the execute node re-reads drift for its targets and compares with the registration snapshot; a difference that touches the plan's assumptions is a new question, not a silent proceed.
- **Cancellation:** `run.cancel` after an effect started leaves the ledger entry open; the cancellation message names it and the verify step is owed to a person or a follow-up run.
- **Duplicate requests:** intake already dedupes by post; two different posts asking the same change are two runs that the target lock serializes and the second's register step reports the first.

**A — rely on nctl's lock and WFE's slot only, no target index.**
Tradeoff: nothing to build; interleaving between a waiting run and a running one on the same host is then only visible after the fact. Hypothetical concern today (no incident), so the index can be a second-iteration item.

**Recommendation.** S for the ledger and prerequisite re-check (evidence: the LXC incident and "no error is not proof"); A for the target lock until a real interleaving is seen, with the index designed but optional. 

## 7. Bootstrap and self-management

**Question.** What can cagent do when WFE, its execution host, Nautobot or transport is down; who finishes an operation that disrupts cagent's own dependencies?

Facts: everything runs on one command node: Nautobot, the executor, Zulip's reader, the Ansible key. A `change` affecting that node, Nautobot, Zulip or the executor can cut the run's own path mid-effect. nctl works without WFE; Ansible works without Nautobot only with a hand inventory.

**S.**
- Targets that host cagent's own dependencies (the command node, Nautobot's containers, Zulip, the executor) are declared in the project's `agent/guide.md` as **person-operated**: a `change` run may plan and prepare the exact commands and a verification script, then asks the person to run them; the ledger records `performed-by: person`.
- Observation only (scan) stays a launchd job on the command node that needs neither WFE nor a model; its output is read by the next WFE run.
- Recovery when WFE is down: nctl's operation records and the ledger files in the run copy are plain files on the host; the Omni Agent or the Developer can read them without WFE.

**A — let runs change their own host with a pre-committed recovery script.**
Tradeoff: fewer person steps; the evidence (forge's SIGKILL trial) is for a run losing its process, not for a run removing its own executor. Not for this phase.

**Recommendation.** S. **Assumption:** the set of self-dependency targets is small and named explicitly.

## 8. Stack disposition

| Component | Responsibilities that justify keeping it | Replacement cost / where responsibilities would move | Decision proposed |
| --- | --- | --- | --- |
| Nautobot + nintent | desired and actual ledgers, IPAM, GraphQL reads, Jobs, the only writer endpoint; every nctl read depends on it | a new state store plus migration of 14 models, drift sources, renders and the ingest Job; a product selection or a build | **Keep for this phase; replacement is its own later phase** (braindump agrees) |
| nctl | sole owner of inventory generation, SSH trust, operation history, drift, observation orchestration, planning | each would need a new home; the read-only CLI is also cagent's whole tool surface today | **Keep; wrap, do not re-implement.** `cagentctl` calls it. Reconsider after the state platform decision |
| nodeutils + nauto ingest | the only observation source and its policy | a different collector; the Proxmox helper | keep |
| Ansible (ansible_agdev) | all host mutation and collection; roles for agents and services | rewriting playbooks as something else buys nothing | keep; retire unused roles (`autolab_gateway`, `cagent_client`) and the power webhook |
| executor/ | nothing; no caller | none | **retire** |
| cagent HTTP doors, CA, ledger, node client, Zulip listener, topic roles, argue role | no callers since August; identity model superseded by WFE intake; the argue role's reads can be a `consult` request | the `consult` and `report` workflows; argue participation needs a decision (§11 Q7) | **retire after cutover**; keep `readonly_nctl`/`cli.py` as the project's read tool with provenance |
| observation-refresh launchd job | the only scheduler | WFE has none | **keep, restricted to observation** (needs an observe-only entry point in nctl, or a tool wrapper that runs only the `observe_node` action) |

Deferrable without blocking migration: state-platform replacement; nctl's long-term fate; continuous convergence; a second execution host; a scheduler inside WFE.

## 9. Migration and retirement

**S — staged, with the old paths off before the new intake is on.**
1. Build `pj-cagent` (project, workflows, guides, `cagentctl`, records areas, `.local` facts), register it, validate in a throwaway registry, place freework with readonly bindings.
2. Trial `consult` and `report` (no effects) from the Developer and from one managed project's delegate node; then `scan`; then `change` on a disposable target (a guest on the Proxmox host, created and retired through the workflow with human agreement).
3. Cutover: stop `cagent-zulip` (so no legacy serving), post a retirement intro in `#agents`, leave `cagent-api` until `cluster:fetch` is re-pointed or dropped, then boot it out; change the refresh job to observation only; retire the unused Ansible roles; Nautobot `DesiredAgent` entry for cagent retired by a batch the Developer applies (as in p3/p4).
4. Records: the 106 request evidence dirs, incidents and four resolved topics are history; nothing is migrated (no open request). The change-record convention (`change-<stem>-o<id>`) is replaced by run records.
5. Callers: `cluster:fetch` (read a snapshot the `scan` run publishes, or read Nautobot directly); Front's guides (left to Front's episode in p3 and p4; this time record it as a named follow-up with the exact channel names); autolab's guides mention cagent nowhere any more.
Coexistence without duplicate execution is safe by construction: nothing in the old path executes on its own, and the only scheduled actor is the refresh job, whose change is one line.

**A — big-bang cutover with all four workflows at once.**
Tradeoff: shorter; but `change` is the only workflow with effects and deserves its own trial scope and agreement.

## 10. Capability and gap matrix

| Capability | cagent-specific work | Reusable in the destination | Missing common capability | Defer |
| --- | --- | --- | --- | --- |
| Request intake and identity | guides, intake config, Front as agent requester | Zulip intake, delegation, `wfe request`, HTTP | — | — |
| Explanation from records | `consult` guide, read-only tool | `wfe projects` reads of other projects, run records | — | — |
| Installation reports | `report` workflow, record format, verification | run copy, acceptance, integration | — | Nautobot-side report model |
| Observation | tool wrapper around `observe_node`; scan record | — | **scheduler** (none in WFE); observe-only nctl entry point | WFE scheduler |
| Desired-state change and execution | `cagentctl`, effects ledger, verify node, plan-agreement guide | plan agreement, `confirms: result`, holds, recovery, IO channel with redaction | effects ledger as a system concept; per-target lock | both, until evidence |
| Authority | intake allow-lists, destroy refusal in the tool | agent requesters, delegation `accepts` | OS-level isolation | second host/user |
| Credentials | `.local/cagent.env` with paths, `io.secretFiles` entry | credential pattern from forge | — | — |
| Recovery | ledger reading guide for the recover run | Observer recovery, holds | — | — |
| Retirement | retire batches, launchd bootout, intro | p3/p4 procedure | — | — |

## 11. Representative requests and paper walkthroughs

**R1 — a managed project reports a service.** rts-vs-bot's run has a delegate node `cagent/report` with the text "installed a results viewer on my node, port 8120, purpose: let the Developer see match replays". The child run: `record` writes `records/reports/0007.md` (reporter = the delegating run uid, claimed node from the project's placement, purpose verbatim); `verify` runs `cagent actual <node> --detail` and an endpoint probe through nctl's read path, records `verification: endpoint-reachable, not-placed`; `confirm` presents the record; the parent accepts (`accepts: parent`). Integration moves the record to main. No host was touched. Same report sent twice: the second run finds `0007` by reporter and endpoint, records `duplicate-of: 0007`, and says so.

**R2 — the Developer asks "which node runs Ollama and since when".** `consult` reads Nautobot (`cagent relations`, `cagent actual`), reads `records/reports/` for purposes, and answers with sources and `last_seen`; if the dump for that node is older than a day it says so and offers a `scan`. No effects.

**R3 — one-time: "stop the snake demo service".** `change` run: `register` records target and current drift; `_plan` presents a batch (`run_state: stopped`) and asks the Developer; on agreement `execute` runs `cagentctl desired apply <batch>` then `cagentctl reconcile <host>` (ledger entries 1 and 2); `verify` reads fresh drift and the service probe; `confirm` presents the ledger. Acceptance; integration of the record and batch document.

**R4 — maintain: "keep a node exporter on every active worker".** Same workflow, `kind: maintain`; the batch adds placements; the plan says what later drift means (a missing exporter is reported by `scan`, not repaired). Verification is the converged drift after reconcile.

**R5 — a change on the command node** (restart Nautobot's worker after a stall). `register` recognises a self-dependency target; the plan prepares the command and the check; the person runs it; the ledger says `performed-by: person`; `verify` reads `nctl status`.

**R6 — interruption mid-effect.** The executor is killed after `cagentctl reconcile` wrote ledger entry 2 without `ended`. Hold `interrupted`; Observer opens a recover run; it reads the ledger, `nctl ops show <id>` says the operation converged; it records the outcome in the ledger and resumes the run at `verify`. If `ops show` had no id, fresh drift decides, and an unknown stays unknown in the record.

## 12. Prioritized questions for the Developer

1. Boundaries: four workflows by authority (§1 S), or one grown request workflow (§1 A)?
2. Reports: project-owned records in Git (§2 S) or Nautobot-side storage now (§2 A)?
3. Drift policy: record and propose only (§3 S), or converge `maintain` placements unattended (§3 A)?
4. Authority: are plan agreement before effects and destroy-class human-only the whole restriction list (§4)? Who besides the Developer may agree a `change` plan?
5. Is the effects ledger plus verify node enough for this phase, with the target lock deferred (§5, §6)?
6. Self-dependency targets person-operated (§7)?
7. Stack: keep Nautobot, nctl, nodeutils, Ansible; retire executor, HTTP doors, CA/ledger, Zulip listener, unused roles; refresh job observation-only (§8). Does cagent keep answering in argues, and if so how (a `consult` request opened by the argue host, or no argue participation)?
8. Migration order and trial target (§9): is a disposable guest on the Proxmox host an acceptable `change` trial, and who performs the retire batch?
9. `cluster:fetch`: re-point, replace with a scan-published snapshot, or drop?
10. Front routing: record as a named follow-up for Front's episode again, or include a minimal guide line in this migration?
