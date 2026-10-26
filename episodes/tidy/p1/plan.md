# Tidy p1 — repairable project structure and documentation

Status: planned. Implementation and trials have not been performed by this plan.

## Objective

Make project tidying a reusable, visible workflow activity. An agent can inspect a project's structure and current-state documentation, correct concrete problems, preserve useful history and intended behavior, and show that another reader can understand and use the result.

Use the existing node catalog and delegation mechanisms. Ponderer coordinates cross-project work; each owning project makes changes in its own run copy. Do not create a dedicated tenderer project in p1. Reconsider that ownership only if the trials demonstrate a recurring responsibility that existing projects cannot maintain well.

Inputs: `braindump.md`, `problem.md`, `devpolicy/styles.md`, `devpolicy/terms.md`, `devdocs/README_DEV.md`, `pj-agdev/.local/devenv.md`, and `localrule.md` (workspace-relative paths for the latter shared documents).

## Execution policy

- Follow `localrule.md`: maintained files in English, communication with the Developer in Japanese. Implement, verify, commit, push and deploy necessary changes autonomously; do not add approval gates.
- Backward compatibility is not required. Update affected references, callers and documentation together when changing a convention or interface.
- Complete agent-operable setup and checks. Leave the delivered capability usable in the normal system and give the Developer a short route to inspect the result. Any review service needed by this work is reachable from the LAN.
- Consult Nautobot or nctl before relying on or changing local service state, following `pj-clusterintent/nctl/README.md`. Keep machine-specific evidence and setup in ignored notes.
- Preserve unrelated and concurrent work. Inspect published revisions rather than assuming local workspaces are current. Record the revision inspected and the revision containing the correction.
- Standard layouts are recommendations. Fix demonstrated ambiguity, misplaced records, broken references and reproducibility problems; preserve justified project-specific structures.

## Design

### The tidy node

Add a `tidy` template over the existing `do` execution type to the operating area's node catalog. Use the supported catalog tools and revision checks. Do not add a new compiler node type, automatic graph insertion, or a mandatory completion gate.

The template's execution description asks the agent to:

1. Establish the target, revision and scope: changes from this run, a named part of the project, or a project-wide tidy request.
2. Read the project's maintained entry and applicable shared guidance. Compare claims with relevant implementation, adopted deliveries and evidence; distinguish implemented capability, locally installed bytes, runtime verification and human judgement.
3. Inspect current-state clarity, run/episode record placement, references, reproducible caches and environment-information ownership where relevant.
4. Correct supported problems in the declared editable bindings, preserving provenance, authored data and intentional domain structure. Update references when moving files.
5. Verify according to what changed, investigate recurrence where evidenced, and record corrections, checks, justified exceptions, uncertainty and remaining work in the run's own records.

The expectation describes when a distinct tidy step is useful: after meaningful implementation or integration, before handing over the maintained result, or when explicitly asked to inspect an existing project. The workflow designer decides its placement and scope. Do not perform a full repository audit after every small edit or require a particular directory layout merely to satisfy the template.

The node's bindings come from its workflow. A template does not grant access to another project. Ponderer's `do` node is not a way to edit arbitrary project workspaces.

### Ownership and delegation

| Work | Owner and execution |
| --- | --- |
| Shared tidy activity and its discoverability | Existing WFE catalog and shared guidance mechanisms |
| Selecting targets, commissioning work and assessing outcomes | Ponderer through normal conversations and delegate nodes |
| Changes to a project's files, tools and local conventions | That project's ordinary workflow run with a tidy node |
| Problems in WFE/executor paths or prompts demonstrated by a trial | System implementation, with focused validation and deployment |
| Shared findings and episode evidence | This episode; project-specific lessons stay with their owner |

Start with each target's existing `freework` workflow. Add a dedicated project tidy workflow only if repeated use demonstrates a benefit. A batch request becomes separate delegated runs with independently assessable outcomes; failure in one does not erase completed work in another.

