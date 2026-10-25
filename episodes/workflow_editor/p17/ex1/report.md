# Workflow editor p17 ex1 — report

Plan: [plan.md](plan.md). Steps: [1 reproduce](report1.md) ·
[2 ownership](report2.md) · [3 continuation](report3.md) ·
[4 verification](report4.md) · [5 deployment](report5.md).
Follows [p17 report7](../report7.md).

## What the Developer can do now

Open `http://agstudio.local:8093/wfe/#/ws/agops/review/release-check/run-006/rv1`,
press **Start review**, then **Approve**. The page should be stopped,
and the run's agent should do its cleanup and final report and complete
the run with no further action. Nothing in the waterfall session was
changed.

## Root cause

In conversation `20261025-013759-jz8ebz`, gimmickimg/freework/run-013
(`present → review → cleanup`) reused its own serving page for rv1.

1. `ReviewHost.cleanup` treated every reused page of an unfinished run as
   another owner's. It recorded `left` ("that run stops it") for the
   review's own run. `left` resolved the cleanup, so the round completed
   while the page still served.
2. Only requested changes had a continuation. Once the cleanup was `done`,
   nothing queued the `cleanup` node the approval had made ready, and no
   reconcile path looked for it. Recovery-agent resumed the run 4.5
   minutes later.

Step 1 reproduced both, plus a third: an approval during an open attempt
held that attempt `interrupted/left-ready`.

## Implemented contract

**Ownership** (`server/reviews.ts`). Cleanup decides by who owns the page,
not by whether the review reused it:

- kept for another active round or a keep request (owner named);
- `left` only for a page serving a **different** unfinished run (owner
  named);
- otherwise stopped and verified through the Presenter, recorded on the
  owning run. This includes the review's own run's page.
- A failed stop keeps the approval, retries, and releases nothing.
- Kept and left pages never read as stopped: the round text, the review
  page, `wfe review show` and the view (`handed`/`kept` with `owner`) all
  say so.
- Explicit cleanup nodes stay ordinary work. `wfe present stop` on an
  already-stopped page says so and succeeds.

**Continuation** (`shared/run.ts`, `server/exec.ts`):

- The cleanup entry that completes an approved round (`release: true`)
  records `release {seq, during, unlocked, continued}`. `unlocked` holds
  the review node's successors made ready.
- The first `attempt.begin` afterwards consumes it, and normal
  interruption rules apply to that attempt. An attempt open at the
  approval is not blamed for the unlocked work at its end.
- `afterRelease` is one eligibility rule:
  - the run is not executing, held, cancelled or completed;
  - the release is unconsumed and its work still ready.

  It queues one ordinary resume, receipt `review-approved:<ws>/<round
  uid>@<seq>`, on the run's assigned node.
- It is delivered by:
  - the cleanup's callback (browser, CLI, reconcile retries);
  - the end of an attempt that was open meanwhile;
  - reconcile, which re-reads the run after its review host work.
- A final review launches no agent turn. Holds wait for their resume.
  Requested changes keep their own `review:` continuation.
- Older entries replay unchanged.
- Facts reach the agent through its brief ("Ready now because of it: …",
  what the review stopped). Agents get no new instruction, and the wfrun
  guide is unchanged.

Contract: agdevworld `workflow_editor/docs/reviews.md` *After approval*.

## Checks

- New tests:
  - `reviewRelease` (6, end to end through executor, notifier, intake,
    CLI and HTTP);
  - `reviewReleaseState` (5, reducer);
  - `reviewHost` (+2: a different run's page, own page with stop
    failure);
  - `reviewFlow` (final review queues nothing).
- WFE `npm run check`: 600/600, `tsc`, both builds. The release check
  repeated 600/600.
- Deployed: see the table in [report5](report5.md). In brief:
  - ordinary path through the real review page: continued 5 s after
    approval, page reachable from agpc while serving and gone after;
  - lost callback + executor restart: repaired by reconcile within 2 s;
  - approval inside a paused open attempt: normal end, one later
    attempt;
  - no `run.recover` and no recovery run in any of them.

## References

- agdevworld `05ba800` (fixture), `93d183b` (fix), `ae7d050` (route
  tests) = release `ae7d050` (previous `3c8b438`). pj-agops `10f0826`
  (`release-check`).
- agpc pins wfe `ae7d050`, executor `cb83634`.
- Deployed runs: agops release-check/run-001 (ordinary), run-002
  (restart repair), run-003 (idle path, race missed), run-004 (race),
  run-005 (cancelled, misattributed), run-006 (waiting example). Event
  references are in report5 and the notes.
- Recovery and rollback (code and records together): ignored
  `pj-agdev/.local/p17-ex1/notes.md`; host notes in
  `pj-agdev/.local/devenv.md`.

## Limitations

- **One autonomous agent per condition, a trivial page.** The race was
  forced by pausing the agent: the natural gap between prepare and end
  was ~3–5 s, and one attempt to hit it missed (run-003).
- **`release` covers direct successors only.** A review whose successor
  was already started by an open attempt before the approval is that
  attempt's work, as before.
- **A job dropped at claim is not re-queued.** This is the existing queue
  semantics: the receipt finds the dropped job. Holds are the common case
  and are covered by the resume.
- **Integration of a run that review cleanup completes on the host**
  (final review, no attempt) is unchanged by this episode.
- **Rolling back to `3c8b438`** requires restoring the release-check
  records, or leaving them unread.
