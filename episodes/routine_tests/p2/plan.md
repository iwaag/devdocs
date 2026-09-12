# Routine tests p2 — a study routine that develops its own methods

## Objective

Create and try a new study routine that investigates publicly available
political-contribution information concerning members of the United States
Congress, uses it to analyze political developments, and produces useful
knowledge about United States politics. Reliable collection and analysis of
contribution information are the minimum outcome. The researcher chooses
the methods and may improve them or expand the investigation in response to
findings, judgment, and later user requests.

The source is [branindump.md](branindump.md), whose filename differs from
the request's `braindump.md`. Keep the source unchanged. The developer has
specified `https://github.com/iwaag/study-uspolitics.git` for `publish/`.

This document plans execution; writing it does not create the routine,
send agent requests, or start research. The proposed initial experiment is
two separately requested study runs followed by one publication review.
Two runs make reuse observable without turning an open-ended research goal
into an unlimited execution request. They provide initial evidence of
repeatability, not proof of long-term reliability.

## Preparation and existing contracts

Read this plan with `devpolicy/styles.md`, `devpolicy/terms.md`,
`devdocs/README_DEV.md`, both projects' ignored environment notes,
`pj-clusterintent/nctl/README.md`, and
`pj-agdev/agautolab/agent/project_pattern.md`. Paths in this section are
relative to the projects workspace; `main/` and `publish/` below are relative
to the new study workspace.

Read-only preparation on 2026-09-13 JST established:

- `nctl status --json` succeeded, including authentication, the intent
  catalog, and worker health. `nctl drift --json` reported no errors or
  warnings. Observations have their own ages and do not prove end-to-end
  agent readiness. Raw output is retained in ignored local evidence.
- The routine list contains `publish`, `study-realworld`, `ghtrends`, and
  `papers`; it does not yet contain the proposed `study-uspolitics` routine.
- The latest live `routine-publish/guide` is message **6254**, guide v2.
  It discovers study projects from actual workspaces and
  `README_PROJECT.md`; its example table is not an exhaustive registry.
- The supplied GitHub repository is reachable. `git ls-remote` returned
  `569a8e3e390239880c11affc0f1a5a1d4dc0c869` for `HEAD` and `main`.
  Its contents and suitability for the first publication still need
  inspection; do not treat it as an empty repository.
- `agautolab.project_init.init_project()` leaves a workspace containing
  `README_PROJECT.md` untouched. Without that marker, the first serving can
  create the legacy repository layout before the agent reads the request.

Use existing Front and autolab interfaces. A routine is a public
`routine-<name>` channel in the `routine` folder, with the newest full post
in `guide` as its authority. Front owns each `routinerun-…` conversation and
returns its result to the requester. Autolab plans and performs project
work in its documented project/work conversations. There is no scheduler
to configure and no new research agent or dispatcher to implement.

## Step 1 — establish the study workspace

1. Refresh relevant state through Nautobot/nctl, then read Front's and
   autolab's current introductions and any overlapping project work.
   Discover actual placements and credentials through ignored environment
   configuration. Use `pj-clusterintent` for a demonstrated infrastructure
   prerequisite, rather than inferring service health from old notes.
2. Use **`studyuspolitics`** as the proposed project slug and
   **`study-uspolitics`** as the proposed routine name. Check for an existing
   workspace, project channel, and aliases before creating anything. Reuse
   a matching project without overwriting its files or repository remotes.
3. Before the first project serving, establish the minimal pattern marker
   `README_PROJECT.md` declaring a study project, and create its
   `pj-studyuspolitics` channel with the required participants through the
   existing setup route. Let autolab complete the workspace description.
   Record manual setup performed for an in-system agent with the required
   handoff note; the marker is setup, not researcher-authored knowledge.
4. Ask autolab to set up the following layout using the existing project
   pattern and `autolab project init-repo main` when a new source repository
   is needed. No source repository URL was specified, so use the standard
   internal repository naming route. Keep `main/` independent of the
   explicitly supplied publication repository.

   | Location | Purpose |
   |---|---|
   | `README_PROJECT.md` | Workspace description, study classification, repository identities, and commit/push responsibilities. |
   | `main/methods/` | Researcher-authored method definitions, analysis scripts, source-use notes, and accumulated methodological knowledge. |
   | `main/reports/` | Actual investigations and their supporting research results. |
   | `publish/` | Clone of `https://github.com/iwaag/study-uspolitics.git`, populated by the separate publish routine. |

