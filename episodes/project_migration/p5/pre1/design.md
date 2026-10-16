# project_migration p5 — cagent as a managed project: agreed design

Written 2026-10-16 from [decisions.md](decisions.md) (D1–D13) over [report1.md](report1.md) and [options.md](options.md). This is a design for the implementation phase; it claims no operational result. Where a choice belongs to a later phase it says so.

**Current amendment (2026-10-16):** the Developer removed the self-dependency-host refusal and person-only handoff in D9. Dependency hosts use the ordinary agreed-plan change path; historical D9 references below describe the original approved design. See [self-dependency-removal.md](../self-dependency-removal.md) and the updated [implementation plan](../plan.md).

## 1. Responsibilities

cagent is the managed project `cagent` (repository `pj-cagent`, agent identity `cagent-agent`). It is responsible for:

1. **Recording reports** of tools and services that projects or people installed on cluster hosts, with the purpose they state, and verifying them against what the cluster shows.
2. **Observing** the cluster through the existing collection path and recording what a scan changed.
3. **Explaining** the cluster from its records and from Nautobot, saying what is reported, observed, desired, stale or unknown.
4. **Registering and carrying out changes** to hosts and VMs that lie outside the requesting project's own node, through the existing desired-state and reconcile machinery, under the Developer's plan agreement.

It is not responsible for: preventing changes made by other routes; keeping the cluster converged on its own; secrets for people; the state platform's future (a later phase); anything on hosts it cannot reach.

## 2. What stays, what changes, what is later

| Component | Disposition |
| --- | --- |
| Nautobot + nintent (desired and actual ledgers, batch writer, Jobs) | stays unchanged; used through nctl |
| nctl (reads, drift, render, reconcile, operation records, SSH trust) | stays; gains one entry point: an observation-only command (§6.3); wrapped by `cagentctl` |
| nodeutils + nauto ingest, Ansible playbooks and roles | stay; the roles `autolab_gateway` and `cagent_client` and the power webhook API retire |
| `pj-clusterintent/cagent` (HTTP doors, CA, ledger, Zulip listener, topic and argue roles, incidents) | retires after cutover (§8); `readonly_nctl.py`'s allow-list and `cli.py` move into `pj-cagent` with provenance as the read tool |
| `executor/` | retires |
| `observation-refresh` launchd job | stays, changed to the observation-only command |
| agdevworld `cluster:fetch` | re-pointed to the snapshot the `scan` run publishes; `cagent-api` retires after that |
| Front's routing to cagent | Front's episode (named follow-up, §8) |
| State-platform replacement, nctl's long-term fate, continuous convergence, per-target lock, a WFE scheduler, a second execution host | later phases |

## 3. Project layout

```text
pj-cagent/
  project.yaml            ag.project.v2; areas: records, desired, agent
  agent/
    guide.md              project-wide facts (hosts cagent manages, self-dependency targets, how to refer to .local facts)
    consult.md report.md scan.md change.md   per-workflow guides (named by execution.guides)
  tools/
    cagentctl/            the project tool (Python, uv project): read, observe, apply, reconcile, ledger
  records/
    reports/<id>.md       reported installations (D2)
    scans/<run>.md        what a scan changed, per run
    changes/<run>.md      registration, prerequisites, verification per change run
    index.yaml            small index the tool maintains (ids, targets, states)
  desired/
    batches/<run>-<n>.yaml  the batch documents a change run applied
  devdocs/
    workflows/{consult,report,scan,change,freework}.yaml
    runs/…                 WFE run records
  .local/                 ignored: cagent.env (paths to credentials, never values), localenv.md (host facts)
```

Host facts and credentials never enter committed files; `agent/guide.md` says to read `.local/localenv.md` and names what it holds.

## 4. Workflows

All five use `documentation: {policy: custom, …}` with `confirms: result` as the last node, so no generated report follows the confirmation (the known integration trap). `execution.agent: cagent-agent`, `execution.profile: sonnet`, `session: fresh`.