For cooperation, the producer tidies its delivery and documentation; the consumer tidies its adoption records, integration and current overview after using the result. A receiving agent returns producer defects to the owning run rather than silently changing another project's published output.

### Shared guidance

The standard must be readable by in-system agents without the Omni Agent's filesystem access. Inspect the existing delivery paths for shared guidance and choose one authoritative, maintained location. Reference it through the supported project/context/guide mechanisms; verify access from a fresh run.

Keep the template focused on the activity. Keep tool usage in help, shared principles in their shared source and project exceptions in project guidance. Avoid copying the full style document into every project or inventing a second independently maintained standard. Version or identify the guidance used in trial evidence.

## Step 1 — Refresh evidence and trace recurrence

1. Re-read the current published state of the projects in `problem.md`; recent work may have changed both symptoms and legitimate needs. Mark resolved findings and revise stale ones.
2. Inspect the existing catalog, guide delivery, project reads, run documentation paths and integration behavior. The relevant contracts include `pj-agdev/agdevworld/workflow_editor/docs/catalog.md` and the shared wfrun guide.
3. Trace protoprey's root `report.md` through its actual run and Git history: the former Task 4 report, its replacement by a later run report, the guide reference and any subsequent changes. Compare the executing prompt, requested report path, tool usage and integration results.
4. Determine whether the root-output cause is ambiguous instructions, path presentation, agent behavior, tooling, or more than one. The common guide already says documentation belongs in the run folder; do not assume another copy of that sentence is the fix.
5. Establish bounded starting cases: protoprey for current state/report placement, gimmickimg for generated cache/source separation, lowerpoly for a consumer-facing entry. Record expected useful outcomes without prescribing every edit.

Deliverable: baseline evidence and a concrete cause assessment, including anything still unknown. Keep machine-specific details in ignored episode notes.

## Step 2 — Provide the activity and correct demonstrated causes

1. Publish the accessible shared guidance chosen above and add the tidy catalog entry.
2. Verify that a newly created run receives the template and can read its supporting guidance. Catalog snapshots are frozen at run creation; do not silently rewrite older snapshots. Use new runs for the main trial, or explicitly adjust an existing run when that is the intended work.
3. Fix the demonstrated source of repeated root reports at its owning layer. Prefer precise context/tool/path improvements where they resolve the ambiguity. Do not automatically move every file named `report.md`, prohibit root files globally, or require tidy to compensate for a known producer bug forever.
4. Update shared agent guidance only where the evidence requires it. Ponderer's role is to commission and assess tidy work, not to become an all-project filesystem editor.
5. If system code changes, run focused checks for the changed behavior and deploy it before the end-to-end trial. A catalog-only change does not require a system release.

Deliverable: a usable tidy activity and any justified recurrence fix, available to normal in-system work.

## Step 3 — Exercise cross-project tidying through ponderer

Commission a normal ponderer-led task covering the three baseline projects. Identify the request as an Omni Agent request on behalf of the Developer; do not fabricate human speech or review decisions. Ponderer delegates to the owning projects' workflows and assesses their returned evidence.

Expected scope by project:

- **Protoprey:** maintain README as the current entry, separate historical/task reports, repair the guide's introductory references and confirmed broken links, scope old verification claims, and explain adopted versus installed external deliveries. Preserve historical reports and the game's authored tuning. Include newer legitimate integrations in the current overview rather than restoring an obsolete p1 snapshot.
- **Gimmickimg:** determine which tracked Godot files are reproducible caches, verify the originals suffice, remove those caches from the active tree and cover the relevant project roots with ignore rules. Preserve intentional media, experiment evidence and published deliveries. Assess root review material by purpose and references rather than moving it on filename alone.
- **Lowerpoly:** provide a concise consumer-facing entry linking current capabilities, workflows, inputs/outputs, constraints, evidence and actual delivery support. Keep domain requirements distinct from original-host instructions. Do not claim an integration exists solely because it is desired or under way.

