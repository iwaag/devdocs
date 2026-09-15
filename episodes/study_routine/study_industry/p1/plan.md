# study-industry p1 — Execution plan

Implement [braindump.md](braindump.md): add a reusable `study-industry`
routine that investigates corporate activity in a requested industry and
accumulates knowledge useful for understanding its direction. Start with
the video game industry. Use
`https://github.com/iwaag/study-industry.git` for `publish/`.

This document plans implementation and its live trial. Creating the routine,
starting research, and preparing publication are execution steps for the
subsequent implementation task.

## 1. Carry forward the relevant lessons

- [study-uspolitics](../../../routine_tests/p2/report.md) demonstrated that
  researchers can choose, reuse, and extend their own methods when given a
  goal and `methods/` and `reports/`. Keep that freedom. Its
  [later trial](../../../routine_tests/p2/ex2/report.md) also found wrong
  source-code definitions and prose contradicting computed results: checking
  arithmetic alone is insufficient, and a supervisor's summary is not an
  artifact review.
- [study-realworld](../../study_realworld/p1/report.md) demonstrated reusable
  acquisition recipes, but a check finding no update left no durable record.
  Retain dates and outcomes of checks, including unchanged, unavailable, and
  incomplete results, in an agent-chosen location in the knowledge tree.
  Publication review must preserve research questions, earlier-report links,
  provenance, and useful retrieval findings when rewriting operational prose.
- The current routine contract is documented in
  [README_DEV.md](../../../../README_DEV.md): a routine has its own channel,
  a fixed `guide` topic, and Front-owned `routinerun-` conversations. Use that
  contract; the scheduled dispatcher and Plane records in older reports are
  historical. The routing repairs following the political study are already
  implemented; refresh the current contracts during setup.
- [better_zulip_call p1](../../../better_zulip_call/p1/report.md) moved board
  reads and listener recovery to persisted mirrors. Use existing read
  surfaces and their freshness information to follow execution. Targeted
  conversation reads remain legitimate; repeated full-realm polling would
  recreate the problem the episode solved.

## 2. Establish the project and repository boundaries

Use an Autolab study project, provisionally `studyindustry`. Check for an
existing project before creating one, read the current agent introductions,
and use the ordinary project initialization workflow.

```text
<project workspace>/
  README_PROJECT.md       # workspace-local repository and operating context
  main/                   # working knowledge repository
    methods/              # reusable explanations, acquisition and analysis code
    reports/              # industry findings and supporting publishable evidence
  publish/                # clone of the specified GitHub repository
```

Create `main/` through `autolab project init-repo` when it does not exist.
Clone the supplied URL into `publish/`, preserving its existing history,
license, and content. Inspect its default branch and current contents at
setup rather than assuming it is empty. Record both origins and publication
handling in `README_PROJECT.md`.

Only `methods/` and `reports/` are prescribed knowledge divisions. Delegate
filenames, subdirectories, navigation, report format, research units, and
choice of tools to the agents. Ordinary repository files such as a README,
license, and `.gitignore` may accompany them. Adapt the study pattern locally;
its paper-specific test levels and per-investigation repositories are not
requirements for this study.

The common project-pattern documentation already places private facts,
credentials, and large artifacts in ignored `.local/`; the workrun guide
also requires checking `.gitignore` before committing. Verify these rules
are available in the actual study workspace and that its ignore rules work.
Use `.local/` or storage outside repositories for downloads, database files,
volumes, caches, and execution logs that do not belong in Git. Keep reusable
code and explanations tracked. Any missing study-specific clarification
belongs in this project's context rather than a new global policy.

## 3. Define the routine with one additional capability

Create `#routine-study-industry` in the existing `routine` channel folder.
Ensure the developer, Front, and the existing observation path can see it,
and verify Front's subscription reaches its active listener. Account for
the documented event-queue subscription refresh behavior when necessary.
Publish the complete guide as a new post in `guide`; later versions are
new full posts. Verify that the ordinary routine board discovers it.

