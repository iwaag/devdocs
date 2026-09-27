# Failsafe p3 — close the gaps and retire experimental residue

## Goal and scope

Consolidate the Front → autolab recovery path built in p1–p2: reliable
replies and acceptance, honest health evidence, current developer reviews,
and an accounted-for backlog. Preserve p2's five-minute silent-exit target.

Use `../p2/report.md`, `report4.md`, `report5.md`, and `report6.md`, plus
`../p1/report.md`, as the initial issue inventory. Verify current code and
records before assuming a reported limitation still exists.

This remains a private experimental, breaking-change phase. No backward
compatibility is required. Choose implementation details freely and remove
superseded paths. Scope cleanup by ownership and usefulness, without a
blanket non-destructive policy or production security expansion. A real
unfinished request is not completed merely to make the board clean.

Do not expand into all-agent/all-harness instrumentation, cross-host
transport, reassignment, or replication. Address Front's missing-reply
failure on this path; that does not require instrumenting every agent.

## step1 — inventory current gaps and cleanup targets

- Inspect current deployments through Nautobot or `nctl status` / `nctl
  drift`, following `pj-clusterintent/nctl/README.md` and local environment
  notes. Distinguish observed state and observation age from old reports.
- Create a compact disposition table in the phase report: evidence, owning
  component, planned fix or cleanup, verification, and any remaining decision.
- Cover nested-fence reply truncation, misplaced task agreement, Front's
  missing reply, health wait semantics, late review updates, and static
  review hypotheses. Include trial records and the old tracked backlog.
- Reproduce defects from saved transcripts where possible. The reply
  splitter already claims fence support: identify the actual failing form
  rather than adding another generic "support fences" workaround.

Hints: inspect `pyagag/src/agag/{reply,topics,execution,health,trace}.py`,
`pj-agdev/agobserver/src/agobserver/{monitor,health,review}.py`, and
autolab's `zulip_listener.py`. P2's known legacy items include held request
o11711/m11741; p1 lists m6113, m7601, m7732, m8519, m9349, and m9697.
These are investigation targets, not instructions to resume or cancel them.

## step2 — repair reply delivery and acceptance routing

- Fix the reproduced `ag-reply` truncation at its owning boundary. Preserve
  code blocks and text following them; use captured p2 output as a regression
  case. Select an unambiguous reply representation and update guides if needed.
- Diagnose Front's no-reply serving. Ensure a failed reply remains a visible,
  unresolved obligation and has a bounded repair or escalation path, even
  when the process exited normally. Reuse existing reply repair and serving
  journals; avoid replaying completed side effects just to regenerate a reply.
- Correct agreement routing: a task agreement reaches that task's record.
  An agreement received in the plan can be routed or explicitly returned for
  correction, but the planner must not claim closure without close-out evidence.
- Keep resumption, shown result, task acceptance, integration, and mission
  acceptance distinct. Exercise both successful acceptance and a refused
  close-out; reported status must match the actual record.

## step3 — make health evidence and wait behavior trustworthy

- Separate harness activity from evidence of work progress. Repeated system
  events or the same open tool call should not indefinitely certify progress.
  Record the nature and age of the evidence instead of treating all events
  as equivalent.
- Exercise the real `waiting` verdict, which p2's healthy trial never reached.
  Inspect a wait target where practical; distinguish an explained long wait
  from a wait that is merely named but no longer advancing.
- Define a bounded reassessment policy for unchanged waits and inconclusive
  activity. A healthy long task can continue without duplicate execution;
  an unexplained hang must reach investigation rather than reset uncertainty
  forever. Choose intervals from trial evidence, retaining p2's timing goals.
- Verify that several slow or failed probes cannot serially consume the
  entire monitor cycle and starve another request's deadline. Add scheduling
  isolation or a cycle budget if the reproduction shows it is needed.
- Keep unsupported harnesses explicit. Do not manufacture certainty from
  missing tool-result events or implement every adapter to complete this phase.

## step4 — keep developer reviews current and actionable

- Propagate later recovery or changed outcome from a reported incident into
  its existing review occurrence, preserving the original escalation history.
  Repeated passes and restart must not create duplicate occurrences or updates.
