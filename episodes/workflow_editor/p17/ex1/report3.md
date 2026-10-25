# p17 ex1 step 3 — work unlocked by approval continues durably

Plan: [plan.md](plan.md), step 3. Code: agdevworld `93d183b`.

## Durable evidence: `release`

The `review.cleanup` entry that completes an approved round now carries
`release: true` (written by the host). The reducer then records:

```
reviews.<rv>.release = { seq, at, during, unlocked, continued }
```

- `seq`/`at`: the completing entry. The trigger is the cleanup that
  completes the review node, not the approval click: a pending or failed
  stop releases nothing.
- `unlocked`: the review node's successors that this completion made
  ready. Unrelated ready work is not part of it.
- `during`: the attempt open at that moment, if any.
- `continued`: set by the first `attempt.begin` after it (attempt and
  seq). That attempt owns the work; from then on the ordinary rules apply,
  including `left-ready` if it ends leaving the work.

Entries without the flag (all history before this release) replay exactly
as decided. A test replays an old-style history to the same hold.

`attempt.end`: ready nodes in `unlocked` of a release with
`during === this attempt` and not consumed are not counted as left by this
attempt. Its own ready work still holds it. A test shows `extra` held
while `cleanup` is not.

## One eligibility rule, three deliveries

`ExecStore.afterRelease(ws, ref, rv)` reads the run fresh and queues an
ordinary `resume` only when:

- the run's executor is this host;
- the run is not cancelled, not held and not completed;
- no attempt is open (an open attempt is reassessed when it ends; nothing
  runs beside it);
- the round is `complete` with an unconsumed release, and one of its
  unlocked nodes is still ready.

Receipt: `review-approved:<ws>/<round uid>@<release seq>`. The uid carries
the run's creation hash, so a later round or a re-created run never reuses
a receipt. `enqueue`'s receipt lookup and one-open-job-per-run transaction
serialize concurrent callers (service and executor processes share the
store). The job carries `nodeOf(run)`, so a run assigned to a remote worker
is claimed there; the coordinator's own worker stays idle for it.

Deliveries:

1. **Normal**: `ReviewHost.cleanup` calls `released` when its entry
   completed the round. This covers the browser, the CLI and reconcile's
   own cleanup retries.
2. **Attempt end**: `finished()` runs `openReleases` after the existing
   continuations, so approval during an attempt queues one later attempt.
3. **Reconcile**: after `reviews.reconcileRun`, the run is **read again**
   before anything is scheduled from it. Then `openReleases`. This repairs a
   missed callback or a lost queue write after a restart.

A final review with no successors records `unlocked: []`; the run
completes on the host and no agent turn is launched. Publication and
integration behavior for host-driven completion is unchanged. Holds stay
holds: a resume's attempt consumes the release, and nothing else is
queued. Cancellation queues nothing. Requested changes keep their own
`review:` continuation; they release nothing.

## What the continuing attempt is told

- Brief (`reviewBrief`), for a complete round whose unlocked work is
  still ready:
  - who approved it, when, and the feedback;
  - "Ready now because of it: cleanup. Do that work as usual.";
  - what WFE's cleanup already stopped and verified (stopping again
    reports nothing to stop), or what it left serving and for whom;
  - "The approval asks for nothing more of the person."
- `wfe run state` / continuation state line: "Review rv1: approved by …;
  cleanup ready after it."

These are facts in the brief, not a new rule for agents. The wfrun guide
is unchanged.

## Tests (`reviewRelease.test.ts`, `reviewReleaseState.test.ts`)

- **Incident shape, idle**:
  - approval → page stopped, `cleanup` ready, exactly one job
    `review-approved:kid/kid/make/run-001#…/rv1@15`, the same after
    reconcile;
  - the claimed attempt runs cleanup and the child completes in 2
    attempts with no `run.recover`; reconcile then queues nothing;
  - the parent receives the completed result.
- **Approval during an open attempt** (end not yet recorded): nothing
  queued beside it, also not by reconcile. The end is normal (no hold);
  `finished` returns the continuation; one later attempt completes it.
- **Lost queue write + restart**: the job is deleted and the stores
  reopened; reconcile queues one job, and again finds the same one.
- **Holds/cancel/duplicates**:
  - approval while a stop is pending: a held `stopped` run gets no job
    from the callback or reconcile;
  - `run.resume` gives one ordinary job, and its attempt consumes the
    release (brief shows "Ready now because of it: cleanup");
  - a late duplicate callback queues nothing. That attempt leaving the
    work is held `interrupted` as usual;
  - after cancellation nothing is queued.
- **Remote worker**: assigned to `pc`, the job's worker is `pc`. A
  duplicate callback returns the same job, and the coordinator's own claim
  is idle.
- **Multiple rounds**: rv1 changes requested releases nothing; rv2
  approved releases its own successor.