The guide should express the following intent in natural language:

> Investigate corporate activity in the requested industry and accumulate
> knowledge that helps the requester understand its current dynamics and
> possible future direction. Cover leading companies and their market
> positions, emerging companies attracting attention and investment, where
> industry participants invest, and other information you judge useful.
> Read existing methods and reports before choosing the next investigation.
> Store reusable methods in `methods/` and findings in `reports/`. Choose
> the research scope, sources, methods, tools, and presentation yourself;
> retain or improve existing methods according to the evidence.
>
> If local statistical or database tools or services would help achieve the
> purpose, notify the requester before using them. You may use suitable
> existing services or start your own service in Docker. Check existing
> service availability with cagent, directly or through Front, before
> introducing another database instance. Preserve reusable or general
> explanations and scripts in `methods/`, sufficient to rebuild the analysis
> from online sources and tracked instructions without the original local
> database. Keep machine-specific configuration and large local artifacts
> outside version control.

This is an option to exercise judgment, not a database requirement or a
requirement to invent a novel method. Preserve the existing lesson about
human dependencies: when an account, API key, paid access, or acceptance of
terms needs the requester, say what is needed and continue independent work
where possible. Do not silently conceal a material loss of coverage.

### Service coordination

Prefer a direct researcher-to-cagent inquiry when the research agent can
reach that entrance; report the result to Front as the supervisor. Front
can relay the inquiry when that is the available route. Discover the current
entrance from the agent's introduction rather than embedding a bot instance
or endpoint in the guide.

Before actual use, send the requester a short notification stating the
analytical purpose and intended resource. When Front is the immediate
requester, it should relay this to the originating developer conversation.
The braindump asks for advance notification; it does not add a second
permission round for ordinary use within the authorized study.

Have cagent establish suitable existing service instances from Nautobot or
`nctl`, including whether research use is appropriate. Existing infrastructure
databases are not automatically research sandboxes. Reuse a suitable service
with a separate study database/schema where appropriate. If a new Docker
service is useful, record its study owner, purpose, reuse location, and
cleanup or retention decision in ignored operational notes so later runs
can find it rather than create duplicates. Retire study-owned temporary
resources when no longer needed; do not remove shared service data.

Planning-time `nctl status --json` and `nctl drift --json` succeeded on
2026-09-15, with no reported drift errors. This establishes access to the
state authority, not suitability of a particular database for research.
Refresh the relevant facts during execution and keep local addresses,
inventory details, and credentials in ignored notes.

## 4. Run the video game industry trial

Ask Front to run `study-industry` once for the video game industry, producing
a report useful for understanding trends and assessing possible future
developments. Front should supervise actual Autolab work through completion.
Use a fresh workplan conversation for the run, start the generated workruns,
and let their callbacks return to the owning routine run. An opened plan or
an acknowledgement is not evidence that research executed.

Let the researcher choose a useful bounded scope and state its geography,
segments, time coverage, and research cutoff. The broad industry request
should result in a useful synthesis with explicit gaps; it is not a claim
that every company or investment can be covered in one run. Additional
investigation can follow from the report's own open questions.

The following are **review questions for the trial**, not a prescribed
source list, calculation sequence, or report template:

- Does the report explain the industry's value chain and identify major
  companies using meaningful, comparable measures? For any market-share
  figure, can a reader identify the market, denominator, period, geography,
  and source? Are estimates and unavailable shares labeled honestly?
- Does it distinguish emerging-company attention from demonstrated business
  traction, and identify the evidence behind funding and growth claims?
- Does it explain investment flows in both directions: who finances the
  industry, and which companies, technologies, markets, or adjacent sectors
  industry participants fund or acquire? Are announced and completed deals,
  funding, acquisition value, and operating expenditure distinguished?
- Are material claims traceable to sources with dates and coverage? Are
  source definitions, missing data, conflicting estimates, and changes of
  currency or accounting scope handled coherently?
