# Retire the old agdevworld UI — phase 1

## Objective

Make the workflow-editor-based agdev dashboard the main entrance, integrate
useful usage information, and remove obsolete screens and their dedicated
runtime dependencies.

Source: [braindump.md](braindump.md), with the Developer's subsequent scope
decision: Front Desk evolution belongs to a separate episode.

## Scope and decisions

- Make the dashboard the default site entrance and unify navigation around it.
- Integrate the cost gauge and subscription usage windows into the dashboard's
  visual language and connect cost reporting to current WFE execution records.
- Retire World views, Operation room, Ops board, and obsolete routine UI,
  including code, assets, polling, and backend paths used only by those screens.
- Keep the existing Front Desk working as a separate screen linked from the
  dashboard. Preserve its current conversation URLs and behavior for this phase;
  this is an active feature, not a compatibility layer for retired screens.
- Keep historical conversations and execution records. Retiring their UI does
  not authorize deleting the underlying history.
- Do not migrate projects that have not yet moved to WFE.
- Do not rebuild cluster visualization or other retired screens in the dashboard.
- Do not maintain compatibility routes, adapters, or fallback screens for
  retired UI. Existing WFE routes may remain where changing them has no benefit.

The future Front Desk episode will consider an entertaining conversational UI
for talking directly with project agents, including Front. Its interaction
design, presentation, and relationship to workflow execution are outside p1.
Do not move or redesign Front Desk as a prerequisite for this retirement.
Phaser and other shared dependencies remain wherever Front Desk still needs them.

## Development and operating policy

Follow `localrule.md`, `devpolicy/styles.md`, and `devdocs/README_DEV.md`.
Generated files are English; communication with the Developer is Japanese.
Implementation, commits, pushes, deployment, and LAN operations need no
additional human approval. Planning records and validation results are not
permission gates.

Use Nautobot or `pj-clusterintent/nctl` to establish service state before
operational changes. Consult `pj-agdev/.local/devenv.md` for local deployment
details, and verify them rather than treating historical notes as current facts.
Keep addresses, host names, credentials, and machine-specific paths in ignored
local evidence. Preserve LAN access in the resulting deployment.

Finish implementation with a deployed result that the Developer can inspect
with few operations, plus a report containing review links and evidence.

## Investigation baseline

Read-only investigation on 2026-10-20 established:

- `agdevworld/src/main.ts` opens the old operation dashboard by default.
  The current agdev dashboard is in `agdevworld/workflow_editor/`, exposed
  through the web server's `/wfe/` route.
- The current Front Desk uses WFE conversation APIs through its room adapter,
  while its scene and supporting modules still live in the old frontend.
  Shared dependencies must be inspected before deletion.
- The cost gauge reads `/cost` and `/budget` from the agentroom relay.
  `/cost` also consults the old ops engine for roster and routine attribution.
- The live cost response covered historical agent roots, including the retired
  Front, and did not represent current WFE activity. Moving that response into
  a new card would not fulfill the usage objective.
- The live budget response returned subscription usage windows. Preserve this
  useful capability, including provider-specific failures and freshness.
- nctl returned a converged result for the local service host using its stored
  observations. This is a baseline, not proof that every application route is
  healthy or that every observation is fresh.

## Step 1 — Establish the dependency and data boundaries

Inspect frontend imports, routes, links, assets, relay consumers, deployment
configuration, and checks. Classify each relevant part as:

- dashboard or Usage functionality to retain or move;
- existing Front Desk functionality to retain;
- functionality exclusive to retired UI to remove.

Trace Front Desk's active adapter and actual calls, including shared modules
whose legacy branches can be removed without changing its behavior.
Check callers outside the frontend before removing backend endpoints.

Trace usage evidence from WFE attempts through executor/harness records on
local and remote workers. Establish record identities, available usage fields,
where records are retained, and how they reach the coordinator. Choose the
smallest collection path that supports current executions without a new
general-purpose telemetry system.

Record collection coverage explicitly. A missing worker, unreadable record,
or run without usage must not look like zero usage. Establish deduplication
across worker copies, coordinator copies, retries, and resumed sessions.
Check whether harness usage is per attempt or cumulative before summing it.

## Step 2 — Integrate Usage

Add a compact Usage summary to the dashboard and a dedicated detail view.
Use the dashboard's typography, spacing, surfaces, colors, and Simple/Detail
conventions. Keep the main page focused on useful status:

- subscription window usage and reset times;
- a concise cost/usage summary with collection coverage;
- visible unavailable or stale states.

