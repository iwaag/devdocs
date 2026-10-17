# p11 restriction inventory

Status: implementation inventory; entries remain open until their proving scenario passes.
Paths are relative to the projects directory; managed project paths are relative to their registry-resolved root.

| ID | Owner / source | Active consumers and trigger | Existing effect | Disposition and proving scenario |
| --- | --- | --- | --- | --- |
| W01 | agdevworld definition loading in `workflow_editor/server/runs.ts` and executor intake | request/claim on an unapproved definition | Approval status is already informational in the inspected creation path | Verify no indirect approval prerequisite; execute an unapproved workflow through CLI/API/executor. |
| W02 | `shared/compile.ts`, `shared/run.ts`, `server/runs.ts` | generated plan/report completion | requires document, latest-plan answer and take-up | Documents become optional records; complete with no question, then change the plan. |
| W03 | `shared/run.ts`, adjustment file layer | start, adjustment, completion, reopened work | predecessors, instruction acknowledgement, immutable started nodes and authorized source constrain work | Remove process prerequisites; start out of order, reopen, revise running/completed work. |
| W04 | `server/exec.ts`, `server/integrate.ts`, `shared/approval.ts`, browser | integration | accepted revision, completed execution, unchanged target and result required | Integrate actual saved inputs without acceptance; final report and overlapping clean merge succeed. |
| W05 | `server/boundary.ts`, `server/access.ts`, API host-path checks | registration, resolution, requests, repository access | system/retired path vetoes and fixed access scopes | Remove policy boundary; use explicit targets including previously protected paths. |
| W06 | `server/publish.ts`, guides and tests | checkpoint or unpublished commit above 50 MiB | commit/push refused | Remove threshold; checkpoint and publish an intended file above it. |
| W07 | `server/shares.ts`, playprey delivery | dot entry in input/output/download | hidden content omitted | Exclude named transport metadata; deliver `.gitignore` and compare manifests. |
| W08 | `shared/run.ts`, delegation and result decision handlers | parent judges/answers child | person-only acceptance, take-up and disposition prerequisites | Remove authority gates; record actual parent decisions without inventing human agreement. |
| W09 | `server/exec.ts`, executor, release script | interruption/recovery/deploy | recovery budget/person release and open-attempt deployment veto | Expose concrete process state and agent transitions; preserve explicit stop/cancel intent. |
| C01 | managed cagent `context.py`, `effects.py`, `.local/cagent.toml` | any recorded effect | specific workflow, open attempt, approved unchanged plan | Separate context/records from authorization; ordinary context works without approvers. |
| C02 | cagent `effects.py`, `sources.py` | batch, reconcile, sources update, playbook | approved hashes/totals/revisions/hosts/playbook list | Remove authority checks; execute changed inputs and an unlisted playbook, recording actual inputs. |
| C03 | cagent `effects.py`, `ledger.py`, CLI | destroy/privileged operations and unknown effects | person handoff, person-only release | Agent-operated supported tools and recovery; retain unknown outcome as an unknown fact. |
| C04 | cagent `nctl.py` | tool wrapper dispatch | read-command/argument allow-list | General tool dispatch with accurate help and effect records. |
| N01 | nctl CLI, reconcile executor/planner/handlers | execution after preview, destroy or host expansion | confirmation, allow_destroy, imposed allowed_hosts | Separate explicit preview from execution; remove extra authority flags without broadening selected targets. |
| N02 | downstream nintent/Ansible | backend dispatch | policy refusals forwarded by nctl | Trace each caller; remove process vetoes while retaining real argument/capability/transport errors. |
| G01 | agautolab wfrun, pyagag shared guides and initializers | new attempts/projects | human questions, sequencing, push/post/scope prohibitions | Rewrite as identity/context/tools/facts; inspect freshly generated project and attempt. S1 commit fix already exists at `500be25`. |
| G02 | nine registered projects' `agent/`, workflow files and ignored notes | effective instructions per workflow | local review/storage/tuning/platform/delegation/maintenance restrictions | Audit every project; remove process constraints at source and generated copies. |
| P01 | playprey `godot_lab/mouth_start/tools/delivery.py` | build from dirty tree | refusal and silent dotfile workaround | Snapshot actual working bytes with truthful provenance, including dotfiles. |
| L01 | root `localrule.md`, devpolicy and maintained developer docs | inherited developer guidance | compulsory commit/push, language, plan numbering, compatibility/deploy method | Replace procedural rules with autonomy statements; preserve useful factual tool descriptions. |
| H01 | project-owned harness settings and external harness | tool execution | local permission config versus external classifier | Inspect actual launcher/configuration; report external owner and effect separately. No repository change can prove provider-side removal. |

## Registered-project coverage

The live registry contains nine projects: rts-vs-bot, ppt, agsfx, jevlab, agops,
forge, archsage, cagent and playprey. All workspace roots and their ignored local
notes/settings are included in the audit. Shared registered repositories include
study-rts, study-ppt, wedo-ppt, wedo-sfx, aisvgs, growbox, worldtrend,
protoprey-research and papers. Their guide imports and submodule pins are part of
G02. Detailed host paths and raw matches stay in ignored evidence.

The active package is `pj-agdev/agdevworld/workflow_editor`, installed as a
release selected by the operating area's `service/current`. The executor reads
agautolab's guide from its checkout. Managed cagent owns cagentctl and pins a
nested cluster backend; the separate live cluster checkout dispatches operations.
Run copies have their own frozen definitions, copied ignored settings and Git
branches. Editing a workspace alone does not update those copies. The frozen
experiment and retired legacy cagent are historical until a live importer proves
otherwise; archival reports are evidence, not current operating instructions.

## Mechanics to classify individually

Malformed arguments, absent files/tools, Git conflicts, source transport failures,
actual process ownership and atomic record writes are execution facts. They need
accurate diagnostics and usable recovery. Their names do not exempt a behavioral
veto. User stop/cancel instructions remain intent. Historic approvals/decisions
remain facts, and no migration fabricates a person or decision.

Project-specific dispositions and the active workflow list are in [project-rule-audit.md](project-rule-audit.md).

## Verified implementation progress

- W06: both WFE size-based checkpoint/push vetoes removed in agdevworld
  `4497e5b`; two regression cases publish real 51 MiB files to temporary
  remotes. Project/shared instructions and installed releases remain open.
- W07: WFE input/output/HTTP transport carries intended dotfiles, with named
  transport metadata excluded. End-to-end regression passes. Playprey's
  workaround remains open; S2 as a whole is not yet closed.
- Current retained WFE source passes `npm run check`, 380/380 tests. No other
  inventory entry is counted as implemented, migrated or deployed.
- Step 4 (source, not yet deployed): G01 wfrun guide rewritten (agautolab
  `d820227`, `47f7f7c`); WFE help/docs/template/basics generators
  (agdevworld `1b66bf4`, `507ba4a`, `a129662`); G02 all nine projects'
  guides and workflows (see report4.md); P01 playprey delivery from the
  working tree with provenance (`85389bba`), closing S2 on the producer
  side; L01 root `localrule.md`, devpolicy `0efc981`, devdocs README_DEV;
  H01 recorded as an external limit. W06/W07 guide equivalents removed.
  Deployment and run-copy migration remain Step 5.
