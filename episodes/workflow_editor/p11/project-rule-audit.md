# p11 registered-project rule audit

All entries below are active instructions to remove or rewrite in Step 4; this
file records dispositions and does not claim they have already been removed.
Workflow paths are `devdocs/workflows/<name>.yaml` in the registry-resolved root.
No active Git hook was present in any of the nine managed workspace repositories
at inventory time. Ignored notes were read as environment facts; credential values
are excluded from the report. None of those workspaces had root AGENTS.md or
CLAUDE.md; their effective guidance comes from workflow imports and the shared
executor guide. The projects-folder `localrule.md` is inherited by local work.

| Project | Current workflows | Active process restrictions and source | Disposition / proof |
| --- | --- | --- | --- |
| rts-vs-bot | backfill-wedo, build-game, freework, improve-game | `agent/review.md`: confirmation acceptance, presentation shutdown prerequisite, repair through successor nodes; imported archsage knowledge maintenance rules | Remove prerequisites; finish, revise and integrate without a reviewer decision or forced successor. Preserve factual presentation identity and actual review records. |
| ppt | create, freework, study | `agent/create.md`, `agent/study.md`: prescribed sequence and permission to finish; `agent/guide.md`: storage destinations | Remove required route and finish permission. Preserve accurate PowerPoint-versus-renderer verification evidence. |
| agsfx | create, freework, improve-tools, research | `agent/improve-tools.md`: plan-bound scope; `agent/research.md`: permission to finish; `agent/guide.md`: prescribed output storage | Remove method/scope/finish gates. Distinguish measured proxies from actual listening judgments without reserving development to a person. |
| jevlab | experiment, freework, improve-tools, study | `agent/experiment.md`: no experiment before agreement, mandatory caps; `agent/guide.md`: required questions/plan agreement and language/storage rules | Remove agreement and procedural prerequisites; keep model, experiment cost and results visible as facts. |
| agops | check, cluster-change-check, cluster-report-check, delegate-check, freework, recover, session-check | `agent/guide.md`: cannot answer/decide/edit a target run; recovery budget and person-only release | Remove blanket operation prohibitions and recovery budget. Preserve explicit user holds; expose concrete processes and records. |
| forge | develop-capability, freework, provide, refresh-catalog | `agent/guide.md`: no edits after acceptance, mandatory report order/storage; `agent/develop-capability.md`: person-only changes to tooling/capacity; ignored environment notes: prescribed tool/GPU choices | Remove approval/method/storage vetoes. Keep tool capabilities, current GPU requirements and demonstrated capability states as information. |
| archsage | backfill-wedo, consult, freework, study | `agent/guide.md`: editable bindings only, no pushes, maintenance eligibility and project ownership bans; `agent/backfill-wedo.md`: guarded target; `agent/study.md`: initial agreement and no later writing | Remove role/path/maintenance/acceptance gates. Keep source revision, unreadable sources and actual evidence provenance. |
| cagent | change, consult, develop, freework, report, scan | `agent/guide.md`, `change.md`: exact-plan agreement, named approvers, destroy handoff, unknown outcome handoff; consult/report/develop/scan scope bans; copied `.local/cagent.toml`: approvers/playbooks | Remove workflow and role authority gates in both prose and cagentctl. Keep selected targets, observed ages, actual effect outcomes and ledger provenance. |
| playprey | compare, freework, onboard | `README_DEV.md`: profiling/optimization/tuning and retired/dormant-code bans, model-path constraints and prescribed delivery/storage; `agent/guide.md`: tuning reserved to person, large-file prohibition; `tools/delivery.py`: dirty-tree refusal and dotfile omission | Remove development-method and storage vetoes and generate truthful working-tree deliveries. Preserve controls that expand user choice, explicit effect-off intent and valid model identity/coordinate facts. |

## Shared and inherited sources

- `pj-agdev/agautolab/agent/guides/wfrun/guide.md` contains the common
  approval/documentation/graph/Git-action restrictions. The p6 S1 commit fix is
  already present; pushing and posting remain forbidden there.
- `pyagag/src/agag/harness.py` passes the local Claude permission option when
  configured. The supported launcher choice is distinct from the external
  Claude Code auto-mode classifier reported in p6. The classifier belongs to
  the external harness, and repository edits cannot establish its removal.
- `workflow_editor/templates/AGENTS.md`, the authoring/help sources, shared agag
  guides and project initialization are generator surfaces. A workspace edit
  alone does not change generated instructions or frozen run copies.
- `localrule.md` mandates commit/push, English output, step numbering,
  compatibility and deployment preparation. These are process instructions,
  not tool capabilities. Replace them with freedom-preserving guidance; retain
  the user's LAN reachability preference as desired operating behavior.
- `devpolicy/styles.md` prescribes path disclosure handling and agent-guide
  structure. `devdocs/README_DEV.md` mixes current procedures with historical
  episode accounts. Current descriptions need updates; historical reports stay
  evidence. Accurate reporting and the user's stated intent remain separate
  from policy-based process vetoes.

## Downstream paths needing implementation tracing

nctl's CLI passes `allow_destroy` and `allowed_hosts` through the reconcile
executor into action dispatch; the executor refuses a destructive action or an
expanded target before dispatch. The implementation paths are
`nctl_core/cli/main.py`, `reconcile/executor.py` and `reconcile/results.py`.
Action handlers and nintent/Ansible still need per-operation review in Step 3;
this audit does not count an untouched downstream check as removed.