The detail view provides period summaries, useful breakdowns, token counts,
and links to the corresponding project/run where attribution is known.
Replace obsolete routine-session attribution with current WFE identities.
Keep historical records available without presenting them as current activity.

Preserve the meaning of the existing measurements:

- distinguish harness-reported USD, estimated USD, and subscription quotas;
- label API-equivalent amounts as such when they are not actual charges;
- do not invent per-run USD for subscription usage;
- display unknown or uncollected usage distinctly from zero;
- expose observation time and incomplete coverage;
- isolate provider failures so one failure does not blank the whole view.

Reuse the working budget readers and useful normalization logic. Remove their
dependency on old roster/routine reconstruction. Choose their final service
boundary based on runtime requirements, including access to host-local CLI
credentials; retaining that reader does not require retaining the old ops
engine. Browser access should use the dashboard's origin where practical.
Keep caching, bounded reads, and polling cleanup on navigation.

Validate current local and remote execution coverage against source records.
Where a worker or harness cannot supply a measurement, state that limitation
explicitly instead of silently dropping its executions.

## Step 3 — Make the dashboard the main entrance

Update the site entry and web routing so opening the site reaches the agdev
dashboard. Update shared headers and links to use it as Home.
Account for both existing web listeners and the direct WFE service; verify
asset paths, API prefixes, origin handling, deep links, and event streaming.
Use the request's host rather than introducing machine-specific URLs.

Provide clear navigation to projects, runs, execution nodes, Context, Usage,
and the existing Front Desk. Keep the existing Front Desk route reachable
before applying the default-entry change, including conversation query
parameters used by Slack links. Do not redirect those conversations to Home.

Retain the existing boundaries on host-path operations when changing proxy
routes. Main-page routing must not accidentally change which operations are
available through the web origin.

## Step 4 — Remove obsolete UI and dedicated runtime code

Delete the retired screens, their navigation, dedicated renderers, styles,
assets, sample fallbacks, polling loops, and obsolete checks. Remove unused
cluster snapshot publication machinery if no remaining consumer needs it.

Remove relay routes and reconstruction/background work whose consumers have
all been retired. Retain only what the surviving features and other verified
callers require. Stop or replace a service only after its useful readers have
an operating destination.

Remove unused dependencies, scripts, configuration, and launch definitions.
Keep Phaser and shared frontend support required by Front Desk. Do not expand
this step into a Front Desk rewrite or a cleanup of unrelated agent systems.

Update current development instructions and ownership documentation. Preserve
episode history while marking obsolete operational instructions as historical.

## Step 5 — Verify and deploy

Run checks appropriate to the affected packages: WFE checks, remaining
frontend build, relay/usage tests, and executor tests if collection changes
touch the executor. Add focused tests for usage attribution, duplicate records,
cumulative usage where applicable, missing data, and provider failures.

Use browser verification to exercise:

- site root to dashboard, project, workflow, and run;
- Usage summary/detail navigation and fresh/stale/unavailable responses;
- representative usage totals reconciled with source records;
- existing Front Desk conversation loading, history, and related run links;
- Front Desk composition and response behavior on an isolated fixture;
- existing Slack-style conversation links;
- narrow and desktop layouts;
- absence of old UI links and requests to removed endpoints.

Deploy the affected web and service releases using the established release
tools. Coordinate service restarts with current executions and preserve
operating data. Keep the previous release identifiers and any necessary
configuration backups as recovery evidence.

Verify the deployed site over the LAN, its service versions, current Usage
data, and existing Front Desk reads. Live agent messages are not required for
this UI retirement; use isolated fixtures for conversational writes.

Update Nautobot through nctl if service ownership, placement, or lifecycle
changes. Check the resulting registered and observed state. Record unavailable
checks and their practical implications rather than treating them as passes.

## Completion criteria

1. Opening the site presents the agdev dashboard.
2. Useful Usage information is integrated visually and reflects current WFE
   execution evidence, with explicit coverage and unknown values.
3. World views, Operation room, Ops board, and obsolete routine UI are absent,
   along with their exclusive assets, routes, and background work.
4. Existing Front Desk conversations remain accessible and functional without
   being redesigned or migrated into the dashboard.
5. No unrelated project migration or deletion of historical records occurred.
6. The deployed result works over the LAN, and relevant checks pass.
7. `report.md` records changes, validation, collection limitations, deployment
   evidence, review links, and the remaining Front Desk episode boundary.

Tracked reports use portable references; machine-specific review URLs and
operational details belong in ignored local evidence and the final chat.
