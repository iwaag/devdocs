# p17 ex1 step 2 — cleanup resolved by actual ownership

Plan: [plan.md](plan.md), step 2. Code: agdevworld `93d183b` (with step
3; the shared record changes serve both).

## What decided it before

`ReviewHost.cleanup` left every `reused` instance alone when its owning
run was unfinished and its presentation open. It never asked whose run that
was. In run-013 the owning run was the review's own, so `left` meant
"you stop it" addressed to itself, while the round counted as cleaned up.

## Facts reviewed

- **Usages** (`present.sqlite` `usages`): which rounds use an instance,
  `started`/`reused`, active/released. Another active round's usage is
  already respected (`kept`).
- **Instance identity**: round subjects carry the owning run's workspace,
  workflow, run and creation hash. A round's own run is identified by the
  same tuple (`own(sub)`), so a re-created run with the same id is not
  mistaken for it.
- **Retention**: `presentation.retain` (`wfe present keep`) gives note and
  owner. It is already respected (`kept`).
- **Results**: `stopped`, `stop-failed`, `kept`, `left`. These are enough
  once `left` has its proper meaning, plus an optional `owner` naming who
  still serves the page. No store change was needed.

## Contract now

| Found at cleanup | Result | Page |
| --- | --- | --- |
| another undecided/handed round uses it | `kept`, owner = that round | serving |
| kept on request | `kept`, owner = who stops it | serving |
| reused, owned by a **different** unfinished, uncancelled run | `left`, owner = that run | serving: that run stops it |
| anything else, including the review's own run's page (started or reused) | stopped through the existing Presenter lifecycle, verified; `presentation.observe stopped` on the owning run | stopped |
| stop not verified | `stop-failed`; round stays `approved`, cleanup `failed`, retried by reconcile; nothing released | owed |

- A completed child's reused page is still stopped (existing test kept).
- **Honest display.** A `left` or `kept` page resolves the round's cleanup
  but never reads as stopped:
  - round text: "Approved · complete · a page still serving (for
    <owner>)" (`stillServing`);
  - review page subject: "Still serving for its run · <owner>" or "Kept
    serving"; the outcome card says "Not stopped by it: …" instead of
    "Cleanup verified";
  - `wfe review show`: a `still serving:` line;
  - view status: `handed`/`kept` with `owner`.
- **Explicit cleanup nodes stay ordinary work.** An already-stopped page
  makes `wfe present stop <run> p1` exit 0 with "p1 is already stopped
  (recorded by wfe-review at …: stopped by the cleanup of human review …);
  nothing to stop." `--all` returns nothing to stop.
- Help: `wfe review help approve|cleanup`; contract `docs/reviews.md`
  *After approval*.

Agents get no new instruction. The fix does not depend on whether a
workflow has a cleanup node.

## Tests

- `reviewRelease.test.ts`, incident shape: reused own page stopped
  (`['stopped']`), `p1` recorded `stopped`, process gone. The child's
  explicit cleanup then calls `stop all` successfully.
- `reviewHost.test.ts`:
  - **different unfinished run**: `left` with owner `p/show/run-…`,
    process alive, record `available`, view `handed`, text "still serving
    (for p/show/run-…)";
  - **own reused page, stop fails**: approval stands (`by: Pat`), round
    `approved`/cleanup `failed`, successor not ready, nothing released,
    process alive. A verified retry stops it, completes the round,
    releases `cleanup` once, and a repeated cleanup adds nothing;
  - the existing kept-on-request and two-rounds-on-one-page cases pass
    unchanged.
- `reviewReleaseState.test.ts`: `left` with owner reads as still serving.