| Workflow | Nodes | Bindings | Intake |
| --- | --- | --- | --- |
| `consult` | `answer` (talk, `confirms: result`) | everything readonly | `zulip: requests`; agent requesters: Front; delegation from any project |
| `report` | `record` (do) → `verify` (do) → `confirm` (talk, confirms) | `records` editable; rest readonly | requests; agent requesters: configured project bots; delegation (`accepts: parent` allowed) |
| `scan` | `review` (do) → `confirm` (talk, confirms) | `records` editable | requests; opened by people or by the refresh job's successor (§6.3) |
| `change` | `register` (do) → `_plan` (generated plan talk, documentation `plan: true`) → `execute` (do) → `verify` (do) → `confirm` (talk, confirms) | `records`, `desired` editable | requests from the Developer; delegation from projects; agent requester Front |
| `freework` | `talk` | explicit readonly bindings | requests |

Node descriptions carry the contract of each step; the guides carry the facts. The `_plan` step of `change` is the authority point (D4): the agent asks the Developer to agree to the exact batch document hash and reconcile scope; `execute` is described as "refused by the agent until `agreement` is recorded on `_plan`".

## 5. Records and their meaning (D2)

| Statement | Home | Fields |
| --- | --- | --- |
| reported | `records/reports/<id>.md` | id, reporter (run uid or person), source (post id or delegation id), time, node, service, endpoint, purpose, owner, verification {outcome: reachable / placed / not-found / not-checked, time, by run}, relation {supersedes / duplicate-of} |
| observed | Nautobot actual ledger; `records/scans/<run>.md` | hosts collected, hosts failed with reason, ingest outcomes, drift summary before/after, dump ages |
| desired | Nautobot nintent; `desired/batches/<run>-<n>.yaml` | the applied document; its hash is what the plan agreement named |
| change | `records/changes/<run>.md` | request, targets, kind (one-time / maintain), prerequisites (drift at registration), plan hash, effects (copied from the ledger at confirm), verification |

Disagreement, duplicates, absence and failed observation are represented as options §2 states: a report a scan contradicts keeps `verification: not-found`; a duplicate names the earlier id; nothing is written for the unknown; failed hosts are listed with reasons and drift marks their targets `unknown` after 24 h.

## 6. The project tool `cagentctl`

A small Python CLI in the project, with `--help` as its documentation (Tool Giving). The agent may use `nctl` directly; the tool is the documented route and the only one that writes the ledger.

### 6.1 Subcommands

| Command | What it runs | Writes |
| --- | --- | --- |
| `read status|drift|relations|actual|ops …` | the read-only allow-list moved from `readonly_nctl.py` | nothing |
| `observe <host>…` | the observation-only nctl command (§6.3) | nothing in the project; the scan run records the result |
| `apply <batch.yaml>` | `nctl desired apply -f … --yes` | ledger entry; copies the batch to `desired/batches/` |
| `reconcile <host> [--allow-destroy refused]` | `nctl reconcile <host> --yes --max-rounds N --json` | ledger entry with the operation id |
| `playbook <name> --limit <hosts>` | an allow-listed playbook from `ansible_agdev` | ledger entry |
| `ledger list|show|close <n> --outcome …` | the run's `effects/` | the entry |
| `snapshot` | the three reads `cluster:fetch` used (drift, actual --detail, workspaces) into one `nctl.drift.v1`-compatible file | the file as a run artifact |

### 6.2 Rules the tool enforces

- A mutating subcommand needs `WFE_ATTEMPT` in the environment and `--plan <hash>`; it refuses otherwise and says why (a person uses nctl directly, not this tool).
- `--allow-destroy` and the destroy-class reconcilers are refused outright with the roadmap's wording: human-only (D4).
- Before running, the entry `effects/<n>.json` is written with `{intent, command, targets, plan, started}`; after, `{operation_id, exit, artifacts, ended, outcome}`. A crash between leaves `ended` absent, which means **unknown** (D8).
- Targets listed in `agent/guide.md` as self-dependency targets are refused for mutation; the tool prints the prepared command for a person instead and writes a ledger entry with `performed-by: person` and no `ended` until the agent records the person's confirmation (D9).
- Output is JSON; secrets never appear (the tool reads credential *paths* from `.local/cagent.env`, which is listed in `io.secretFiles`).

