# Project structure and current-state documentation

Recorded: 2026-10-27 (Asia/Tokyo). Status: worked in tidy p1; dispositions at the end (*Disposition after tidy p1*).

## Why this episode exists

In the aghangout conversation `20261026-233302-g6y7ni`, ponderer said that protoprey had no mouth scene or 3D display (#397, then #399). It searched the published tree for Node3D, GLB and SepRig and treated zero matches as evidence that the interaction had not been integrated.

That conclusion was wrong. At the very revision it named, protoprey `ce8e7b325e`, the README's Start section described entering the wolf's mouth, `docs/cooperation.md` explained embedding, `data/deliveries.json` recorded the adopted wolf delivery, and `data/preds/wolf/predation.json` wired the interaction into the game. The delivered runtime lives in ignored `external/playprey/wolf`, obtained by `tools/deliveries.py install`; it is deliberately absent from the tracked source tree. project_coop p1 had already verified entering, operating and leaving that scene.

This is both an evidence-reading failure and a documentation-maintenance problem. Existing documentation was sufficient to contradict the answer; tidying documents alone does not explain or automatically fix the agent's unsupported inference. However, current state, historical reports and local installation state are not presented consistently, making cross-project discovery unnecessarily fragile.

## Basis and inspection scope

- `devpolicy/styles.md` recommends README/README_DEV for project entry points and `devdocs/episodes/.../report.md` for development reports. Its layout section explicitly says **NOT rule, just a recommendation**. Missing optional files or alternative domain-specific layouts are not automatically violations.
- The same style recommends separating machine-specific facts from portable tracked material, and documents producer/consumer cooperation entries and adopted deliveries.
- WFE-managed run records under `devdocs/runs/<workflow>/<run>/` are legitimate development records; they do not need to be moved into episodes merely to match an example layout.
- Inspected published main through `wfe projects list|tree|read`, not the potentially stale local checkouts. Root inventories covered all 18 registered projects; deeper reads focused on concrete suspicious files and the projects involved in this incident. This is not a complete audit of every nested file, link, credential, binary or Git history.
- Read-only `nctl status` confirmed access to Nautobot. Historical service statements in documents below were inspected as documents, not revalidated as current service health.

## P1 — A root report is a changing per-run output, but the guide treats it as introductory context

**Evidence: protoprey `ce8e7b325e` and `dcfc792937`.**

At the earlier revision, root `report.md` was an old Task 4 report about the three-cycle loop. At the newer inspected main, it has been replaced by a 431-byte `freework/run-003` report about SFX/LOD intake. Thus the problem is not just an old file left behind: the same root pathname now holds a different task's report.

`agent/guide.md` still says to read `GOAL.md`, `README.md`, `VERIFY.md` and `report.md` first. The new root report says “No sound/model/game code exists or was played” and lists runtime playback as not done. Those statements are about that run's scope, but as introductory project context they can be mistaken for claims about the whole game. The same main's README already describes the wolf interaction and the newly added waterfall sound launcher.

**Impact:** a task-local report can become apparent project truth, overwrite another task's visible report, and contradict the current overview. This defeats the purpose of scoped development records.

**Desired correction:** keep reports with their run/episode and preserve their provenance; make the guide point to a maintained current-state entry. Preserve historical content and fix references when moving files. Investigate why runs write/integrate the root report rather than treating this as a one-time move. The inspected freework definition requests a final report but does not itself specify this root pathname, so the generating cause is not established yet.

## P2 — Protoprey's README mixes current instructions and obsolete general claims

**Evidence: protoprey `dcfc792937`, `README.md`, `agent/guide.md`.**

- The title remains “v0.1.0 — first build”; the body combines initial-build notes, old tasks and current integrations without a concise current capabilities/status summary.
- Start correctly describes Forward Plus, installing external deliveries, the animated waterfall, the wolf-mouth shortcut and the new agsfx ambience launcher.
- Known limits still says “Run on macOS only”, although project_coop p1 report5 records Linux tests and a windowed run. Historical verification limits are not clearly scoped to their original build.
- General Controls says Esc returns to the menu, while the mouth interaction has different Esc behavior explained in Start. Readers must reconcile the exceptions themselves.
- README references `direction/REFERENCES.md`, which does not exist in this published revision. Its root tree has no `direction/` directory.

**Impact:** readers cannot reliably separate current behavior, earlier-build limitations, unjudged quality and missing features. Correct information exists but competes with stale statements and a broken reference.

**Desired correction:** maintain a short current overview in README, with capabilities, known limitations, supported/tested environments, artifact setup and links to details. Move or explicitly label historical narratives; repair references. Do not create a second competing current-state document unless ownership and purpose are clear.

## P3 — Other projects also retain individual reports/review instructions at their roots

| Project and revision | Evidence | Concern |
| --- | --- | --- |
| agops `7599556b27` | Root `report.md` describes recovery of one incident; root `review.md` asks the reader to check a “Review release check” page | An incident report and fixture review instructions lack run scope in their root names. `devdocs/workflows/recover.yaml` explicitly asks for report.md **in this run's folder**, so the intended destination already exists. The originating run of these root files needs tracing before relocation. |
| cagent `1feebe2e98` | Root `answer.md`, “Local environment check — animation over window”, is a dated answer from a specific investigation | A single request's answer occupies a general root pathname. Its observation dates are useful and must be retained, but the pathname does not identify its request/run. |
| gimmickimg `feec1955fc` | Root `sheet-review.md` describes the ch-0001 comparison sheet; root `sheet/` holds the presentation | Review material is separate from its trial/run context. Unlike protoprey's report, it may be a deliberately retained deliverable; identify its owner and references before deciding whether to move it. |

**Desired correction:** distinguish maintained project documents, named reusable deliverables and per-run development records. Move only misplaced records, update consumers and links, and preserve evidence. Root placement alone is not sufficient reason to delete a file.

## P4 — Several registered projects lack a reader-facing root entry

**Evidence: published root inventories.** No root README.md or README_DEV.md was present in:

| Project | Inspected revision |
| --- | --- |
| rts-vs-bot | `61fbae3bb5` |
| ppt | `66eac3d8dc` |
| jevlab | `90be9454cf` |
| agops | `7599556b27` |
| front | `e231f8c1ae` |
| lowerpoly | `fe4fa5d744` |
| misc | `0b833d73eb` |

These projects do have `project.yaml` and `devdocs/`; several have useful agent guides. Their absence of a README is therefore an entry-point consistency gap, not proof that the projects are undocumented or unusable.

lowerpoly is a concrete cooperation case: `agent/guide.md` has useful pipeline, input/output and evidence information, but another project must discover that role-specific guide and the first-experiment summary to understand the offer. There is no root README providing the cooperation entry recommended after project_coop p1. agsfx does have a README (`606aeef40a`), but it mainly maps folders and does not link a consumer delivery contract. Its latest work must be inspected before claiming no delivery implementation exists.

**Desired correction:** provide a small, maintained entry appropriate to each project's purpose. For producers, make the offered capability, workflow, input/output contract and current limitations discoverable. Link authoritative details instead of duplicating tool manuals or forcing all projects into an identical directory tree.

## P5 — Generated Godot cache is tracked in gimmickimg

**Evidence: gimmickimg `feec1955fc`.** The published tree contains `godot/.godot/editor/`, `filesystem_cache10`, `project_metadata.cfg`, and multiple generated `.ctex`/`.md5` files under `godot/.godot/imported/`. The bounded listing returned 25 entries with 9 more; several individual texture caches exceed 1 MB. `.gitignore` contains `.local/` and `player/.godot/`, but does not cover the older `godot/.godot/` tree.

**Impact:** generated editor/import state is mixed with source assets and is carried by clones. Ignoring the new player project did not clean up the older experiment project. This is a generated-cache problem, not a blanket objection to keeping intentional media in Git.

**Desired correction:** verify the source assets reproduce the needed imports, remove tracked caches from the active tree and cover all relevant Godot project roots with ignore rules. Do not rewrite history or remove original evidence/assets as an incidental cleanup.

## P6 — Portable guidance still contains local environment facts

**Evidence:** lowerpoly `fe4fa5d744` `agent/guide.md` names the original host/workspace and a host suitable for Metal baking; cagent `1feebe2e98` root `answer.md` contains concrete local service URLs and executable paths; agops `7599556b27` root `report.md` includes a local presentation endpoint.

**Impact:** portable project entry points and task outputs carry facts that can go stale or be mistaken for the executing run's environment. This conflicts with the intended separation between reusable project data and environment-owned configuration.

**Qualification:** cagent intentionally manages cluster records in a private repository. Host-specific evidence can be legitimate there; it must not be blindly erased or moved out of its domain. The problem to resolve is which records are intentional cluster evidence, which are misplaced run outputs, and which are environment instructions incorrectly embedded in portable guides. Exact machine addresses are not repeated in this problem document.

**Desired correction:** put execution configuration in the appropriate local/context mechanism, preserve dated evidence with its owning record, and describe portable requirements separately from a particular host assignment.

## Priorities and limits for the follow-up

1. Correct current-state entry points and run-report placement in protoprey, including the process that keeps recreating root reports. This addresses an observed wrong answer, not a hypothetical naming preference.
2. Trace the other root outputs, repair the confirmed broken reference, and remove reproducible tracked caches.
3. Improve producer discovery, starting with lowerpoly and agsfx, then assess the other missing entry points according to actual use.
4. Audit local-environment facts with project-specific ownership in mind, especially cagent.

Do not classify legitimate domain folders such as `knowledge/`, `lab/`, ponderer's conversation archives, or named deliverables as violations merely because the style's example does not list them. A lagging local workspace is not itself structural corruption; use published revisions for conclusions and make source freshness visible.

The follow-up should demonstrate that a reader can learn current capabilities, limitations and setup from the project entry without reverse-engineering source or mistaking one run's report for the whole project. For the original incident, it should also verify that the agent distinguishes adopted capability, installed bytes and runtime verification. Source-search absence alone cannot establish feature absence.

## Disposition after tidy p1

Published revisions after the episode: protoprey `2ee7d8f28d`, gimmickimg `84af344030`, lowerpoly `8d1bf5278d`.
Evidence: `report1.md`–`report4.md`, `report.md`.

| Finding | Disposition |
| --- | --- |
| P1 protoprey root report | **Corrected.** freework/run-006 moved it with `git mv` to `devdocs/runs/freework/run-003/report.md`; the Task 4 report is restored to `devdocs/reports/task-4-report.md`; the guide points at README and `devdocs/runs/`. **Cause addressed at WFE** (agdevworld `a6564d3`): the brief names the run folder and every documentation step's document as paths. The same cause produced agops' and cagent's root files (below). Follow-on runs 007 and 008 wrote their reports in their folders |
| P2 protoprey README | **Corrected** (run-006): current-state table (built / adopted / installed / verified), verification scoped to macOS as VERIFY records it, Esc in the mouth scene, dangling `direction/REFERENCES.md` removed in README, GOAL, VERIFY. A fresh reader answered the original question correctly (report4) |
| P2 (found in tidy p1) protoprey's agsfx adoption | **Open.** `location_sfx` r3 is adopted from an absolute path in agsfx's run copy on one host (not a share reference) and is not installed in the workspace; a fresh copy cannot obtain it. `deliveries.py status` now warns (run-007/008). The repair is agsfx publishing the pack (`wfe run publish`) and protoprey re-adopting it; the README's "Installed" row is not accurate for this delivery until then |
| P3 agops root `report.md`, `review.md`, `page/`, `presentation/`, `presentation_src/` | **Cause addressed, files still open.** 11 recover runs wrote root `report.md` although the workflow said "in this run's folder"; the brief fix covers new runs. The existing root files were not moved in p1 (agops was outside the trial) |
| P3 cagent root `answer.md` | **Cause addressed, file still open** (consult/run-006, run-007; consult/run-001…005 wrote in-folder). Moving it is cagent's run, keeping its observation dates |
| P3 gimmickimg `sheet-review.md` | **Corrected** (run-016): moved to `devdocs/runs/freework/run-009/review-summary.md`; root `sheet/` removed as a byte-identical copy of run-009's frozen presentation (kept in history) |
| P4 lowerpoly entry | **Corrected** (run-004 and its correction c1): root README with the offer, input/output, how to ask, evidence, limits; host facts removed from it. Remaining: the README no longer says its runs need the toolchain host, and run-004's report describes the first README version |
| P4 agsfx entry | **Open, reassessed.** agsfx does deliver (`agsfx_delivery` v1 in create/run-005); its README does not say so or link a contract |
| P4 rts-vs-bot, ppt, jevlab, agops, front, misc | **Open**, not assessed beyond the root inventory; not declared tidy |
| P5 gimmickimg caches | **Corrected** (run-016): 148 `.godot/` files of three projects untracked (not 30 of one, as first recorded); `.gitignore` covers `.godot/`; a fresh clone re-imports the same files and runs with no errors (report4). History not rewritten |
| P6 lowerpoly guide host facts | **Retained with reason** (run-004): `agent/guide.md` and `wedo/` keep host paths and LAN URLs as working notes and dated evidence; the README no longer repeats them. Whether the guide's original-host instruction should move to local context stays open |
| P6 cagent `answer.md`, agops `report.md` local endpoints | **Open** (with the files above) |
| The original reasoning error | **Addressed at the tool**, separately from the cleanup: `wfe help projects` (agdevworld `46fe968`) says it reads tracked content and that finding nothing is no evidence of absence. The fresh reader cited it. One reader is not proof the error cannot recur |
