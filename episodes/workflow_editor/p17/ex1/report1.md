# p17 ex1 step 1 — both gaps reproduced

Plan: [plan.md](plan.md), step 1.

## Fixture

agdevworld `workflow_editor/test/reviewRelease.test.ts`, modeled on
gimmickimg/freework/run-013 of conversation `20261025-013759-jz8ebz`:

- a parent (`use`) delegates to the child (`make`);
- the child's graph is `present → review → cleanup`, with a custom
  documentation policy (`final: true`). Compiler v5 puts `_final-report`
  after `present`, as in run-013;
- the child registers p1 and starts its server before preparing its own
  review, prepares rv1 on p1 and writes its final report;
- an identified check actor (`p17ex1-check`, `synthetic: true`) starts
  and approves the review. No recovery agent and no `run.recover` are
  involved.

With `DEBUG_RELEASE=1` the test prints the round state, cleanup results,
presentation state, ready/waiting nodes, open attempt, hold and the
child's open jobs at each boundary. A sweep kills any page a test leaves
serving, whatever its assertions found.

The three cases are committed as `todo` tests, so the suite stays green
until steps 2–3 make them pass.

## Old behavior (agdevworld `3c8b438`)

**Idle child, approved after its attempt ended** (the incident):

| Boundary | seq | Round | Cleanup | p1 | Ready | Jobs |
| --- | --- | --- | --- | --- | --- | --- |
| attempt ended | 10 | awaiting | — | available | — | none |
| review started | 12 | reviewing (`reused`) | — | available | — | none |
| approved | 14 | complete | done, `left` | available | cleanup | none |

- The cleanup result was `left`: "serving before this review, for
  kid/make/run-001, which is still under way; that run stops it". That
  run is the review's own run. The page process was still alive and the
  record still said `available`.
- `left` resolved the cleanup, so the review was complete and `cleanup`
  ready. No job was queued, and `reconcile` queued none
  ("repaired from the durable run facts: 0 !== 1"). Only an outside
  resume (in production, recovery-agent's `run.recover`) would continue it.

**Approval while the attempt is still open** (end not recorded yet):
the same `left`, then `finished` recorded the end as `interrupted` with
hold cause `left-ready` (`cleanup` ready and nothing waiting). The attempt
had no chance to see work unlocked during it, and nothing queued its
successor. The run needed a person to resume it.

**Lost queue write and restart:** with the job removed and the stores
reopened, `reconcile` queued nothing: `hostOwes` is false after the
cleanup is `done`, and no reconcile path covers an approved, completed
review with ready successors.

These match the confirmed cause in the plan: `ReviewHost.cleanup` treats
the review's own unfinished run as another owner, and only requested
changes have a continuation (`decide`, `afterReview`, `finished`,
`reconcile`).