### 6.3 Backend change: observation-only entry point

nctl has no observation-only command and `reconcile` has no action filter (checked 2026-10-16). Add `nctl observe [HOST]…` (or `reconcile --observe-only`) that runs exactly the `observe_node` action for the scoped hosts and the ingest Job, with the same operation record, and nothing else. The refresh script then calls it instead of `reconcile --refresh-observation --yes` (D3). This is the only nctl change the migration needs.

## 7. Lifecycle of a change (D6–D8)

```text
request ──► register ──► _plan (agreement by the Developer) ──► execute ──► verify ──► confirm ──► decision ──► integration
                │                                                  │            │
     records/changes/<run>.md                       effects/<n>.json      fresh drift per target
```

- **Registration** records the request, targets, kind and the drift at that moment.
- **Agreement** names the batch document hash and the reconcile scope. Re-presenting a revised batch asks again.
- **Execute** re-reads drift for the targets first; a difference touching the plan's assumptions becomes a question (D7). Each effect goes through `cagentctl`.
- **Verify** reads fresh host-scoped drift and the service probes and writes the outcome into the change record; an effect without `ended` is written as unknown, never as success.
- **Confirm** presents the change record and the ledger; the Developer's answer becomes the decision (`decide --question`).
- **Integration** moves the records and batch documents to main. It never claims to undo an effect; the guide says a reversal is a new `change`.
- **Long effects** (guest creation, baseline playbooks) use `wfe run wait --external nctl:<operation id> --deadline`; Observer sees the wait.
- **Interruption**: hold → Observer's recover run reads the ledger, then `nctl ops show`, then fresh drift (D8). Resume continues at `verify` when every entry has an outcome, else at `execute` for entries that never started.
- **Cancellation** after an effect started: the message names the open entry and that verification is owed to a person or a follow-up run.
- **Concurrency**: nctl's file lock plus the host's single slot; a target index is designed (`records/index.yaml` `open_effects`) but not enforced until an interleaving is seen (D7).

## 8. Migration and retirement (D10–D13)

1. Build `pj-cagent` (layout §3, tool §6, workflows §4, guides), validate in a throwaway registry, push, register, obtain, place freework with readonly bindings, add `.local/cagent.env` to `io.secretFiles`, add intake entries (Developer, Front as agent requester for `consult`/`change`, project bots for `report`).
2. Backend change §6.3 in `pj-clusterintent/nctl`; refresh script switched to it.
3. Trials, in order, each with plan or confirmation by the Developer: `consult` (two questions, one with stale data), `report` (from the Developer and from one managed project's delegate node; a duplicate), `scan` (all hosts; one asleep host expected to fail), `change` (create and retire a disposable guest on the Proxmox host; the retire half is destroy-class and therefore **person-performed through the prepared command**, which is itself the test of §6.2's refusal and hand-over).
4. Cutover: stop and boot out `cagent-zulip`; post the retirement intro in `#agents`; re-point `cluster:fetch` to the `scan` snapshot artifact; then boot out `cagent-api`; archive the plists; retire the two Ansible roles and the webhook; the Developer applies the Nautobot batch retiring cagent's `DesiredAgent` and placements.
5. Follow-ups recorded in the plan: Front's guides (channel names `wf-cagent.consult`, `wf-cagent.change`, agent-requester entry), the argue host opening `consult` requests, and the per-target lock if interleaving is observed.

Legacy records (106 request evidence directories, 7 incidents, 4 resolved topics) stay where they are as history.

## 9. Scenario walkthroughs

