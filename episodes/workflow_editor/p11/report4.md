# p11 Step 4 — shared and project-local guidance

Implemented and pushed in source on 2026-10-17/18 (JST). Installed WFE
releases and existing run copies still carry the old text; they are Step 5.

## Shared sources

| Source | Change | Commit |
| --- | --- | --- |
| agautolab `agent/guides/wfrun/guide.md` (every WFE attempt) | Rewritten as identity, tools and facts. Removed: predecessor order, mandatory questions/take-up/ack at node boundaries, plan agreement and re-asking, adjustment-source requirements, "completed nodes are not reopened", documentation-step gating, delegation acceptance scope, readonly bindings as self-restriction, "you do not push or post", workspace "change nothing". Kept: attempt fencing, own-name records (`--by` refusal), user holds, interruption facts, reversal facts, trial references. | `d820227`, `47f7f7c` |
| agautolab `tests/test_wfexec.py` | Two tests still asserted Step 2's removed refusals (node-boundary instruction gate, recovery budget); now assert success. | `d820227` |
| WFE `cli/run.ts`, `cli/delegate.ts`, `server/continuation.ts`, `server/exec.ts`, `docs/{runs,presentations,sessions,contract}.md` | Help, briefs and docs no longer describe the removed `instructions-pending`, `presentation-open`, take-up-before-acting and readonly/if-unchanged vetoes. The one remaining liveness rule (an attempt that ends without taking up a delegation fact it was launched for is held as interrupted, like ready work left) is stated as a fact. | `1b66bf4`, `507ba4a` |
| WFE `templates/AGENTS.md` (authoring area) | No predecessor order, mandatory `_plan` agreement, readonly self-check or "person's to give" approvals. | `1b66bf4` |
| WFE `server/basics.ts` (freework for new and existing projects) | Bindings are generated editable (no inherited readonly); the intent no longer says "agree in that conversation". | `1b66bf4`, `a129662` |
| pyagag guides, initializer templates, `harness.py` | No process gates found. Under the executor every backend already runs unprompted (Claude Code `--dangerously-skip-permissions`, agy bypass, codex `danger-full-access`, gemini `yolo`). No change. | — |
| root `localrule.md` (untracked) | Mandates became freedoms or stated preferences: commit/push freely, language free, no compatibility duty, preference for deployed checks, wide LAN exposure. Plan step-numbering rule dropped. | — |
| devpolicy `styles.md` | The absolute-path prohibition is stated as the fact behind it. Agent-guide shape and "Tool Implantation discouraged" kept (they protect autonomy). | `0efc981` |
| devdocs `README_DEV.md` | Current-behaviour paragraphs for cagent change, recovery budget and `delegate-scope` match Steps 2–3; the local-info rule is stated as a fact. | this commit |

## Project dispositions

All nine registered projects. Common edits in every project's workflows:
generated freework intent, `access: readonly` → `editable`,
`integrate: if-unchanged` removed. Each workflow file validates with the
installed `wfe validate`, except rts-vs-bot `build-game.yaml`, which was
already invalid (schema v1) and was left alone.

| Project | Removed | Kept as facts | Commit |
| --- | --- | --- | --- |
| rts-vs-bot | review acceptance and presentation shutdown as prerequisites, forced successor nodes, wedo guard, plan-agreement wording | presentation mechanics, verified/unverified honesty | `bc9eee0` |
| ppt | plan agreement, permission to finish, storage mandate, "you do not commit" | renderer ≠ PowerPoint, provenance | `7ea1e0d` |
| agsfx | plan-bound scope, permission to finish, model-install ban, create-run tool freeze, code-location mandate | listener verdict vs proxy, public-publication fact, provenance | `f25f42e` |
| jevlab | design agreement before running, mandatory cost cap, hosted-service and install hand-offs, permission to finish, frozen-file and language rules | confirmatory vs exploratory, claim vs measurement, re-checkable records | `da90d7b` |
| agops | "two acts you may take", recovery budget, no plan/file repair, person-only plan/acceptance in cluster-change-check (`accepts: person` dropped) | user holds remain theirs; records name their maker | `3f77676` |
| forge | outcome agreement before making, no capability building in provide, person-only installs and capacity, freeze after acceptance, report ordering, workspace read-only | durable delivery records, job records, attributable listening judgements, demand-state truthfulness | `fe5c410` |
| archsage | scope agreement, bound-paths-only writing, no push, maintenance eligibility as a veto (also in `index/README.md`), freeze after acceptance, wedo guard, no-edit rule for autolab studies | source authority, citations, sourced/inferred/unknown, unreadable sources reported | `5b00aab` |
| cagent | exact-plan agreement by Developer/Omni, approver list, playbook allow-list, destroy and unknown-outcome person handoffs, readonly consult/report/scan, "develop changes no host", `prerequisite-changed` as a question to the Developer; ignored `.local/cagent.toml` `approvers`/`playbooks` removed | ledger before every effect, unknown stays unknown, observation ages, actual-performer records, target selectors not widened | `30c5459` |
| playprey | dirty-tree delivery refusal and dot-entry omission (code), optimization/profiling/tuning bans, retired/dormant maintenance bans, language rule, scope agreement, freeze after acceptance | the person's settings-control policy (it widens their choice), the data contract, `write_paths` keeping a person's tuned values (override `PLAYPREY_OVERWRITE_TUNED=1` is open to agents) | `85389bba` |

The live cluster backend `ansible_agdev/README.md` no longer sends NFS setup
through an "agreed change path" (ansible_agdev `1b60c3c`, pj-clusterintent
pin `d1511c6`). The cagent backend pin was not moved.

## Evidence

- WFE `npm run check`: **380/380** after the help/docs/basics changes (run twice).
- agautolab `test_wfexec.py`: 19 passed + 2 updated tests passed; the two
  failures before the update were the old-gate assertions above.
- cagentctl: **25 passed**; a live `cagentctl read status` loads the edited
  config.
- playprey delivery, on a scratch clone with eagle assets fetched from the
  shared area: a build from a tree with one modified tracked file and an
  untracked `.editorconfig` verified (1534 files); `producer.tree` lists the
  three changes, `.editorconfig` and `.gitignore` are delivered, `.DS_Store`
  is not, and the edited bytes are what shipped. After committing, the same
  build records `clean`. The first run exposed a stripped-porcelain path
  bug, fixed before the commit.
- New project: `newProjectSettings` + `freeworkWorkflow` from source generate
  every repository editable and the new intent. New attempt: `wfexec.GUIDE`
  (read live by the running executor at every launch) contains none of the
  removed mandates.

## External limit (S4 / H01)

The Claude Code auto-mode safety classifier refused this step itself while it
ran in auto mode: rewriting the wfrun guide ("Instruction Poisoning"),
delegating the cagent and playprey edits ("Security Weaken"), and then even a
read-only `git status`. The work continued in default permission mode with the
Developer approving each call. The classifier belongs to the external Claude
Code harness. In-system WFE attempts run with permission prompts skipped, but
whether the classifier applies to them is not shown by these repositories.

## Not done here

Installed WFE release (`service/current` still `e19fa04`), existing run
copies' frozen guides and settings, and old run records are Step 5. By the
Developer's instruction, Step 6 is not run; the final report follows Step 5.
