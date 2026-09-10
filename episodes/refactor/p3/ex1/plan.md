# Refactor p3 ex1 — remove obsolete surfaces and finish execution options

## Goal and working latitude

Finish the useful follow-ups from refactor p1–p3 and runtime-profile: remove
obsolete product paths, publish working execution options for agforge and
arxivsage, correct the remaining observed inconsistencies, and close the gaps
in execution-option validation and evidence.

Prefer deletion when a feature, configuration, or deployment no longer serves
the current workflow. This is a breaking-change experiment: no compatibility
layer, migration of disposable old work, replacement legacy dashboard, or new
approval framework is needed. Choose implementation details and refactoring
boundaries freely. Trace consumers and ownership before deletion; keep local
machine facts and credentials in ignored files. Historical reports are useful
evidence, not active dependencies to erase merely for mentioning Plane.

Paths below are relative to the shared projects directory.

## step1 — Remove the dead autolab UI and its unused dependencies

- Delete `pj-agdev/agdevworld/src/autolabState.ts` and the UI that depends on
  the removed `/api/autolab/*` assistant gateway. Remove its navigation,
  selection types, job/project popups, summary polling, and dedicated fixtures
  and documentation. Reconnect navigation among the retained views.
- Trace `views.ts`, `detailPopup.ts`, their callers, and startup/configuration
  references. Keep shared rendering utilities only where current views use
  them. Use the existing agent room, operation room, and conversations to
  inspect work; rebuilding the old gateway is unnecessary.
- Follow obsolete callers into backend, deployment, and local startup files.
  Delete further dead code and settings when no current workflow uses them.
  Establish actual usage before retiring a whole gateway or remote service;
  a dead browser caller alone does not establish that every endpoint is dead.
- Inspect disposable pre-refactor work and fixture agents discovered during
  the sweep. Retire or remove entries with no current purpose instead of
  preserving adapters for them. Record meaningful removals in the report.

Verify: TypeScript and build checks pass; browser navigation and retained
detail views work; browsing them makes no `/api/autolab/*` requests and offers
no controls for removed features.

## step2 — Publish and apply agforge and arxivsage execution options

- Use `ag.exec-options.v1` and the existing `AgentSpec` / `context.selection`
  machinery. Publish options in each agent's introduction and wire selection
  into actual runs, rather than only adding a menu.
- For agforge, cover entrance replies, asset planning, generator runs, and
  collection after asynchronous callbacks. Inherit the plan's selection into
  its child run before work starts, following existing snapshot semantics.
  Distinguish the agent's LLM execution option from media-generation models
  and toolsets: selecting agy must not silently change the requested asset.
- For arxivsage, wire selection through `src/arxivsage/listener.py` and its
  serving routes while retaining its knowledge-reading and study-queue role.
  Its committed configuration currently has only `sonnet` and the test `stub`;
  add a usable alternative such as agy if supported by the environment, rather
  than publishing fake choices. Keep test-only profiles out of public menus.
- Publish default behavior, available alternatives, usage pools, and coverage.
  Verify runtime availability on the actual host. Reuse existing agent
  configuration patterns, with role-specific mappings only where needed.

Hints: forge's `instance.py`, `role_run.py`, `entrance_topic.py`,
`assetplan_topic.py`, and `assetrun_topic.py` contain the integration points.
Autolab already implements child inheritance; Front and autolab's instance
modules demonstrate menu publication. Keep the shared command/reset/refusal
semantics rather than inventing agent-specific variants.

Verify: discovery, selection, reset, invalid-option refusal, child inheritance,
restart recovery, and callback continuation. Read written run records to
confirm the selected option and actual backend; returned dictionaries alone
missed a persistence bug in the original episode.

## step3 — Make published usage pools match effective execution

- Derive or validate pool declarations against the resolved harness/provider
  for each covered role, including instance overlays and default behavior.
  Front/autolab currently filter public names against configured profiles but
  hardcode pool descriptions, including the default's `anthropic` pool.
- Apply the same rule to the new forge/sage menus. Prefer a small shared
  helper to four independent implementations. Support truthful descriptions
  of mixed-role defaults; do not force a single pool where execution differs.
- Detect a mismatched declaration before publishing or accepting it, with a
  useful diagnostic. Keep transient harness availability failures distinct
  from an invalid contract, so one unavailable option need not disable every
  unrelated conversation.
- Retain the current provider-level pool convention for this environment.
  If inspection finds multiple accounts behind the same provider, distinguish
  the relevant account in execution and observation; otherwise no account
  registry is needed.

Verify: wrong pool, changed default overlay, role-specific mapping, and mixed
defaults cannot produce a misleading single-pool advertisement.

## step4 — Resolve the remaining desired/observed inconsistencies

- Read `pj-clusterintent/nctl/README.md` and use Nautobot/nctl to refresh the
  relevant observations and inspect desired state before acting.
- Reassess `service_missing` on agfront. The p3 report and follow-up read used
  an observation from before deployment; repair discovery, service state, or
  the obsolete declaration according to fresh evidence.
- Resolve `agent_zulip_channel_unsubscribed` for agforge and
  agecho-agautolab1. Agforge's reported channel is the retired `FreeForge`:
  compare the current introduction and desired channels before subscribing
  anything. Remove stale expectations. For an unused fixture agent, retire
  its deployment/registration instead of reviving it solely to clear drift.
- Re-observe and confirm the affected gaps are gone for the correct reason.
  Keep registration separate from process health; informative liveness states
  need not be converted into errors or hidden to obtain a clean result.

## step5 — Deploy and prove the retained workflow

- Run focused component suites and the affected repositories' required gates.
  Update dependency locks/submodule pointers, deploy changed components,
  restart affected listeners, and refresh introductions. Use local environment
  notes for revision-aware Nautobot rebuilds or plist reloads when applicable.
- Perform bounded live demonstrations: discover forge/sage options through
  their introductions and run small requests using an advertised alternative.
  For forge, exercise selection across planning and generation; use fixtures
  for expensive asynchronous edge cases unless live behavior remains unclear.
- Demonstrate an agy-selected autolab task delegating and resuming from a
  callback, with the inherited selection present in the continuation's actual
  run record. p3 proved the callback route but did not establish this property.
- Exercise Front's routine decision with controlled budget observations in
  actual agent servings, not just the renderer: below threshold, exceeded,
  already exceeded, reset, and stale/failed read. Confirm the requested pool
  is the pool observed and that unknown usage is not reported as success.
  A real account need not be consumed to 70%; restore the normal source after
  the demonstration. Record fixture and live evidence separately.
- Messages and temporary settings needed for these bounded demonstrations are
  part of this work. Inspect results, then close or remove obsolete test work
  using the system's normal completion/retirement paths.

## step6 — Close out

- Update active documentation and guides to describe the retained surfaces and
  published options. Remove obsolete operational instructions and local startup
  artifacts found during the work; retain useful historical reports.
- Write `report.md` with deletions, public option coverage, drift resolutions,
  test/build results, live run evidence, and any concrete remaining limitation.
  Do not label fixture coverage as a live demonstration.
- Commit and push changed repositories, dependency locks, and required
  superproject pointers following `localrule.md`.

Done means the dead autolab UI is gone, both agents publish executable options,
pool declarations are checked against effective execution, the three reported
drift targets are resolved, and the missing selection/budget evidence exists.