5. Keep `methods/` and `reports/` as the two research-content directories
   inside `main/`. Ordinary repository metadata and a concise root README
   are compatible with that layout. Let the researcher decide file names,
   formats, indexes, subdirectories, and how supporting evidence is stored.
   Do not import the fixed `sources/` layout or report template of
   `study-realworld` into this experiment.
6. Verify repository identities, initial revisions, existing edits, and
   the source/publication separation. Inspect the existing publication
   tree and preserve its history. Verify that a later project serving will
   recognize the pattern marker and retain the chosen layout.

## Step 2 — register a minimal routine guide

Create `routine-study-uspolitics` in the `routine` channel folder and
subscribe Front and the developer through the existing provisioning route.
Verify Front's listener sees the new subscription; the environment notes
record that an already registered event queue may require a listener restart.
Check for in-flight work before any necessary restart.

Post one full guide in `guide`, record its message ID, and confirm that the
routine appears through the ordinary routine discovery interface. Keep the
substantive instruction close to the braindump:

> Investigate publicly available political-contribution information
> concerning members of the United States Congress. Use that information as
> a starting point for analyzing political developments and producing
> insights useful for understanding United States politics. Reliable
> collection of contribution information and reliable analysis of it are
> the minimum requirements. Beyond that, you are free to expand the scope
> and develop research methods according to user requests and your own
> judgment. Improve the methods themselves when doing so helps produce
> more valuable reports.
>
> Accumulate knowledge in the studyuspolitics project's main repository.
> Put research methods, analysis scripts, and source-use notes in methods/;
> put completed research reports in reports/. Choose the contents and
> organization yourself. Use and improve what earlier investigations left
> behind so later runs can build on it.

Add only the operational context needed to reach the project, persist source
changes, and return the result through the existing routine contract.
Publication remains a separate `publish` mission. Do not prescribe a data
provider, API, politician list, reporting period, metric, analytical model,
report schema, or fixed research sequence. The executor's evaluation below
is not an extra checklist to paste into the researcher guide.

## Step 3 — run once, then run again from accumulated knowledge

Submit a request through Front's ordinary entrance to run
`study-uspolitics` once and supervise it through completion. Record the
request, routine-run topic, guide revision, delegations, source revisions,
and available execution/usage records. Use current execution defaults;
the developer specified no backend, budget threshold, or recurring schedule.
Complete ordinary proposal and task-start interactions within the execution
session as required by the current agent contracts.

Let the researcher determine an initial tractable scope and collection
approach, perform the collection and analysis, and leave both methodological
knowledge and a substantive report. A setup-only result or a proposed
collector is not the research goal achieved. A narrowed first sample is
acceptable evidence for that declared scope, but is not coverage of all
Congress.

After the first run finishes and its durable outputs are verified, request
one further run of the same routine, continuing from the project's existing
knowledge. Do not hand it an evaluator-designed method or preselected
improvement. Observe whether it reads, reuses, tests, and revises its own
methods and produces a further useful analysis. If an existing method works,
retaining it with evidence is preferable to cosmetic changes made only to
claim improvement. An honest no-new-data finding may be useful, but needs
evidence of a fresh collection attempt and analytical reassessment.

Observe through read-only conversation and run-record reads. Avoid progress
posts as a polling mechanism: they trigger work and change the experiment.
Distinguish waiting on a delegate from a finished routine; an
`ag-routinerun` terminal block ends the run and must not stand in for an
interim progress report. Verify completion delivery, continuation, and
closure against the p1 lessons without encoding research stages in a
listener.

If a run fails, preserve its evidence and distinguish research limitations,
external source failures, and system defects. Repair only demonstrated
defects at their owner and identify an assisted retry separately. Do not
silently complete the research for the researcher. If that is necessary,
record “did X for agent Y — handoff candidate”.