Record the nodes, inspected revisions, changes, tests and dispositions. If Omni Agent must supply missing facts or manually repair the path, record that intervention, improve the owned mechanism and repeat the affected portion. Do not count the repaired attempt as evidence of independent success.

Deliverable: integrated corrections from three owning-project runs and a ponderer summary showing what was checked and what remains.

## Step 4 — Verify readability, behavior and recurrence

Use fresh run contexts that did not perform the cleanup. Assess them against published project state and artifacts rather than coaching them with the desired answers.

| Check | Evidence required |
| --- | --- |
| Current-state understanding | A reader identifies whether the wolf interaction is implemented, how a fresh copy obtains it and what remains unverified, using the maintained entry and linked sources |
| No false absence inference | Missing GLBs in Git or missing local installation is not reported as absence of the adopted feature |
| Producer discovery | A reader finds lowerpoly's actual offer, limitations and request route without private workspace knowledge |
| References and history | Moved records retain identity/provenance; relevant guide, README and tool references resolve |
| Cache reproducibility | A fresh gimmickimg copy regenerates required imports from intended sources and passes the relevant load/run check |
| Follow-on work | A subsequent small, legitimate development task in a fresh protoprey run records its report in the intended run location and leaves the current overview accurate |
| Concurrent changes | The published correction preserves unrelated work integrated since the initial inspection |
| Batch coordination | Ponderer can commission and assess separate tidy tasks without editing the other projects' working checkouts |

For file moves, check references and consumers. For runtime/tool changes, test affected behavior. Do not add tests that merely assert headings or mirror implementation. Passing structural checks alone is not proof that the prose is current, or that an agent interpreted it correctly.

If the reader still gives the wrong answer despite clear documentation, record and address that evidence-reading failure separately. Do not report document cleanup as proof that the original reasoning error was solved.

Deliverable: trial results with explicit limits. One successful follow-on run is evidence of improvement, not proof that recurrence is impossible.

## Step 5 — Publish, hand off and decide remaining scope

1. Confirm all intended changes are committed, pushed and integrated in their owning repositories; record published revisions, not just local workspace state.
2. Confirm the live catalog exposes tidy and any changed system components are deployed. Verify the normal UI/CLI path to inspect the template and trial runs. Preserve the usability of existing review builds if cleanup affects their setup.
3. Update `problem.md` with the disposition of each finding: corrected with evidence, deliberately retained with reason, or still open. The other audited projects are not implicitly declared tidy because three trials passed.
4. Record whether further batch tidying is warranted for agops, cagent, agsfx and the remaining entry-point gaps. Continue small directly related corrections where justified; do not turn p1 into an unbounded migration of all projects.
5. Write `report.md` here with implementation, root-cause findings, run references, interventions, checks, deployment details at the appropriate local/portable boundary, limitations and recovery instructions. Update relevant shared development documentation.
6. Give the Developer a concise handoff with the tidy entry and the published before/after evidence. Do not require setup or an approval action merely to complete technical delivery.

## Completion criteria

- Tidy is an available, reusable catalog activity using existing execution semantics.
- New in-system runs can access its supporting standard without Omni-only paths.
- Ponderer has coordinated actual owner-executed tidy work across the selected projects and assessed the results.
- Protoprey's maintained entry distinguishes current capability, installation, verification and historical work; misplaced report handling and its demonstrated recurrence cause are addressed or explicitly unresolved.
- Gimmickimg's targeted cache cleanup is verified reproducible; lowerpoly's current consumer entry is discoverable.
- A fresh reader and a follow-on development run provide evidence of improved understanding and record placement.
- The implementation is integrated and usable; findings outside the demonstrated scope retain honest status.

## Out of scope

No dedicated tenderer, new node execution type, scheduled all-project sweep, mandatory tidy gate, automatic rewriting of completed runs, repository-wide history rewrite or uniform directory schema. These may be reconsidered only with evidence of a need. Backward compatibility is not required, but preserving useful history and intended artifacts remains part of a correct tidy operation.
