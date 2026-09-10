# refine_routine p2 — report

Date: 2026-09-10. Plan: [plan.md](plan.md). Braindump: [braindump.md](braindump.md).

## What was asked, and what ran

One real `study-realworld` run through Front's ordinary entrance, with the
execution profile chosen at request time and an end condition the run
observes itself:

- Front on its configured defaults (Claude Sonnet 5 through `claude_code`) —
  its `sonnet` profile is private, so nothing was selected and `default` is
  the published name for it.
- autolab on its published `agy` option (Antigravity CLI, Gemini 3.8 Flash,
  pool `antigravity`), reselected at every delegation.
- End when agy's **`Gemini Models` 5-hour** window reaches 30 % used — named
  by scope, because the `antigravity` pool carries two 5-hour windows and the
  pool alone does not pick one.
- One mission per delegation, Front repeating until the condition is met.
- Mode and sources at the executing agent's discretion.

Requested 01:45 UTC in `#front › front-20260910-refine-routine-p2` (5587),
run opened at 01:45 in `#routine-study-realworld ›
routinerun-20260910T0145Z` (5590), ended 02:18 UTC. About 33 minutes, four
missions.

## Verified

- **The profile reached every delegation.** Each of the four missions got its
  own `workplan-` topic with `agentchat use … agy --to "autolab-agstudio1"`
  posted *before* the request, and the option was reselected again on the
  `workrun-` task topic autolab opened (5683) — not only on the plan topics.
  Front's own conversations stayed on the default throughout, which is the
  distinction `runtime-profile` step4 was built for: asking Front to have
  autolab run on `agy` is not asking Front to run on `agy`.
- **A usage condition reached mid-run, live.** p1 could only prove a
  condition already met at startup. Here the window ran 0 % (01:46) → 26.9 %
  (02:10) → 28.3 % (02:12) → 35.6 % (02:17), and the run ended on the last
  read with nothing in flight. Every entry named the read, its time, and the
  window it judged.
- **`achieved` and `reason` stayed two fields.** The finish block says
  `achieved: true` for the routine's goal and gives the threshold as the
  `reason` the run ended — the conflation p1 warned about did not happen.
- **Delivery and closure.** The report landed in the requesting conversation
  (5695) with a link to the run, and the run topic was resolved.
- **The work is real.** Four commits pushed to `origin main` in the
  studyrealworld project — `a4ee489`, `1fc5849`, `78e9e10`, `8f517ed` —
  one investigation, one discovery (three new sources: `worldbank-documents`,
  `bls-news-releases`, `fao-news-feed`), then two second-data-point
  investigations. `main/sources/INDEX.md` now lists six verified sources,
  each with at least one report; the three original sources have two. A
  spot-read of `reports/fed-fomc-minutes/2026-09-10-001/report.md` is a cited
  report from retrieved document bodies, with publisher claims and
  interpretation kept apart, forward-linked from its predecessor.
- **Cost.** 32 runs on this host today, none failed: 20 `claude_code` runs at
  $4.07 reported (Front's servings), 12 `agy` runs on the subscription
  (22.7 M tokens in, 224 k out). Front's own pool afterwards: 5-hour 6 %,
  weekly all models 51 %, weekly Fable 77 %.

## Findings

- **The threshold overshoots, structurally.** One mission moves the window
  about 7 points, so a run that reads 28.3 % opens another mission and lands
  at 35.6 %. The guide already says reaching the condition is not a wall; the
  measured granularity says how wide the wall is not. A threshold worth
  hitting closely needs missions smaller than the tolerance, not a stricter
  rule.
- **agy's reset time is rolling until the window starts.** Two reads thirteen
  minutes apart, both at 0 % used, each answered `read_at + 5 h` exactly. The
  CLI reports the reset of a window that has not begun; once the first agy
  work landed it settled at 06:47 UTC. A reset time read before any usage
  says nothing, and a run that judged a horizon by it would be judging a
  moving one.
- **The run could not write its `threads/` files** and said so in its first
  entry, recording the thread information in the entry text instead. No
  visible harm — the listener supplies `threads/` from Front's own
  participation and every callback routed correctly — but the run noticing a
  permission it does not have is worth a look before it matters.
- **The guide's "one mission" line held only because the opening post
  overrode it.** `study-realworld`'s guide says to ask for one mission and see
  it through; the request said to repeat until the threshold, and the routine
  run guide's "where they disagree, the opening post wins" is what made four
  missions correct rather than three violations.

## Left

- Still unproven live: a hold and what resumes it, a hand-opened run, Front
  itself running on a non-default option, and the other three routines.
- Still absent: any way to stop a running run. This run ended on its own
  condition inside half an hour, which is not the same as being stoppable —
  developer interrupt and forced cancel remain the next phase.