- Replace fixed per-kind text as the only diagnosis with a bounded,
  evidence-based assessment: observed failure, plausible cause, confidence,
  missing evidence, and a concrete improvement or investigation candidate.
  Agent analysis may run asynchronously; handoff and recovery must not wait
  for it. Unknown cause remains a valid result.
- Keep grouping by symptom distinct from an established common cause.
  Attach follow-up decisions and fixes to the relevant occurrences; avoid
  presenting injected test failures as unexplained operational recurrences.
- Distinguish "reviewed", "fix planned", "fixed", and "accepted limitation"
  as needed without creating a new tracker. A topic's resolution must not
  imply the original mission was accepted or its underlying defect fixed.

## step5 — clean up p1–p2 residue and bound retention

- Inspect trial missions, conversations, review topics, fault files, timing
  overrides, temporary processes, and mission workspaces. Retire completed
  disposable trials and redundant artifacts after retaining useful evidence.
  Keep reusable fixtures and fault hooks; remove accidental active settings.
- Reconcile the two p2 review topics with their final outcomes and trial
  provenance. Record their findings and remaining fixes before resolving
  disposable experiment follow-ups; do not impersonate a developer acceptance.
- Give each old tracked mission and o11711 an explicit disposition based on
  current records: already terminal, still active, replaced, or held pending
  a named decision. Do not replay the old backlog merely by removing the
  rollout horizon. Prepare decision-ready summaries for genuinely unresolved
  human intent; complete independent cleanup without waiting on those decisions.
- Fix unbounded tracking of plain conversations. Define when a satisfied
  conversational exchange can leave active tracking, while unfinished work,
  owed replies, and held requests retain their meaning. Verify rediscovery
  on a new request and survival across restart; age alone is not completion.
- Bound accumulated execution snapshots, probe state, and closed incident
  caches where needed. Preserve references needed by active work and review;
  stale indexes should not re-create retired work after restart.
- Update affected dependency pins and remove obsolete configuration or
  duplicated code based on actual consumers, rather than updating everything
  solely for version uniformity.

## step6 — prove the consolidated path and report

Minimize test waiting without weakening evidence. Use injected clocks,
captured transcripts, and short configured intervals for repeated trials.
Do not wait 30–45 minutes to test a timer boundary. Preserve real process
and delivery behavior in integration trials, and distinguish accelerated
results from operational measurements.

| Trial | Required outcome |
|---|---|
| Nested code fences and text after them | The complete intended reply reaches its recipient. |
| Front ends without a usable reply | The owed reply is repaired or escalated within a stated bound; no silent completion or duplicate work. |
| Agreement reaches the plan instead of the task | Correct routing or explicit correction; no unsupported closure claim; actual acceptance completes close-out. |
| Quiet live wait and a stuck wait with repetitive activity | Live `waiting` is observed; healthy work continues; the stuck case receives bounded investigation. |
| Several probes time out | An unrelated stopped task still meets its detection target. |
| Reported incident later recovers, with restart during update | Its review shows the later outcome once, with original evidence retained. |
| Retention and cleanup followed by restart | Completed trials stay retired, active/held work survives, new requests are discoverable. |

Run one bounded end-to-end silent-exit recovery under operational settings:
exit → Front within 300 s, resumption without Omni Agent rescue, explicit
acceptance, and an accurate developer follow-up. Combine scenarios where
useful; run affected suites and repeat only trials impacted by subsequent fixes.
Restore trial timing overrides and remove active fault files afterwards.

Write `report.md` with the issue dispositions, cleanup performed, measured
timings, false interventions, duplicate actions, costs, tests, and remaining
limitations. Record developer interventions honestly, including fault release.
Keep machine-specific details in ignored notes. Commit and push changes in
their owning repositories and update development documentation.

P3 is complete when the covered defects are fixed and verified, reviews
reflect final known outcomes, test residue is retired, retention is bounded
without losing obligations, and the five-minute recovery target still holds.
Every remaining legacy item must have an explicit owner and disposition;
pending human intent may remain held and documented, never silently closed.
