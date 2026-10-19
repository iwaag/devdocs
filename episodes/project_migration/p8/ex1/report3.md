# project_migration p8 ex1 — Step 3 report: recovery gaps

Date: 2026-10-19. Source and fixture work; deployment is Step 5.

## Two levels of repair, as built

| Condition | Owner | Response now | Where |
| --- | --- | --- | --- |
| A child's durable fact missing from the parent | delegation relay (intake pass, executor reconcile) | relayed by receipt, once; a completion once per completing attempt (Step 2) | `DelegateStore.relay`, `childFacts` |
| Parent has an actionable fact or answer, no attempt, no job, no hold | executor reconcile | queued again (existing) | `ExecStore.reconcile` |
| **An unread instruction whose job was lost** | executor reconcile | **queued again** under its receipt `instruction:<ws>/<run>/<i>` | new |
| **A `run.resume`/`run.recover` after the last attempt, its job lost** | executor reconcile | **queued again** under the resume/recover receipt; never for a completed or held run | new |
| **An open delegation's request/answer/instruction post whose notifier row is missing** | notifier scan | **queued again from the parent record under the same event id**, marked uncertain, so the transport searches for its marker since the event's time before posting | new: `NotifyStore.scanRun` |
| Child process lost, or its record conflicts with its execution | Observer | the child's own incident (`lost-attempt`); the parent's wait stays quiet | existing, now tested from the parent's side |
| Child unreadable from the parent's host | Observer | `unobservable` / `child-unreadable`; nothing concluded; owners after 30 min | new |
| Parent exits without recording its wait or taking up a fact | WFE + Observer | interruption with its structured cause (Step 2); the Observer fact leads with it in short and `affected` names the children concerned; the recovery request lists every delegation by identity | new |
| The same interruption keeps coming back | Observer | after two recoveries in a row with the same cause (kind and what it names), the incident goes to the owners, not a third recovery | new: `wfclassify.repeated` |
| Relay or delivery stays stalled, dispatch never bound | Observer | `relay-stalled`, `unbound-dispatch` → owners (existing); `unrelayed[].at` is now when the fact became tellable, so a child that completes late in a long attempt is not reported stalled at once | adjusted |
| Recovery run ends with no work recorded (Step 1 finding 5) | WFE + recovery guide | the end is `interrupted` / `nothing-recorded` (Step 2); pj-agops guide: start and complete `investigate` with `report.md` | pj-agops `bc73863` (pushed in Step 5) |

A long elapsed wait alone is never a failure: a delegation wait observed
eight hours on, with its child working, is `awaiting-delegation` with no
incident (tested).

## Commits

- agdevworld `0b1fe65` — reconcile requeue (instruction, resume/recover),
  notifier repair of missing delegation outbox rows with marker search,
  `wfe observe` `hold.cause`, `attempts.ends`, `delegations[].childUid`;
  `docs/observation.md`.
- pj-agdev `edf445c` — Observer: cause text, repeat rule, unreadable child,
  recovery-request delegation context; `agobserver/docs/workflow.md`.
- pj-agops `bc73863` — recovery guide.

## Failure-injection evidence (fixtures; all pass)

WFE `test/reconcile.test.ts`:

| Injected | Result |
| --- | --- |
| an instruction's job deleted | reconcile: `resume for instruction i1 queued again`; a second reconcile adds nothing; one job |
| a person's resume, its job deleted | reconcile requeues once; the parent is claimed |
| the request post went out, then its outbox row, delivery and delivery link were deleted before the intake saw it | scan: `queued … again (its outbox row was missing)`; the transport's marker search finds the earlier post; **one request post, one child**, bound |
| an answer's outbox row deleted before it was sent | queued again; the child records the answer **once** |
| an attempt leaves a node running, is recovered, and leaves it running again | the observation gives `hold.cause = left-running [use]` and `ends` = exited / interrupted (recovered) / interrupted |

Observer `tests/test_wfmonitor.py` (120 tests pass):

| Case | Result |
| --- | --- |
| delegation wait observed 8 h later, child working | parent `awaiting-delegation`, no incident; the child's own silence is the child's suspicion |
| child process lost | child `lost-attempt` (recoverable); parent stays `awaiting-delegation` |
| child record unreadable | parent `unobservable` / `child-unreadable`, not recovered |
| parent interrupted with a missed completion | fact `held interrupted after attempt …: d1 completed #1 not taken up (…)`; `affected` has the child; the recovery request has the cause, the child uid and situation, the untaken fact and the no-replacement rule |
| the same `left-running [talk]` after two recoveries | `repeated`, not recoverable → owners; with another cause between, recovered as usual |

Step 2's lifecycle tests cover crashes between event persistence, queueing
and take-up (a lost continuation requeued once; a fact during the attempt
continued at its end; a fact before it and not taken up a `missed-event`
interruption) and restarts of every store.

Full WFE suite: 434 tests pass.

## Limits

- Repairs are of records this host derives from run records (jobs, outbox
  rows). A Zulip post that vanished from Zulip itself is not recreated
  without a person (`wfe notify resolve`), as before.
- The repeat rule needs causes, so it applies to holds recorded after this
  release; older holds keep the existing per-episode handling.
- Recovery run records written before the guide change stay as they are
  (five `not-started` runs); they are history, not repaired.
