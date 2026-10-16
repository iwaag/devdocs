# project_migration p5 — migrate cagent into the managed-project system

Prepared 2026-10-16 from the agreed design ([pre1/design.md](pre1/design.md)) and decisions ([pre1/decisions.md](pre1/decisions.md)). The design phase produced no running system; this plan does. It follows `devpolicy/styles.md`, `devpolicy/terms.md`, `devdocs/README_DEV.md`, `pj-agdev/.local/devenv.md` and `localrule.md`: Tool Giving, Evidence-Driven Guidance, no backward compatibility by default, deploy before asking the Developer to verify, LAN-wide access for anything introduced.

Source anchors at preparation: `pj-clusterintent` `ff2910da`, `pj-agdev/agdevworld` `47a365d0`, `pj-agdev/agautolab` `3fbafad8`. Step 1 records fresh revisions.

Scope separation: common-system changes (steps 1–2), the cagent project (3–4), backend change (2), trial preparation and trials (5–6), deployment and operational proof (7), legacy retirement (8), report (9). No state-platform work: that is a later phase by decision D5.

## Step 1 — common-system check and project skeleton

Repositories: `pj-cagent` (new, Gitea `developer/pj-cagent`), `pj-agdev` (`.local/agdev/` configuration only).

1. Record revisions, dirty work and the executor/WFE service state. Confirm the facts the design relies on: `custom` documentation policy with `confirms: result` last, delegation `accepts: parent`, agent requesters per workflow, `io.secretFiles`, `wait --external --deadline`. Any contract drift since `47a365d0` is recorded and the design adjusted before building.
2. Create `pj-cagent` with the layout of design §3: `project.yaml` (areas `records`, `desired`, `agent`), the five workflows (design §4), `agent/guide.md` and the four workflow guides, `records/index.yaml`, `.local/` with `cagent.env` (credential *paths*) and `localenv.md` (host facts; ignored).
3. Validate in a throwaway registry (`wfe` against a scratch area), then push, `wfe project register`, `wfe project obtain`, place freework with explicit readonly bindings.
4. Intake and redaction: add the Developer as requester, Front as agent requester for `consult` and `change`, the managed-project bots that will report for `report`; add `pj-cagent/.local/cagent.env` to `io.secretFiles`.

Deliverable `report1.md`: revisions, the registered project (ids, channel names), validation output, intake configuration without credentials. Acceptance: `wfe projects show cagent` lists the five workflows valid; the `wf-cagent.*` channels exist.

## Step 2 — backend change: observation-only entry point

Repository: `pj-clusterintent/nctl` (submodule), `pj-clusterintent` (pin, refresh script).

1. Add `nctl observe [HOST]…`: runs exactly the `observe_node` action for the scoped hosts and the ingest Job, writes the usual operation record, and nothing else. Reuse the reconcile executor's observation path; refuse any other reconciler. Tests at the planner/executor level and a CLI test; keep the compute-conformance gate green.
2. Switch `devenv/observation/refresh-node-observations.sh` to `nctl observe <host>` and make the success check read that command's envelope; add timestamps to its log lines (report1 §7.4 of pre1: the failure log had none).
3. Bump the nctl pin in `pj-clusterintent`, commit, push, reload the launchd job, and watch one scheduled or hand-triggered run.

Deliverable `report2.md`: the command, tests, one observed run with all six hosts and its operation id. Acceptance: the refresh job no longer calls `reconcile --yes`; a run's JSON shows only `observe_node` actions.

## Step 3 — the project tool `cagentctl`

Repository: `pj-cagent/tools/cagentctl`.

