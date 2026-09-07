# Operation room p8 — two windows, one session

Prove, against the running system rather than from the code, that a routine
session started in one browser window is reflected in an operation room open
in another window — and measure how quickly. The finding decides whether the
"instruction scene in one window, monitoring scene in another, on another
display" arrangement can be adopted as it stands, and what, if anything, has
to be built for it.

This is a verification phase. It changes no product code unless a check
fails; the deliverables are the step reports, their numbers and screenshots.
Main ground: `pj-agdev/agdevworld` (dashboard on `:5173` or `:8090`, relay on
`:8094`), the Developer Zulip account for writes.

## Why

The code says the answer is yes: every state the dashboard shows comes from
the relay's memory (`/routines`, `/routines/<name>`, `/ops`) or from host
directories (`/inflight/<name>`); the browser keeps only two display
preferences in `localStorage`. The dashboard re-reads the relay every 5 s
(`operationDashboard.ts`, `tick()`), and the relay's event queue carries a
Zulip post back in a second or two (p7 step 4 measured ✔/un-✔ landing "within
seconds"). So a second window should trail the first by under ten seconds.

That reading has not been observed. Three things in it are only inferred:

- the end-to-end latency from "Start new session" in window A to the run
  card appearing in window B, which has never been timed;
- what is *not* shared — the routine selection, the pending-fire card and the
  "Fire #… posted" result line are per-window state — and whether window B
  still lands on the right session without them;
- the cost of a second (or third) poller: the relay serves these routes from
  memory, so more windows should mean more HTTP requests and zero extra Zulip
  calls, but nobody has counted.

And one thing is known to be *no*: the Phaser views `?view=ops` and
`?view=routines` have no timer — only the ⟳ chip (`views.ts`,
`PanelGridScene.ts`). A monitoring display opened on one of those would show
the state at load time forever.

## Decisions already made

- **Zero-cost first, one paid run second.** Latency and per-window state are
  measured with resolve/un-resolve on an existing run topic (a Zulip
  `update_message`, which buys nothing) before anything is fired. One paid
  Front run is authorised for the real end-to-end check, started **from the
  board** in window A — the path p7 did not exercise for real. More only if
  the first fails for a reason unrelated to the question.
- **Real windows, not one window with two tabs.** Chrome throttles timers in
  background tabs; two tabs would measure the throttle, not the design. Use
  two top-level windows, both visible (side by side, or on two displays if
  available). The existing CDP driver (`.local/opsshot.mjs`, p7's
  `.local/p7/step4/a.js`) can open two targets in one Chrome and capture
  both; a hand-driven pass with a wall clock is acceptable if the numbers are
  written down.
- **The dashboard `/` is the monitoring candidate.** The Phaser views are
  checked only to record that they do not refresh; making them refresh is a
  step of this phase only if the report needs it (Step 4).

## Step 1: per-window state and zero-cost latency

- Open window A `/` and window B `/?routine=ghtrends` (or whichever routine
  holds the newest open run). In B, confirm the routine is preselected from
  the URL and the newest run is selected without a click. In A, select a
  *different* routine; confirm B does not follow — the selection is per
  window, and the report says so as a fact rather than a suspicion.
- Latency, ten trials or so: ✔ the run topic with `agentchat resolve` (the
  Developer, `.local/zulip/developer.env`), note the wall-clock second of
  the command's return and the second the session chip in B reads
  `✔ resolved`; un-✔ and note the return to `○ open`. The chip is
  `RESOLUTION_MARK` in `operationDashboard.ts`; a CDP script can poll its
  text at 250 ms. Record min / median / max. Expected shape: a few seconds,
  bounded above by the 5 s poll plus the queue.
- Also record the "observed <time>" value in B's health line across the
  change, so the report can separate relay latency (`generated_at` moves)
  from browser latency (the next tick draws it).
- Preference isolation: toggle Show resolved and Compact/Detailed in A;
  confirm B keeps its own until reloaded, and that B picks A's values up on
  reload (same origin, same `localStorage`). Decide and write down whether
  that reload behaviour is fine for a monitoring window.

Report: `report1.md` with the timing table, the two screenshots, and the
list of what turned out per-window.

## Step 2: the real run, started from window A

- In window A, on a routine whose newest run is resolved or answered,
  `New session` → `Start new session` with a one-line instruction that makes
  the run cheap to recognise (e.g. "p8 two-window check; answer with one
  line"). `ghtrends` is the p7 precedent; `localtest` if it has been
  un-retired for this. One paid Front run.
- Window B, which has never been touched: time, from the moment A shows
  "Fire #… posted", until B lists `Run <stamp>` as the selected newest
  session and its chat pane shows the fire line as the first post. Then time
  Front's ack and answer appearing in B's chat, and the graph gaining the
  `workplan-` child if Front opens one. Capture B at each state.
- Confirm A and B agree at the end: same run selected, same posts, same
  resolution chip, same `observed` time within one tick. Confirm A's
  "pending" card resolved into the real card and that B never showed a
  pending card (it has no `pendingFire`).
- If Front answers in the same minute, note whether B's `Host observation`
  line showed `in flight` at any moment; it polls `/inflight` every 15 s,
  so a short run can slip between two polls. That is a known gap, not a
  failure — record it.

Report: `report2.md` with the timeline (fire → B lists it → ack → answer),
the run topic name, screenshots of both windows at the end.

## Step 3: the cost of a second window

- With A and B open for ten minutes on the same routine, count the relay's
  requests per route from `agentroom/.local/out/agentroom.log` (or a
  foreground `serve.sh` run with the launchd job stopped — the port is
  single-occupancy). Expected: ~2 × (12 `/routines` + 12 `/routines/<name>`
  + 12 `/ops` + 4 `/inflight`) per minute, and `sweep_calls` in `/ops`
  `health` unchanged across the window — i.e. no Zulip call attributable to
  the second browser. If `/ops` `sweeps` moved, find out why before writing
  the report.
- Open the `?view=agentroom` world view in a third window and reload it
  twice inside 30 s; confirm the relay's 30 s cache answers the second reload
  (one Zulip sweep, not two — `AGENTROOM_CACHE_SECONDS`). This is the one
  route that does read Zulip on request.
- Open `?view=ops` and `?view=routines` in a window, make a change (Step 1's
  ✔/un-✔ is enough), wait a minute, and screenshot: the report records that
  these views did **not** move until ⟳ was pressed. A fact for the readme, and
  the input to Step 4.

Report: `report3.md` with the request counts and the sweep-calls before/after.

## Step 4: verdict, and only what the verdict demands

- Write `report.md` for the phase: can the two-window arrangement be used
  today, with which URLs (`/` for instructions, `/?routine=<name>` for
  watching, `/?parts=gauge` beside it), with what latency, and with which
  caveats (per-window selection, the 15 s in-flight window, Phaser views
  static). Update `agdevworld/README_DEV.md` — a short "two windows" note
  under the dashboard section — and, if Step 3 measured anything worth
  keeping, the relay README's cost paragraph.
- Build only if a check failed or the report cannot recommend the
  arrangement without it. The candidates, in the order they would be worth
  it: (a) a `?mode=watch` (or similar) that hides the New session and Run
  conversation panes so the monitoring window cannot post; (b) a periodic
  reload for the Phaser `ops`/`routines` views, matching the dashboard's
  5 s and using the relay's memory only; (c) nothing about push — a relay
  SSE route replaces polling only if Step 1 shows the 5 s tick is what a
  human notices, and the plan's expectation is that it is not.
- Anything built gets the same treatment as p7: relay tests / `npm run
  build`, a CDP pass, `docker compose up --build -d web`, relay kickstart if
  touched, `nctl status` / `nctl drift`, commit/push.

## Minimal constraints

- One paid Front run for Step 2; every other measurement uses ✔/un-✔ or
  reads. No bot posts into a run topic to "generate activity" — a bot line
  in an agent topic buys a run and misdirects the agent.
- Two visible top-level windows for every latency number; a number taken
  from a background tab is not reported as the design's latency.
- No product change before Step 4, and none there without a failed check or
  a stated gap in the recommendation.
- Credentials and machine paths stay out of tracked files; screenshots and
  CDP scripts live under `agdevworld/.local/p8/`.
