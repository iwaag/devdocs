# tidy p1 — Step 1 report: refreshed evidence and the cause of root reports

Date: 2026-10-27 (JST). Machine-specific observations are in the ignored
`pj-agdev/.local/tidy-p1/notes.md`.

## Revisions read

Published `main` through `wfe projects list` (observed 2026-10-26 19:35 UTC):

| Project | Published | Same as problem.md |
| --- | --- | --- |
| protoprey | `dcfc792937` | yes |
| gimmickimg | `feec1955fc` | yes |
| lowerpoly | `fe4fa5d744` | yes |
| agops | `7599556b27` | yes |
| cagent | `1feebe2e98` | yes |
| agsfx | `606aeef40a` | yes |

Nothing moved between the problem record and this step, so no finding was resolved by later work.
Several local workspaces lag their published main (protoprey, agops, cagent, front, archsage, misc,
ponderer, lowerpoly, agsfx); conclusions below come from the published revisions.

## Findings revised

| Finding | Revision |
| --- | --- |
| P1 protoprey root report | Confirmed, and the cause is now traced (below). |
| P3 other root outputs | **More widespread than recorded.** agops root `report.md` was rewritten by 11 `recover` runs (run-018 … run-037), and agops root also holds `review.md`, `page/`, `presentation/`, `presentation_src/` from `release-check` runs. cagent root `answer.md` comes from `consult/run-006` and `run-007`, while `run-001`…`005` wrote it in their run folders. gimmickimg `sheet-review.md` comes from `freework/run-009` (the fix4 trial). Every one of these workflows asked for the file "in this run's folder" or "in the run folder". |
| P5 gimmickimg cache | **Larger than recorded.** 148 tracked files under three Godot projects' `.godot/` (`godot/` 30, `experiments/waterfall-loop/godot/` 108, `experiments/window-loop/godot/` 10), 15.7 MB. `.gitignore` covers only `player/.godot/`. |
| P4 agsfx | agsfx *does* deliver: `create/run-005` packed `protoprey_locations` revision 3 as `agsfx_delivery` v1 with `delivery.json` and hashes. Its root README still does not say so, so this stays an entry-point gap, not missing capability. |
| P4 lowerpoly | `lod-chain/run-003` completed after problem.md (it is the top commit `fe4fa5d744`). Whether a protoprey-consumable LOD delivery exists is not established and is for lowerpoly's own run to state. |
| P2 protoprey README | Unchanged at `dcfc792937`: "Run on macOS only" (README:24), `direction/REFERENCES.md` referenced from README:26, GOAL.md:65 and VERIFY.md:56 but absent. |

## Why runs write reports at the project root

### protoprey freework/run-003

The run was delegated by ponderer `conduct/run-046` (`accepts: parent`). Its frozen documentation is
`custom` with only `final: true`: the `_final-report` step whose document is
`devdocs/runs/freework/run-003/report.md`.

From the attempt log (`wfe run io run-003 --attempt amvpdmlx9-41124f`):

1. The agent's working directory is the run copy's root. Its first `ls` shows a root `report.md` (the old
   Task 4 report), and the project guide `agent/guide.md` says to read `GOAL.md`, `README.md`,
   `VERIFY.md` **and `report.md`** first.
2. It did all the work inside the talk, then wrote `cat > report.md` in the copy root, replacing the
   Task 4 report, and committed it as `bc7ac5c` "report".
3. `wfe run complete … talk --artifact report.md` met two reminders. `unrecorded-activity` named
   "artifacts outside the run folder are recorded (docs/sfx-lod-intake.md, report.md)". The agent
   repaired the missing work node with `record-done`. It passed `presentation-missing` with `--yes`.
4. The talk's completion completed the run ("Run … is completed; ready: _final-report"). The real
   `_final-report` stayed pending, and the agent told the requester the final report was in `report.md`.

### What the agent was told

