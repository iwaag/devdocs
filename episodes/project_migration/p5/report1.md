# project_migration p5 — Step 1: contracts verified, workspace prepared

Written 2026-10-16. This step read the installed workflow editor (WFE), the executor, nctl and the old cagent, and created the `pj-cagent` skeleton in an isolated migration clone. Nothing operational changed: no reconcile, refresh, deployment, service or credential change was made. Host facts, operation ids and credential locations are in the ignored notes `pj-agdev/.local/project-migration-p5/step1.md`.

## 1. Revision anchors and state

| Item | State |
| --- | --- |
| pj-clusterintent | `ff2910d`, clean, equal to origin. Submodules clean: nctl `0d18eb6`, ansible_agdev `366d7ed`, nauto `38f758c`, nintent `2ff73f8`, nodeutils `d68b963`. |
| pj-agdev | `1b58a37`, clean. agdevworld `47a365d` (equal to the installed WFE release), agautolab `3fbafad`. |
| devdocs | `c3e84ad`, clean. |
| pj-cagent | Did not exist. Skeleton committed in the migration clone (`724dd67`), not pushed. |
| Repository instructions | No `CLAUDE.md`/`AGENTS.md` applies beyond the workspace policies (`devpolicy/`, `localrule.md`, `devdocs/README_DEV.md`) and pj-clusterintent's `README_DEV.md` test matrix. nctl changes need the nctl suite and, when the compute contract is touched, the compute-conformance gate. |
| Cluster, read through nctl | `status` ok; Nautobot reachable; dumps about 3 h old for the six managed hosts, one fixture row stale since July. Drift 45 converged, 1 drifting (Observer's known `service_missing`). pre1's inventory still holds; no new caller or open cagent request appeared. |

## 2. WFE capabilities against what cagent needs

| Need | Supported operation | Gap / decision |
| --- | --- | --- |
| Project registration | `wfe project register <owner>/<name>`, `wfe project obtain <id>`; the Gitea module pushes | The module creates repositories `private: false` only. pj-cagent is created private through the module's raw API call (§5). |
| Isolated run copies | Every executed run works in `.local/wfe/runs/<wf>/<run>/tree`, on its own branch, for its whole life | Tracked tools (`tools/cagentctl`) are in the copy; their `.venv` is not, so the tool runs with `uv run --project tools/cagentctl`. |
| Custom documentation, generated `_plan` | `documentation: {policy: custom, plan, reports, final}`; `_plan` records the plan.md hash it asked about and the agreeing question | Used for `change` only. |
| Final result confirmation | One `confirms: result` talk; `decide --question` binds the deliverables by hash | Put last in every workflow. |
| Delegation | `type: delegate`, `accepts: person|parent`; the child is an ordinary request run | A delegated `change` still asks the Developer (§3). |
| Requesters | `intake.requesters` (people), `intake.agents` (an agent per named workflow) | Developer is already a requester; agent entries are added in Step 4. |
| Execution profile | `execution: {agent, profile: sonnet, guides}` | As designed. |
| External waits | `wfe run wait --external <ref> --deadline` | Nothing wakes a run when the external result arrives. cagentctl therefore waits in the foreground (§4). |
| Observer recovery | `interrupted`/`unknown` holds go to `agops/recover`, which may `wfe run recover` | The recovery guide already leaves a run held when an external effect may have happened. cagent's own agent reconciles the ledger on resume. |
| Snapshots / artifacts | A run-folder file is published to the run branch at checkpoints and to `main` on integration; `wfe projects read|run` and `GET /api/projects/…` read it with provenance | Step 7 reads the snapshot through project access. |
| Integration | Accepted results integrate into the project's `main` | Records and batch documents reach `main` only this way. |
| IO redaction | Patterns, plus exact values from `io.secretFiles` parsed as JSON or `KEY=VALUE` | **Gap G4**: a raw-value secret file (a bare token, a vault password) yields no value, so its value is not redacted by value. cagent's credentials are of that kind. |

## 3. The tool-to-run contract

`WFE_ATTEMPT` plus a hash the caller supplies proves nothing. `cagentctl` therefore derives the agreement from the run's own record, as follows.

1. **The attempt.** A mutating or observing subcommand takes `--run <workflow>/<run>` and needs `WFE_ATTEMPT`. It reads `wfe run show <run> --json` from the run copy and requires `record.control.attempt.id == WFE_ATTEMPT`. Otherwise it refuses and dispatches nothing.
2. **The workflow.** Mutations (`apply`, `reconcile`, `playbook`) are allowed only in a `change` run. `observe` and `snapshot` are allowed in `scan` and `change` runs.
3. **The agreed plan.** The governing plan question is the latest question that recorded a `document` (the `_plan` question, or a later `--about-plan` one). It must satisfy all of these:
   - its document is the run's `plan.md`, and its `sha256` equals the current file's;
   - for the `_plan` question, `_plan` is `completed` with `agreement` naming it;
   - for a later revision, the question is taken up;
   - the taken-up answer's `from` is a configured approver.

   Approvers are configured in the project workspace's ignored `.local/cagent.toml`: the Developer, and the Omni Agent's recording name only while a delegation for the episode is configured there.
4. **The plan content.** The agreed `plan.md` carries one fenced `cagent-plan` YAML block: `kind` (`one-time` or `maintain`), `batches` (path and sha256), `reconcile` (host, allowed hosts), `playbooks` and `person` (handoffs). The hashes and scopes the tool enforces are read from these agreed bytes, never from arguments.
5. **Durable effects.** Each effect is `devdocs/runs/change/<run>/effects/<nnnn>.json`, created exclusively, then written to a temporary file, fsynced and renamed. The entry exists before dispatch. An entry without `ended` means unknown.
6. **Available to recovery.** The copy lives as long as the run, and checkpoints publish the run folder to the run branch. A later attempt, a recovery run (`wfe runs index`, the copy) and project access all read the same entries. Record ids are `<run>/e<n>`, so distinct runs cannot collide.

## 4. Implementation decisions closed here

| Question | Decision |
| --- | --- |
| Observation authority, `scan` vs `change` | Observation needs no plan agreement (D3: it collects and ingests and never repairs). It still needs an open attempt of a `scan` or `change` run, and it is written to the same ledger as kind `observation`, so recovery sees it. |
| Effects entry vs observation record | One ledger, two kinds (`effect`, `observation`), one entry format. |
| Snapshot schema and selection | `cagent.snapshot.v1`: the three nctl envelopes (`drift`, `actual --detail`, `workspaces`), each with its `generated_at` and errors; the observation operation id and per-host outcomes; `complete` (all three reads succeeded and every scoped host was observed). The consumer reads the newest **integrated** scan run that has a snapshot, and shows `complete`, its time and failed hosts. A newer scan without a snapshot is named, not skipped silently. |
| A long nctl operation when the attempt ends | WFE kills the attempt's process group. cagentctl starts nctl in its own session, with an operation id it chose and recorded first, and waits for it in the foreground. If the attempt dies, nctl keeps running, and the open entry names the id that `nctl ops show` finds. |
| Actions nctl infers | Host-scoped `reconcile --yes` re-plans every round, and `service_profile` targets every managed placement of a service. Its action set can therefore exceed the dry plan (§6). |

No substantive design change is needed. The two nctl additions below implement design §6.2's refusal rules where the design assumed nctl already exposed enough.

## 5. Ignored configuration, secrets, publication

- **Reaching run copies.** The regular files directly in the project workspace's `.local/` (up to 1 MiB) are copied into each run copy's `.local/`. `cagent.toml` (the approvers, the path of the pj-clusterintent checkout, self-dependency hosts) and `localenv.md` reach every run this way. Credentials stay where nctl and Ansible already read them.
- **Redaction.** Listing a file of credential *paths* in `io.secretFiles` redacts nothing that matters. The referenced raw-value files must be listed, and the redactor must read a raw-value file as one secret (G4). That is verified in Step 3/4 with synthetic secrets.
- **Publication boundary.**
  - `devdocs`, `pj-clusterintent` and `pj-agdev` publish to GitHub, so they get no host names, addresses or inventory. The refresh script's hard-coded host list moves out (Step 2).
  - pj-cagent's records name hosts by nature (D2). The repository is created **private** on the LAN Gitea and has no other remote.
  - Raw operation evidence stays in nctl's events directory and in ignored notes.

## 6. Bounded common-system changes required

| Id | Repository | Change | Why it cannot be avoided |
| --- | --- | --- | --- |
| G1 | nctl | `nctl observe [HOST]…`: only `observe_node` collection and ingest, plus an explicit host set | Design §6.3. Audit: of 1616 forced-refresh plans, **14 also planned and executed** `service_profile` or `dnsmasq_config` actions (latest 2026-10-03). The scheduled job is a repairer. |
| G2 | nctl | `--operation-id` on `reconcile` and `observe` | The id must be in the ledger before the effect starts. Today it exists only in the output. |
| G3 | nctl | `reconcile --allowed-hosts h1,…`: an action whose contacted or targeted hosts fall outside the set is refused before dispatch, in every round | Without it, cagentctl cannot keep a re-planned round or a multi-host `service_profile` off a self-dependency host. Destroy needs no change: nctl refuses it unless `--allow-destroy` is given, and cagentctl never passes it. |
| G4 | agdevworld (WFE) | The IO redactor reads a single-value file as one secret | cagent's credentials (Nautobot token file, vault password) are raw-value files. The existing Gitea token file has the same gap today. |
| G5 | — (pj-cagent) | Effects ledger, detached long operations | WFE has no effects concept (D6). This stays project code. |

## 7. Repository ownership

| Repository | Owns in p5 |
| --- | --- |
| `pj-cagent` (Gitea, private) | project, guides, workflows, `cagentctl`, records, batches |
| `pj-clusterintent/nctl` | G1–G3 and their tests |
| `pj-clusterintent` | submodule pin, refresh script, later retirements (Step 8) |
| `pj-agdev/agdevworld` | G4; `cluster:fetch` (Step 7) |
| `pj-agdev` | pointers; ignored WFE/intake configuration |
| `devdocs` | reports |

## Acceptance

- Implementation seams are identified (§3, §4).
- The required common-system changes are G1–G4, each bounded and justified by a recorded fact (§6).
- No operational migration has happened. The skeleton is unpushed, and no service, schedule, desired state or WFE configuration changed.