For each: who acts, what is read and written, what authority suffices, what proves the result, what stays uncertain after a failure, and whether it is checkable from code and tests or needs a bounded trial.

| # | Case | Who acts / reads / writes | Authority | Proof | Uncertain after failure | Check |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | Reported installation with purpose, delivered twice, later contradicted by a scan | project run → delegate `cagent/report`; child reads Nautobot and `records/`; writes `reports/0007.md`; second delivery: intake dedupes by post id, a second distinct post yields `duplicate-of: 0007`; a later `scan` run writes `verification: not-found` into 0007 | the delegating run's standing (intake agent / delegation); parent accepts | the record with verification fields; integration on main | if the verify step dies, the report stays `not-checked`; a scan fixes it | code/tests: intake dedupe (WFE tests), record format (tool tests); trial: one delegated report |
| 2 | Unreported service, unreachable node, stale record used to answer | person asks `consult`; agent reads drift (service unknown), dump age (host failed last scan), `records/`; answers "observed on <date>, not reported, host unreachable since <scan>" and offers `scan` | none (readonly) | the answer cites sources and ages; `confirms: result` | nothing executes | code: readonly bindings; trial: one stale question |
| 3 | One-time operation and a maintain request | `change` × 2: one-time (stop a service) applies `run_state`, reconcile, verify converged; maintain (exporter on workers) adds placements; later drift is reported by `scan`, never repaired | Developer's plan agreement (D4) | fresh drift converged for the targets; ledger entries closed | an entry without `ended` is unknown until `ops show`/drift says otherwise | trial (disposable target) |
| 4 | Request from another managed project, with questions and result return | parent run's delegate node → `cagent/change`; questions from the child reach the parent's channel (p10 relay); the Developer still agrees the plan; result returned as the child's confirmation, parent disposes | delegation for the request; Developer for the plan | child's change record; parent's disposition | same as 3 | code: delegation tests; trial: one delegated change |
| 5 | Concurrent changes on one target; a precondition changed before execution | second `change` registers, sees the first's open entry in the index (advisory), says so and waits on a question; execute re-reads drift and asks when assumptions changed | Developer | both change records name each other; verification per run | interleaving between a waiting run and a running one is visible only in the records (lock deferred, D7) | code: prerequisite diff in the tool; trial only if interleaving appears |
| 6 | External action completed, response lost; partial success; restart; cancellation | ledger entry open; hold `interrupted`; recover run reads ledger → `nctl ops show` → drift; records the outcome; resume at `verify`; cancellation names the open entry | Observer's recover run (budget 2), Developer for resume | the closed ledger entry plus fresh drift | an operation with no id and ambiguous drift stays unknown and is said so | code: ledger states; trial: forge-style SIGKILL during a reconcile on the disposable target |
| 7 | Work affecting cagent/WFE or their dependencies | `change` targets the command node or Nautobot: tool refuses mutation, prints the prepared command and check; the person runs it; the agent records `performed-by: person` from the person's answer | the person | the person's confirmation plus a read (`nctl status`) | if the person's action broke the path, recovery is by hand from files on the host | code: refusal list; trial: a harmless command on the command node |
| 8 | Cutover with an open legacy request; later discovery of a missing caller | no open legacy request exists today; if one appears before cutover, it is restated as a `change` request by the Developer; a caller found later (a node still carrying the client, Front) gets a refusal from a dead door and is re-pointed, with nothing executed twice because the legacy path never executed by itself | Developer | retirement intro; dead doors answer nothing | a node-side client that nobody finds | checked by the cutover checklist and the Front follow-up |

## 10. Remaining decisions

- None blocking implementation. Deferred by decision: state platform, nctl's long-term fate, continuous convergence, per-target lock enforcement, WFE scheduler, second execution host, Nautobot-side report model.
- To confirm at review: the trial target (a disposable guest on the Proxmox host, with the retire half person-performed) and that Front's follow-up stays outside this migration.