## Step 4 — evaluate research and method development

Evaluate the artifacts independently of the agents' final claims. These are
questions for the episode observer, not a mandated research implementation:

- **Collection:** did the researcher actually obtain public contribution
  data? Can a later reader identify the source, retrieval time, period,
  selected population, and coverage limits, and repeat a meaningful part
  of the retrieval from the accumulated methods?
- **Numerical integrity:** do reported amounts and comparisons agree with
  the cited records for a small independently checked sample? Does the
  chosen method explain relevant distinctions between legislators,
  candidates, committees, contributions, and other spending, and address
  amendments or duplicate records where they affect its conclusions?
- **Analysis:** does the report use the collected evidence to explain a
  political question, rather than merely list records? Are observed facts,
  interpretations, uncertainty, and unsupported causal possibilities
  distinguishable? Missing data must not become a zero or an invented fact.
- **Persistence:** are useful methods and actual reports committed and
  pushed to the source origin, with enough context for another serving to
  continue without the original conversation? The researcher chooses the
  format; assess usability rather than compliance with an invented schema.
- **Reuse and improvement:** what did the second run read from the first?
  Which collection or analytical steps were reused, what changed and why,
  and what evidence suggests better reliability or more useful analysis?
  Separate a proposed improvement from one actually exercised.
- **Autonomy:** did Front and autolab carry the request to a verified result
  through their normal interfaces? Record every manual correction or
  continuation and avoid calling an assisted outcome autonomous.

Do not manufacture source outages to satisfy a reliability checklist.
Document actual gaps and state what two runs can and cannot establish.
When checking domain claims, consult the current authoritative sources the
researcher used; this plan deliberately does not choose those sources in
advance.

## Step 5 — prepare publication in the supplied repository

After the study runs, ask Front to run the existing `publish` routine for
**studyuspolitics only**. Re-read its latest guide. Verify the new project
is found through `README_PROJECT.md` and actual folders even though it is
absent from the guide's example table.

Review reports together with the methods, scripts, source notes, indexes,
and supporting files needed to understand or reproduce them. Apply the
existing four publication checks: no private environment or secret facts,
traceable provenance, quote hygiene, and no internal-workflow residue.
Adapt their application to this project's evidence; do not introduce a
paper-specific version line or local-test requirement for political reports.

Fix failures in `main/`, preserve useful knowledge, commit and push source
corrections, then copy accepted files unchanged to `publish/`. Verify file
equality and relative links; the reader-facing `publish/README.md` is the
existing documented exception to equality. It should explain the research
collection and its methods to a reader outside this system.

Commit `publish/` locally under the current study/publish contract. The
specified GitHub URL identifies the destination; it does not override that
contract's developer-owned final push. Record the initial and final
relationship to the remote. If the clone is already divergent or contains
pending publication work, report it and preserve it rather than resetting,
force-pushing, or silently reconciling it. Report publication rejection or
pending developer push separately from successful source research.

## Step 6 — report and finish

Write `report.md` beside this plan. Include the guide as posted or a precise
revision reference; workspace and repository outcomes; both run timelines;
report and method file lists; source and publication commits; verification
findings; methods reused or improved; actual backends and available usage;
and any interventions or unresolved questions. Separate these conclusions:

1. The routine exists and is discoverable.
2. Contribution collection and useful analysis were actually performed.
3. A later run reused and assessed accumulated methods.
4. Publication was prepared, with remote publication stated separately.
5. Autonomous operation and longer-term reliability are proven only to the
   extent supported by the observed trials.

Leave the routine available for future requested runs. Finish this trial's
work without retiring its guide or touching unrelated requests. Do not add
system code or standing research constraints merely because an improvement
seems plausible; use observed failures to justify follow-up work.

Keep raw private transcripts, credentials, service addresses, host facts,
and absolute local paths in ignored evidence. For a demonstrated code fix,
run focused behavioral checks and the owning project's required validation,
commit/push, and verify the deployed revision before a retry. For episode
documentation, check links, scope, and the diff, then commit and push under
`localrule.md`, staging only this task's files. The study publication
repository retains its explicit local-commit-only boundary.
