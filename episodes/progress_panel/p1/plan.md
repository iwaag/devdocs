# Progress panel p1 — see plans and runs in the Front Room

## Goal and approach

Add a show/hide progress panel to the Front Room (Front Desk). Show multiple
requests together, including requests in different Front conversations, with
plan meters and run indicators. A person should see what is advancing, what
is waiting, who has the next move, and whether they need to respond.

Use implementation and two concurrent study trials to expose and fix gaps in
the underlying observability. This is a private experimental, breaking-change
phase: backward compatibility is unnecessary. Implementers may choose the
layout, API shape, caching, and refactoring approach. Prefer existing evidence
and shared interpretations; avoid adding a parallel task ledger or elaborate
security and migration machinery.

## step1 — map evidence and choose the display model

- Read the failsafe p1–p4 reports and inspect the current implementation.
  Check local deployment state through Nautobot or `nctl status` / `nctl drift`
  (`pj-clusterintent/nctl/README.md`); use ignored environment notes for host
  details.
- Trace one representative request through Front, plan, tasks/runs, result
  delivery, acceptance, and completion. Inventory available facts, missing
  links, and which owners expose execution health.
- Establish card identity and scope using recorded anchors and relations.
  Start with active Front requests across conversations, retain recent results,
  and make the current conversation easy to locate. Choose practical bounds
  for history and listing; one request's children belong under its card.
- Specify the few user-facing states and their evidence: planning, queued,
  working, waiting for another agent/tool, awaiting your response, completed,
  cancelled, stopped, and unknown. Keep work state, execution health, and
  recovery status distinguishable rather than forcing them into one enum.

Useful starting points:

- `pyagag/src/agag/trace.py`: request tree, stable anchors, state, execution,
  holder, and delivery receipts; `MirrorReader` supplies reads from a mirror.
- `pyagag/docs/health-v1.md`: serving/ack-specific process and work evidence,
  tool waits, observation time, and unknowns. Instrumentation is opt-in;
  failsafe's deployed health coverage is primarily autolab, not every agent.
- `pj-agdev/agobserver/src/agobserver/`: health checks and recovery records.
- `pj-agdev/agdevworld/agentroom/src/agentroom/`: `frontdesk.py`, `routines.py`,
  `scope.py`, and `closing.py` for existing conversation and work relationships.
- `devdocs/README_DEV.md`: request progress and study lifecycle. Study setup,
  accepted/integrated research, and sage knowledge refresh are separate facts.

## step2 — expose a shared progress read model

- Assemble request cards and their plan/run children through the existing
  relay. Reuse trace, task records, execution health, and Observer evidence;
  share or extract interpretation logic when the consumers need the same rule.
- Include a concise activity description, owner/next actor, wait reason,
  latest work time, observation time, evidence link, and recovery status when
  available. Match health facts to the actual serving/ack, not just the topic.
- Distinguish conversation-only evidence from confirmed execution health.
  An unclosed acknowledgement is not proof of a live process; housekeeping
  events are not work progress, and work activity does not measure completion.
- Use the relay's existing mirror and bounded refresh/cache mechanisms.
  Expose missing, stale, or unavailable evidence explicitly. Keep last-known
  data useful without presenting it as current. Display toggling must not
  control work execution or Observer monitoring.
- Fix concrete record/link gaps needed by the panel. Broader all-agent or
  cross-host instrumentation can remain a documented limitation unless the
  study trials demonstrate it is necessary for this phase.

## step3 — build the panel and plan/run meters

- Add a discoverable progress toggle and a usable panel alongside the Front
  conversation. Support multiple cards, compact summaries, expandable child
  work, and links to the relevant conversation or result. Keep it usable on
  smaller windows and preserve show/hide preference across reloads.
- Render each plan with a segmented meter once its tasks are known: completed
  tasks / current total, with working and awaiting-acceptance segments marked.
  While planning, show that state without inventing a percentage. A changed
  plan may change the denominator; explain that visibly. Task counts are not
  estimates of elapsed time or effort.
- Render runs as smaller meters beneath the plan. Use determinate progress
  only when real completed/total units exist. Otherwise use an activity band
  or pulse supported by fresh evidence, plus the current action and its age.
- Give queued, tool/delegate wait, approval wait, stopped, and unknown states
  distinct text and visual treatment. Stale or unavailable observations must
  not leave an apparently healthy moving meter. Use labels as well as color.
- Keep result delivery, task acceptance, plan completion, and any required
  study knowledge refresh visible until their respective records confirm them.
  A delivered answer or an ended run alone does not complete the whole request.

Hints: `src/scenes/FrontDeskScene.ts` already hosts toggleable panels;
`src/frontDeskClosePanel.ts` and `src/roomState.ts` show existing UI and relay
integration patterns. Choose the visual implementation that fits this scene;
no new design system is required.

## step4 — verify transitions and observability gaps

- Add focused read-model tests using representative recorded conversations:
  planning, changing task totals, queued work, healthy quiet tool waits,
  approval waits, delivery pending, completion, cancellation, and unknown data.
- Check two requests sharing an agent, retries/new servings, renamed/resolved
  topics, and late results. Their identities, counts, and health must stay
  separate. Verify receipt/acceptance semantics against the existing readers.
- Exercise stale observations, relay unavailability/restart, and a stopped
  run followed by recovery. Fixtures, replay, and controlled faults are suitable
  here; another full failsafe campaign is unnecessary.
- Verify the rendered UI in a browser: meters, details, links, show/hide,
  conversation switching, reload, and a small viewport. Confirm passive viewing
  does not start agent work or generate repeated live Zulip reads per card.
- Repair discovered shared-model defects at their owning layer and rerun the
  affected checks. Run the frontend build and relevant backend regression tests.

## step5 — run two studies together and check the live panel

- Deploy the necessary changes and dependency pins. Choose any two suitable
  studies with small, concrete outputs and start them through Front in separate
  conversations, close enough together for their request lifetimes to overlap.
  Existing studies are sufficient; avoid unrelated setup work.
- Observe both cards together through planning, execution, waiting, acceptance,
  integration, and the routine's required sage refresh. Record timestamps and
  screenshots alongside the underlying conversation/record evidence.
- Distinguish concurrent requests from concurrent execution. If an agent queues
  one study, show and report that queue accurately. Actual execution overlap is
  worth measuring but does not require a scheduler redesign in this phase.
- Check that one study's waiting/completion does not change the other's meter;
  hide/show the panel, switch conversations, and reload while work is active.
- Fix misleading displays and backend gaps found by the trial, then repeat the
  affected path. Record human responses and developer repairs so assisted
  outcomes are not described as autonomous successes.

## step6 — consolidate and report

- Remove superseded code, update development documentation and ignored local
  deployment notes, and retire temporary trial aids. Commit and push changes
  in their owning repositories.
- Write `report.md` with the resulting UI, evidence sources and coverage,
  backend defects found/fixed, validation results, both study outcomes,
  concurrency observed, interventions, and remaining limitations.

Completion means the show/hide panel displays both real study requests with
accurate plan/run meters; waits, unknown/stale evidence, and final outcomes are
distinguishable; navigation and reload preserve correct identity; both studies
reach their requested outcomes; and any remaining observation limits are
explicit. A cosmetic meter demonstration alone is insufficient.
