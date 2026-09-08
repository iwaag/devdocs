# study-realworld p1 — Plan

Implement `braindump.md`: discover reusable sources of public institutional
information, investigate their documents, and accumulate readable reports.
Start with one `study-realworld` routine, with discovery and investigation
selected through natural-language requests.

This is an experimental, non-public execution environment and a destructive
development phase. No backward compatibility is required. The implementer
may simplify or replace existing arrangements and choose tools, formats,
sources, and task boundaries. The layout and examples below are starting
points, not a prescribed sequence of agent tool calls.

## 1. Establish the project

Use an Autolab study project, provisionally `studyrealworld`, and document
its repositories and conventions in `README_PROJECT.md`:

```text
main/                         # working knowledge repository
  README.md
  sources/
    INDEX.md
    <source-id>.md
  reports/
    <source-id>/
      INDEX.md
      YYYY-MM-DD-NNN/
        report.md
        ...                   # optional investigation material
publish/                      # reviewed knowledge, same relative layout
```

- Create `main/` through the ordinary `autolab project init-repo` route.
- Clone `https://github.com/iwaag/study-realworld.git` into `publish/` and
  use that URL as its origin. Inspect its existing contents when setting up.
- Adapt the study pattern locally in `README_PROJECT.md`: source acquisition
  recipes are knowledge too. One investigation directory is sufficient;
  separate repositories per investigation are optional, not required.
- Keep reports and acquisition recipes suitable for readers outside the
  execution environment. Large downloads and execution logs can live in
  ignored `.local/`; retain source links and enough metadata to trace them.

## 2. Make sources reusable and reports traceable

One source file represents a useful acquisition unit: an institution's
minutes collection, report series, or dataset, rather than necessarily the
whole institution. Give it a stable ID independent of its current URL.

Record the publisher, geography/topic, available information, official
entry points, how to find new material and retrieve its body, one successful
example, update cadence if known, last verification, and useful limitations.
The source index summarizes these entries and distinguishes candidates from
sources whose document retrieval was actually verified.

Each investigation records its question and coverage, findings, evidence
links, and unresolved questions; additional files and structure are free.
For cited documents retain title, publisher, publication date, covered period,
version when available, retrieval time, and relevant pages or sections.
Distinguish the publisher's claims from the investigator's interpretation.

Update each source's report index with the date, scope, result, and report
link. On later runs compare document IDs/URLs and versions with previous
work. Revised documents and new questions can justify another investigation;
link it to the earlier one. Record no updates, retrieval failures, and partial
coverage honestly, without manufacturing a full report for an empty check.

## 3. Define and exercise the routine

Write the standing request in `#front` → `routine-study-realworld`.
Front supervises ordinary Autolab workplans and their workruns in the project.

- **Discovery:** find sources for a requested topic/region, test retrieval,
  save reusable recipes, and write a short substantive report from retrieved
  material. Finding a URL alone is not verified discovery.
- **Investigation:** read the source and report indexes, choose relevant
  registered sources, retrieve documents, report findings, and update indexes
  and acquisition recipes as needed.
- Requests may specify topic, region, period, sources, and an effort budget.
  Use a small bounded selection when unspecified. Scheduled runs default to
  investigation; an empty catalog starts with discovery. Explicit requests
  can select either mode without changing the standing default.

Reuse the current dispatcher and Front conversation path. First run a small
discovery mission covering roughly three sources with differing acquisition
methods. Then run an investigation through a one-shot scheduled event,
using the saved recipes and previous reports. It is fine to correct real
workflow failures and repeat only the affected part. No recurring schedule
is needed to prove p1.

## 4. Exercise publication

Use the existing independent `publish` routine for this project; extend its
standing request if it assumes arXiv-specific paths. Review the source
recipes, reports, README, and indexes as a coherent collection.

Correct publishability defects in `main/`, then copy reviewed files into
`publish/` unchanged. Check that relative links work within the published
tree. Explain the sources and reports to readers without requiring knowledge
of Front, Autolab, or internal repositories. Avoid committing credentials,
private machine facts, or substantial third-party document copies; concise
summaries, evidence references, and acquisition recipes are sufficient.

Commit and push completed implementation and knowledge changes per
`localrule.md`, including reviewed publication content to the specified
GitHub origin. The current session's commit/push instruction takes precedence
over the study pattern's older manual-push convention.

## Implementation hints

- Read `pj-agdev/agautolab/agent/project_pattern.md` and use
  `autolab doc patterns` / `autolab --help` for the current project commands.
- `pj-agdev/devenv/routine/trigger.sh` already accepts arbitrary routine names
  and creates `front-routine-<name>-<UTC stamp>` per execution. Its standing
  request topic contains instructions; put execution discussion in the run
  topic. A scheduled fire currently carries no arbitrary mode payload:
  natural-language Front requests and the standing default suffice for p1.
- Creating an Autolab workplan does not start its tasks. Post to the workrun
  topics it creates and follow completion through Front's normal callbacks.
- Prefer API/RSS or straightforward document downloads where available.
  Browser-based retrieval and source-specific helper scripts are legitimate
  choices. Add automation when repeated work or observed failures justify it;
  a generic crawler, database, new agent, or mode CLI is not a prerequisite.
- For running service facts use Nautobot or `nctl`, as described in
  `pj-clusterintent/nctl/README.md`. Planning-time `status` was healthy and the
  inspected host drift was converged; refresh relevant facts during execution.
- Let in-system agents produce the study evidence. Direct Omni intervention
  is allowed; record `did X for agent Y — handoff candidate` when applicable.

## Completion evidence

Write `report.md` with the resulting layout, routine definition, run/topic
and schedule references, commits, costs/timing when available, and useful
failures or follow-up advice. Keep private environment details in ignored
notes. Demonstrate that:

1. Discovery retrieved real documents and produced reusable source files and
   substantive reports, linked from the indexes.
2. A later run used those records to investigate without rediscovering the
   acquisition route; repeated, revised, missing, or unchanged material was
   handled honestly. A live update need not occur for this check to pass.
3. The scheduled run reached actual work and completion, beyond an ack or
   merely opening a workplan.
4. Reviewed knowledge reached the specified publication origin with working
   links and traceable evidence.

Use focused checks for any code actually changed. The two real study runs
and publication check are the principal validation; document-only changes
do not need a new automated test suite. Split discovery and investigation
into separate routines later only if observed cadence or workload warrants it.