- Does the synthesis connect the evidence to plausible future developments,
  with assumptions, uncertainty, and indicators that could support or
  weaken those interpretations? A projection must be recognizable as an
  inference rather than a reported fact.
- Did optional computation or service use answer a useful question? Record
  what the agent chose and why, including a justified decision that no
  service was useful. Tool usage alone does not demonstrate better research.

If a download or computation outlives a serving, arrange a real completion
callback using the currently available agent contracts. The Observer is an
available option for a reachable job. Leave the job identity, outputs, and
next action in the owning conversation; do not assume a background process
will wake the researcher by itself.

## 5. Verify the evidence and reproducibility

Inspect the committed methods and report artifacts, not only Front's final
summary. Select central quantitative findings and consequential definitions
for independent checks against their original sources. Recompute selected
figures and check that prose, tables, and charts agree. Have the responsible
researcher correct defects in `main/` before publication review.

For any computational analysis, use a fresh temporary workspace or an empty
study-owned database to reproduce a representative result from tracked
material and newly retrieved online data. Do not delete the original
database to perform this check. Methods should retain what the chosen
analysis needs: source identifiers and retrieval instructions, relevant
versions or dates, dependencies, transformations, schema/load steps where
applicable, and commands or parameters. Keep environment values configurable
and provide generic examples without real credentials or host details.

Distinguish exact replay from recomputation against updated source data.
Record checksums or versions where available and explain limits caused by
revised, withdrawn, or access-restricted sources. A local database dump as
the sole surviving input does not meet the braindump's reproducibility goal.
For a study without computation, verify representative retrieval and the
documented reasoning instead of inventing a database exercise.

A small follow-up serving may consume the saved methods for this check or
address a gap discovered by the first report. A second full industry survey
is unnecessary unless the first result gives a substantive reason for one.
Record whether methods were reused, extended, or merely proposed, and where
the optional-tool experiment remains untested. One successful trial cannot
establish a general improvement in agent autonomy.

## 6. Prepare publication and close the episode

Run the existing independent `publish` routine for `studyindustry`, with
its repository mapping supplied through project context. Review methods,
scripts, reports, and their navigation together. Preserve the specified
origin's existing history and license.

Correct issues in `main/` first, then copy reviewed files unchanged into
`publish/`. Inspect substantive deletions in the review diff: rewrite or
relocate useful knowledge rather than discarding it as workflow residue.
Check relative links in the resulting tree and ensure readers can understand
the material without private infrastructure or agent-workflow context.

Commit and push the episode documentation and working-repository changes
under `localrule.md`. For the separate publication workflow, record the
current standing contract explicitly in `README_PROJECT.md`: the existing
`publish` routine prepares a reviewed local commit for the developer's final
push. A destination URL identifies the repository; it does not itself
change that routine contract. Report publication preparation and remote
publication as separate outcomes, and carry any explicit implementation-time
override into the run request rather than relying on an old episode's rule.

Have Front verify results, record task acceptance and mission completion
through the current interfaces, and deliver the routine's terminal report
to the originating conversation. Check the actual resulting topic and work
states. Retain an honest partial result if research remains incomplete.

Write this episode's `report.md` with:

1. The project/repository layout, guide version, and run/work references.
2. The video game industry's research scope, findings, and artifact commits.
3. Selected source and numerical checks, and the fresh-reproduction outcome.
4. Optional tools considered or used, advance-notification evidence, service
   reuse or creation decisions, and remaining operational ownership.
5. Publication review results and the local/remote commit states separately.
6. Costs and timing when available, observed failures, and limits of the
   autonomy experiment. For direct outside assistance, include the standard
   note: “did X for agent Y — handoff candidate.”

Keep local infrastructure evidence in ignored episode notes. Documentation
changes need a diff/link review, not a new test suite. If implementation
requires code changes, run focused checks appropriate to the affected owner;
the real study, artifact audit, and reproduction are the principal proof.
