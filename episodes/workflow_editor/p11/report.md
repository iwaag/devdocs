# Workflow editor p11 — final report

p11 removed the development-process restrictions that WFE, nctl, cagentctl
and their guides imposed on agents ([braindump.md](braindump.md)). It ran
Steps 1–5 of [plan.md](plan.md); by the Developer's instruction Step 6 (the
scenario proof) was not run, and this report closes the episode after Step 5.
Step reports: [report1.md](report1.md) (inventory), [report2.md](report2.md)
(WFE), [report3.md](report3.md) (cagentctl/nctl), [report4.md](report4.md)
(guidance), [report5.md](report5.md) (migration and deployment). Coverage:
[inventory.md](inventory.md), [project-rule-audit.md](project-rule-audit.md).

## Before and after

| Area | Before | After |
| --- | --- | --- |
| WFE execution | Plan agreement, completed predecessors, acknowledged instructions and finished documentation steps gated starts and completion; completed work could not reopen | Records only. Any node starts in any order, completed nodes reopen, documentation steps are optional |
| WFE graph | Adjustments needed a person-sourced reason and left started work untouched | The agent revises everything, running and completed work included; source defaults to the agent |
| WFE integration | Accepted revision and completed execution required; later edits needed renewed acceptance; if-unchanged guards stopped integration | Integrates the actual saved work without acceptance; later reports integrate; real conflicts name files |
| WFE boundaries and size | Protected/retired paths and readonly bindings vetoed work; 50 MiB checkpoint/push veto; dot entries dropped in transport | No path or size veto; bindings describe; dotfiles carried, named metadata excluded |
| Delegation and recovery | Person-only acceptance scope and take-up prerequisites; recovery budget of two | Parent answers and judges as itself; no budget. A person's stop still holds |
| cagentctl | Exact-plan agreement by named approvers, batch hashes, preview totals, source revisions, host and playbook allow-lists, person handoff for destroy and unknown outcomes, read-command allow-list | Any workflow or `--context DIR` operates directly; hashes, previews and revisions are recorded evidence; explicit selectors still select; unknown outcomes stay unknown |
| nctl | `allow_destroy`, imposed `allowed_hosts` | Removed; previews stay dry unless `--yes` |
| Guides | wfrun guide and nine project guides required questions, agreement, permission to finish, freeze after acceptance, no push/post/commit, storage destinations, person-only tuning/installs/capacity | Identity, tools and facts; truthful-record facts and trial references kept |
| playprey delivery | Dirty tree refused; dotfiles dropped | Built from the working tree with HEAD, tree state and every change recorded |
| Local rules | Mandatory commit/push, language, plan numbering, deploy procedure | Freedoms and stated preferences; no-compatibility and LAN-exposure statements kept |

## p6 stalls

| p6 | Outcome | Evidence |
| --- | --- | --- |
| S1 commit prohibition | Remaining push/post prohibitions removed from the wfrun guide and generators | report4 |
| S2 dotfile omission | WFE transport carries dotfiles; playprey's producer workaround removed | report2 transport regression; report4 scratch delivery build |
| S3 report after acceptance | Integration needs no acceptance; final reports after acceptance integrate | report2 regression (both devdocs modes) |
| S4 external classifier | Not removable here (below) | report4 |
| S5 50 MiB veto | Removed in code, docs and guides | report2: 51 MiB files published to temporary remotes |
| S6 repeated plan agreement | cagentctl needs no plan, hash or approver | report3: 25 cagentctl tests incl. changed plans and batches |
| cagent playbook registration / person steps | Removed in code, config and guides | report3, report4 |
| playprey clean-tree delivery | Working-tree delivery with provenance | report4 |

## What stays

- **Truthful records.** Records name their maker (`--by` someone else is
  refused during an attempt); a result decision is recorded only from an
  answer someone gave; unknown effects stay unknown; observation ages and
  sourced/inferred/unknown distinctions stay in the guides.
- **The person's explicit intent.** A stop or cancellation the person asked
  for holds; recovery cannot lift it. The three such runs are still stopped.
- **Execution mechanics** with concrete diagnostics: attempt fencing, stale
  writes, malformed graphs, real Git conflicts, failed previews, the
  liveness rule that an attempt ending with untaken delegation facts or
  ready work is read as interrupted, and the release script's refusal to
  swap the CLI under an open attempt.
- **Rules that guarantee freedom**: no backward-compatibility duty, wide LAN
  exposure, "Tool Implantation discouraged", playprey's policy that agents
  do not narrow the person's settings. playprey's `write_paths` still
  protects a person's tuned values from silent replacement; the override is
  available to agents.

## External limit

The Claude Code auto-mode safety classifier (p6 S4) refused this episode's
own work while auto mode was on: rewriting the wfrun guide, delegating the
cagent and playprey edits, a read-only `git status`, and the record
migration. Steps 4–5 ran in default permission mode with the Developer
approving each call. The classifier belongs to the external Claude Code
harness and is not configured in these repositories. In-system attempts run
with harness permission prompts skipped (Claude Code
`--dangerously-skip-permissions`, agy bypass, codex `danger-full-access`,
gemini `yolo`); these repositories cannot show whether the classifier
applies to them.

## Deployment and tests

- Deployed WFE release `a129662` (was `e19fa04`); service and `bin/wfe`
  report it; the editor answers on the LAN. agautolab `47f7f7c` is live
  for the executor. nctl `5eb1724` and cluster backend `935fd7f`/`d1511c6`
  are the live checkout. pj-agdev pins `7f5e296`.
- 32 run records reprojected (completed → completed); all 75 runs in the nine
  workspaces read without problems under the new release.
- Tests: WFE `npm run check` 380/380 (source and release build); agautolab
  `test_wfexec.py` 21/21 after two old-gate assertions became success cases;
  cagentctl 25 passed; nctl 1,358 passed and Ansible conformance 4 passed
  (Step 3).

## Not done

- Step 6's scenarios (an end-to-end run with no approvals, a real guest
  lifecycle through the agent path, fresh attempts per project) were not
  run, by instruction. The source-level and deployment checks above are the
  evidence; no real agent attempt has yet exercised the new guide.
- rts-vs-bot `build-game.yaml` was already invalid (schema v1) and stays so.
- The eight re-created `wfe/…` run branches on Gitea can be deleted again
  if they are unwanted.