`documentationBrief` (agdevworld `workflow_editor/server/exec.ts`, release `9fd7a3f`) names a step's
document path **only for a step that is running, ready or waiting**. At this attempt's start only `talk`
was ready, so the brief said that `_final-report` is a step, but not where its document goes. The run
folder appears in the brief only as the location of `request.md`; nothing says that it is where the
run's own documents belong. The wfrun guide's Documentation section names the files (`report.md`) and
says a step completes when "its document is a nonempty file of the run folder", which is correct but
does not give a path. The same holds for runs whose policy is `manual`: they get no path at all.

### The same failure elsewhere

Every other root output has the same shape. A workflow's prose says "write report.md in this run's folder"
(agops `recover`), "write answer.md in the run folder" (cagent `consult`) or "in this run's folder"
(agops `release-check`). The agent starts in the copy root and resolves the phrase there. About half
the time it gets it right (agops recover: 10 in-folder, 11 at the root; cagent consult: 5 in-folder,
2 at the root). The common guide already says documents belong to the run folder. The problem is that
the agent is never given **the** path, and a root file of the same name is already there to overwrite.

### Cause assessment

| Cause | Evidence | Owning layer |
| --- | --- | --- |
| Path presentation: the brief names no run folder for documents, and no document path for steps that are not yet ready or for `manual` runs | `exec.ts` `documentationBrief`; run-003 brief; agops/cagent recurrence | WFE (system) |
| A root file of the same name invites overwriting, and protoprey's guide presents it as introductory reading | run-003 lines 7–8, 53; `agent/guide.md` | protoprey (project) |
| Completion with a ready documentation step says nothing | "completed; ready: _final-report"; fix4's known limit "freework runs end Done with `_final-report` still pending" | WFE (system) |
| Agent behaviour: work inside the talk, `--yes` past reminders | run-003 lines 55–60 | agent; reminders worked as designed |

Unknown: whether a concrete path alone is enough. The recurrence is about 50 %, so one corrected trial
is weak evidence; Step 4 checks a fresh follow-on run.

The Task 4 report at the root predates the managed project. It was committed `243a95d` (2026-09-21)
by the retired autolab mission system, and Front's onboarding (`de32d81`) wrote the guide that names it.

## Existing mechanisms for the tidy activity

- **Catalog**: `<area>/node-catalog.json`, templates over `do`/`study`/`talk`/`delegate`/`review`
  (`workflow_editor/docs/catalog.md`). It now holds `present`, `cleanup`, `prestudy`, `boast`, `work` and
  `human-review`. New runs freeze it at creation. Writes go through `wfe nodes create|update`.
- **Shared guidance reachable by in-system runs**: `wfe projects read` reads registered projects and
  registered repositories (`repo:<owner>/<name>`), and `execution.guides` reads another project's guide.
  Context blocks (system scope) reach every prompt, and their details are read on demand.
- `devpolicy/styles.md`, the standard this episode relies on, lives only on GitHub (`iwaag/devpolicy`).
  It is **not registered**, so no in-system run can read it.

Choice for Step 2: publish the devpolicy repository itself to Gitea and register it as a shared
repository, so `wfe projects read repo:developer/devpolicy styles.md` works from any run. The tidy
template points at it. There is one source (the same history pushed to both remotes), not a second copy
of the standard in a context block or a guide.

## Baseline cases for the trial

| Project | Case | Useful outcome |
| --- | --- | --- |
| protoprey | current state and report placement | README is the current entry (capabilities, adopted vs installed deliveries, tested environments, limits); the root report returns to its run(s) with provenance; the guide stops pointing at it; the broken `direction/REFERENCES.md` references are repaired or scoped |
| gimmickimg | generated cache and source separation | 148 tracked `.godot/` files leave the active tree, every Godot project root is ignored, and a fresh copy re-imports and loads |
| lowerpoly | consumer-facing entry | a root README with the offer (`lod-chain`), inputs and outputs, limits (worker `studio` only, toolchain on one host), evidence and how to ask, with no host instructions in portable text |

## Next

Step 2: register devpolicy, add the `tidy` catalog template, give the brief a concrete run folder and
document paths, decide whether a completion that leaves a documentation step ready needs a reminder,
and deploy.
