# sage p1 — final report

## Result

`arxivsage-agstudio1` is a live, launchd-managed agag agent for the public
`iwaag/study-arxiv-trend` knowledge tree. It serves `entrance-…` topics in its
own Zulip channel, cites the files it read, records researchable unknowns in
an ignored study queue, and reports `liveness=polling` after a fresh nctl
observation.

Step evidence: [generation](report1.md), [listener](report2.md),
[contract](report3.md), [knowledge deployment](report4.md),
[launchd](report5.md), [acceptance](report6.md), [desired declaration](report7.md),
and [documentation](report8.md).

## Delivered repositories

- `iwaag/arxivsage`: generated project, knowledge-sync script, custom sage
  listener, scope guide, intro, launchd template, and locked GitHub pyagag
  dependency.
- `iwaag/pyagag`: commit `a49ec8e` preserves explicitly supplied run metadata
  in persisted records; focused agent/harness tests passed (35).
- `iwaag/devdocs`: this phase's reports and the shared sage documentation.

The arxivsage listener and its direct dependency updates were pushed to GitHub
before being used locally.

## Acceptance and costs

Six requested acceptance cases passed after one intentional failure-farming
guide correction. The cases consumed $0.5529926 across sage runs (including
the observed failure and its retry). A post-fix metadata verification run cost
$0.0856824, for a total observed sage cost of **$0.6386750**. Each record has
cost, turns, transcript path, and — after the pyagag fix — the knowledge
revision; `run-0007` records `knowledge_revision: "31ff73c"`.

## Open handoff: study queue consumer

The next phase must make someone read `arxivsage/tostudy/`, run the requested
study, publish the resulting paper directory to `study-arxiv-trend`, and
delete the consumed note. The first produced note demonstrates the contract:

```md
# AutoDesign (arXiv:2608.13560)

Asked in <channel> by <asker>: "<question as asked>"

## Why in scope

<why this is an LLM-agent or agent-harness research request>

## Status

Not present in knowledge/README.md at revision <revision>.

## What to look for

- arXiv id: <id>
- Title: <title>
- Requested artifact: a summary (summary.md)

- asked again in <channel>/<topic>: "<one-line question>" (<date>)
```

The intended consumer is a `workplan-` topic in `#pj-studyarxiv`. Its result
must land in the public `publish/` tree and be pushed by the developer; then an
operator runs `sync_knowledge.sh`. No automatic refresh or study-side consumer
is part of p1.
