# p17 step 2 — presentations reopenable and owned

Plan: [plan.md](plan.md), section 2. Code: agdevworld `3fa432f`, `408e0dd`
(`workflow_editor/server/reviews.ts`, `server/present.ts`,
`server/exec.ts`, `server/api.ts`, `cli/review.ts`).

## What the implementation already gave, and what it did not

- **Already there:**
  - immutable bundles (`presentations/<p>/`, manifest digest), with hashes
    checked at every start;
  - instance identity: id, token, pid with start signature, and command
    line;
  - one start per presentation (`reused`);
  - verified stops.
- **Missing:**
  - `Presenter.start` refused every completed run ("its presentations are
    not served again"), so a child that had completed could not be
    reviewed later;
  - nothing recorded which review used an instance.
- **Remote execution:** a remote worker's `wfe present register` is
  forwarded to the coordinator and runs in the record copy, so the bundle
  is on the coordinator either way. Starting it needs no remote routing.
  The deployed check is in step 6.
- **Retention:** WFE never deletes run copies (storage.md *Lifetime*;
  there is no prune command) and integration leaves the record copy in
  place. A referenced bundle therefore stays readable. A missing or altered
  one is a visible failure, never replaced.

## What was added

- `Presenter.start(…, {review})`:
  - a human review round may serve a completed run's presentation again,
    the same frozen bundle;
  - the start is recorded on the owning run as `available`, and its stop
    as `stopped`;
  - a cancelled run is refused;
  - without a review, a completed run is still refused, now with "served
    again only for a human review".
- `present.sqlite` v2:
  - `usages` (instance, review uid, subject, `started`/`reused`, active or
    released);
  - `review_work`, a host operation claimed by a living process, so a
    reconcile never runs a start or a cleanup twice at once.
  - `wfe present status` and the presentation view name the rounds that use
    an instance.
- `ReviewHost` (`server/reviews.ts`):
  - **prepare** resolves `p1`, `d1[:p2]` or `<ws>:<wf>/<run>:p1`. It freezes
    the owning run's creation hash, the presentation and its revision, the
    delegated result (event, manifest, child seq) and the trees of the
    owning copy's repositories. It checks every bundle file on disk against
    the manifest and refuses a cancelled owner or a changed bundle.
  - **start** records the request first, then starts each subject through
    its owning run. A start already in progress is waited for, a serving
    instance is reused (never a duplicate), and the creation and revision
    are checked again. It records one `review.serve`. The service answers
    the click once the request is recorded (`wait: false`).
  - **decide**:
    - approval re-checks every bundle against its manifest, records the
      decision and cleans up;
    - changes hand the started instances to the responsible agent.
  - **cleanup**:
    - an instance another active round uses is `kept`;
    - an instance kept on request (`presentation.retain`) is `kept`, with
      its owner;
    - a reused instance that serves its own unfinished run is `left`;
    - everything else is stopped and verified through the owning run.
    - A failed stop is recorded and retried after 60 s, and never asks
      again.
    - The approval of a round also stops what earlier rounds handed to the
      agent and nobody stopped.
  - **reconcile** (`wfe exec reconcile`, whatever the run's executor or
    hold) continues a round left `starting` and retries owed cleanups. Run
    cancellation (`ExecStore.cancel`) cleans up its rounds.
- `wfe review prepare|list|show|start|approve|request-changes|cleanup|take-up`
  with `help` per subcommand. Synthetic actors use `--synthetic`; relayed
  words use `--because`.
- HTTP: `GET runs/<wf>/<run>/reviews[/<rv>]`, `POST …/reviews/<rv>/start|decide|cleanup`.
  The generic `ops` route refuses `review.*`, so nobody can post host facts.
- Brief and continuation:
  - a ready review node: how to prepare it, and to end the attempt;
  - rounds;
  - requested changes in full, with handed instances.
  - The executor continuation `review:<ws>/<wf>/<run>/<rv>` is queued after
    an attempt and by reconcile.
- Fixed in passing: `--no-change` was not a CLI flag, so `wfe run
  correction <run> --no-change --note …` (fix6) took `--note` as its
  value. It is a flag now.

## Checks

- `test/reviewHost.test.ts`, 4 tests that serve real bundles:
  - a completed child's stopped page is served again from the frozen
    bundle, while the worktree file it came from was changed;
  - a second Start review reuses the one instance;
  - approval stops it, records `stopped` on the child, and completes the
    round, the node and the run;
  - an old browser's second decision is refused;
  - a corrupted bundle gives `start-failed` with the server's refusal, no
    instance, and no approval; after repair the same round starts, and a
    changed bundle is refused at preparation;
  - two rounds use one page: the first cleanup keeps it, the second stops
    it, and a start left `starting` is finished by reconcile, reusing the
    instance;
  - a page kept on request is kept, with its owner.
- Full WFE suite: 582/582.

## Remaining for later steps

- Agent guidance and the catalog template.
- Assessment and reminders recognising a pending review.
- The follow-up of a satisfied delegation (step 3).
- The views and the conversation entry point (step 4).