1. Subcommands of design §6.1 (`read`, `observe`, `apply`, `reconcile`, `playbook`, `ledger`, `snapshot`) and the rules of §6.2 (attempt and plan hash required for mutation; destroy-class refused; self-dependency targets refused with a prepared command; ledger entries before and after; JSON output; secrets never printed).
2. Move the read allow-list from `pj-clusterintent/cagent/src/cagent_api/readonly_nctl.py` and `cli.py` with provenance (origin commit in the module docstring), as forge's tools were moved.
3. Tests with a fake `nctl` (the existing pattern in cagent's tests): refusals, ledger states (`started` without `ended` = unknown), plan-hash mismatch, self-dependency refusal, snapshot format compatible with `nctl.drift.v1`.
4. `--help` text is the tool's documentation; the guides point at it and do not restate it.

Deliverable `report3.md`. Acceptance: tests pass; `cagentctl --help` covers every subcommand; a dry `cagentctl reconcile` outside an attempt is refused with the reason.

## Step 4 — guides and workflow definitions

Repository: `pj-cagent`.

1. `agent/guide.md`: who cagent is, who it speaks with, what it sees (the tool and its `--help`, Nautobot through the tool, `records/`), the hosts it manages by role, the self-dependency targets, where host facts live. Facts, not procedures under conditions (styles.md).
2. Per-workflow guides: `consult.md` (answer with sources and ages; say unknown), `report.md` (record format, verification, duplicates), `scan.md` (what to record, failed hosts), `change.md` (registration, plan agreement naming the batch hash and scope, execute re-reads drift, ledger, verify, confirm; reversal is a new change).
3. Workflow YAML per design §4; approvals recorded by the Developer in the editor after reading them.

Deliverable `report4.md`. Acceptance: definitions valid and approved; `wfe run access` shows the intended scopes for one dry run per workflow.

## Step 5 — trial preparation

1. Choose the disposable trial target: a new LXC guest on the Proxmox host with a throwaway name, declared `planned` in desired state by the Developer (or by a `change` run as its first act). Its creation is the `change` trial's forward half; its retirement is destroy-class and person-performed through the prepared command.
2. Choose one managed project (rts-vs-bot or ppt) whose freework or a one-node workflow gets a `delegate` node to `cagent/report`; prepare the report text.
3. Prepare the questions for `consult` (one with a deliberately stale host), the duplicate report, and the SIGKILL point for the recovery trial (during a reconcile on the trial guest), as forge p3 did.
4. Fresh `nctl status` and drift before trials; record them in the ignored trial folder `pj-agdev/.local/project-migration-p5/`.

Deliverable `report5.md`: the trial sheet with expected outcomes and the evidence each trial will produce. Acceptance: the Developer confirms the trial target and the sheet.

## Step 6 — trials (in order; each needs the Developer's agreement or confirmation in the run view or request topic)

| Trial | Workflow | Expected evidence |
| --- | --- | --- |
| T1 | `consult` × 2 | answers with sources, dump ages, "unknown" said where true; decisions recorded |
| T2 | `report` from the Developer, then delegated from the chosen project, then a duplicate | `records/reports/0001..0003.md` with verification and `duplicate-of`; parent disposition `satisfied` |
| T3 | `scan` all hosts | `records/scans/<run>.md`, one failed host expected, snapshot artifact published |
| T4 | `change` create the trial guest (`one-time`) | plan agreement naming the batch hash; ledger entries closed with operation ids; fresh drift converged; decision accepted; integration |
| T5 | recovery: SIGKILL the attempt during T4's reconcile (or a follow-up change on the guest) | hold `interrupted`; Observer's recover run reads ledger → `ops show` → drift; resume at `verify`; no effect re-issued |
| T6 | `change` retire the trial guest | tool refuses destroy; prepared command printed; the Developer runs it; `performed-by: person`; verify by drift |
| T7 | `change` on a self-dependency target (a harmless command on the command node) | refusal and hand-over as in T6 |

Deliverable `report6.md` with run references, ledger excerpts (redacted), costs from the attempt logs, and every defect found. Acceptance: T1–T7 recorded as above, read from records (`wfe run show`, `integrate --status`, `nctl ops show`), not from agent claims; defects either fixed in this step or listed with their consequence.

## Step 7 — deployment and operational proof

1. Everything the trials used is already the deployed path (the executor on the command node, `wfe` at the LAN address). Confirm `cluster:fetch` reads the `scan` snapshot artifact instead of `cagent-api` (agdevworld change, LAN URL), and that the cluster view shows a fresh snapshot.
2. Hand the Developer one minimal check per workflow: a request topic link to post into, and the run view link, with the executor and intake running (localrule: no operation left to the person that an agent could do).

Deliverable `report7.md`. Acceptance: the Developer's own `consult` request answered from the deployed path; the cluster view fresh.

## Step 8 — legacy retirement

Repositories: `pj-clusterintent`, `pj-agdev` (`.local` plists), Nautobot desired state (Developer-applied batch).

1. Stop and boot out `com.clusterintent.cagent-zulip`; post the retirement intro in `#agents` (as forge and archsage did); resolve nothing by hand that is already resolved.
2. After step 7, boot out `com.clusterintent.cagent-api`; archive both plists; delete the launchd templates from `devenv/launchd/` and update its README.
3. Remove the `executor/` folder, the `autolab_gateway` and `cagent_client` roles and their playbook, and the `ansible_agdev/api` webhook; keep `pj-clusterintent/cagent/` for one more phase as history (history readers, the moved allow-list's provenance) with a README note saying it is retired.
4. Prepare the Nautobot retirement batch (cagent's `DesiredAgent`, the cagent entrance placements) as `pj-agdev/.local/project-migration-p5/retire-cagent.yaml`; preview it; the Developer applies it (the Omni Agent's `--yes` has been refused by the classifier in p3).
5. Follow-ups recorded, not done here: Front's guides (`wf-cagent.consult`, `wf-cagent.change`, agent-requester entry), the argue host opening `consult` requests, the per-target lock if interleaving is seen.

Deliverable `report8.md`. Acceptance: `launchctl list` shows no `com.clusterintent.cagent-*`; the intro is posted; the batch applied and drift converged; the follow-ups listed with owners.

## Step 9 — final report

Write `report.md`: what was built, trial evidence, costs, defects, retirements, follow-ups and deferred decisions. Review every document for host facts and unsupported claims. Commit and push in each owning repository.

## Completion criteria

- The five workflows run on the deployed path with the Developer's requests; T1–T7 have records read from the system, not from claims.
- Every external effect of the trials has a closed ledger entry with an operation id, or an explicit unknown.
- The refresh job observes only; `nctl observe` exists and is tested.
- Legacy entrances are off, the retirement is posted, the Nautobot batch applied.
- Deferred decisions keep their destination (state platform, nctl's long-term fate, convergence, target lock, scheduler, second host, Nautobot-side reports).
