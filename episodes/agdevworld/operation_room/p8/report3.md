# Step 3 — the cost of a second window, and two things the plan had wrong

Same rig (`.local/p8/step3/run.mjs`, `:8090`). The relay logs no requests
(`log_message` is silenced), so requests were counted on the browser side
with Resource Timing, per window, filtered to `:8094`; `/ops` `health` was
read before and after for the Zulip side.

## Two dashboards for ten minutes

Window A on `/`, window B on `/?routine=ghtrends`, 10.0 minutes:

| route | A | B | per minute per window |
|---|---|---|---|
| `/routines` | 120 | 120 | 12 |
| `/ops` | 120 | 120 | 12 |
| `/routines/<name>` | 120 | 120 | 12 |
| `/inflight/<name>` | 40 | 40 | 4 |

Exactly the design: three reads per 5 s tick, one host-directory read per
15 s. `sweeps` / `sweep_calls` / `last_sweep_at` were 1 / 63 / 06:29Z before
the window and identical after it — **the second window cost the realm
nothing.** Two windows are 2 × 40 loopback requests a minute against an
in-memory relay; a third or fourth would be the same again.

## The agentroom view's cache

`/agents` timed from Node: 184 ms cold (the `#agents` sweep, 7 Zulip calls),
then 4 / 4 / 6 ms across two reloads of `?view=agentroom` inside 30 s, and
87 ms again after 31 s. The 30 s cache (`AGENTROOM_CACHE_SECONDS`) answers
the second reload; this is the only route that reads Zulip on request, and a
second window on it doubles nothing inside the window.

## The Phaser views do not move

`?view=ops` and `?view=routines` each opened in B, a ✔ / un-✔ made on the run
topic, sixty seconds waited: **zero requests to the relay** from either view
in that minute (`relay_requests_in_60s: {"/ops": 0}` and `{"/routines": 0}`).
A reload showed the change. The screenshots (`03-*-after-60s`, `04-*-reloaded`)
look alike because the `routines` cards do not show run resolution at all —
the request count is the evidence. Side note for later: at 960 px the Phaser
title bar overflows the canvas; the dashboard does not.

## What the plan had wrong — and what it cost

**Un-resolving a run topic buys a Front run.** Zulip posts a Notification Bot
line ("Developer has marked this topic as unresolved") into the topic; that is
a new message in an unresolved `#front` topic, and Front's listener serves it.
Rapid toggles did not show this because Zulip withholds the notification when
a resolve is undone within its 60 s grace period, which is why Step 1's eleven
toggles at 5 s spacing were free and the plan's premise looked true. The three
un-✔s made more than a minute after a ✔ were not free:

| Front run | when (UTC) | topic | USD |
|---|---|---|---|
| run-0557 | 15:04:51 | `…11:02Z` (Step 1, first un-✔) | 0.091 |
| run-0558 | 15:10:39 | `…11:02Z` (Step 1 rerun, un-✔) | 0.070 |
| run-0559 | 15:14:35 | `…15:14Z` — **the authorised run** | 0.057 |
| run-0560 | 15:32:27 | `…15:14Z` (Step 3, un-✔ under `?view=routines`) | 0.158 |

Three unauthorised runs, USD 0.32, all on `ghtrends` run topics, all
answered by Front with a one-line "already confirmed / no new instruction".
The plan's Minimal constraints said one paid run; this step reports four.
The rule for anyone repeating this: **✔ is free, un-✔ is not** — measure with
resolves only, or un-resolve inside 60 s of the ✔.

**Front posts into the bare topic after its ✔.** run-0560 ✔'d the topic at
15:32:54 (message 5165, by user 15) and two seconds later posted its report,
message 5166, with the *bare* subject `front-routine-ghtrends-2026-09-07T15:14Z`
— Zulip then held two topics of that name, one ✔ with eight posts and one
open with Front's post. The relay keys by bare name and reads resolution
from the newest post's subject, so it said `open` with the evidence "the run
topic is not resolved" while the ✔ topic sat beside it. That is not a relay
bug: there *was* an unresolved topic of that name, and the board exists to
say so. It is a Front behaviour — report, then ✔, not the reverse — and the
second handoff candidate of this phase for the Front agent (the first, in
report2, is "say ✔ only after doing it"). Resolving message 5166 by hand at
15:38:24 folded the stray topic into the ✔ one; the relay read `resolved`
six seconds later, no run followed (a ✔ notification lands in a resolved
topic, which Front does not sweep).

## Files

`agdevworld/.local/p8/step3/` — `step3.json`, screenshots `01-ten-minutes-*`,
`02-agentroom-B`, `03-ops-after-60s-B`, `04-ops-reloaded-B`,
`03-routines-after-60s-B`, `04-routines-reloaded-B`; `.local/p8/zq.py`
(read-only Zulip lookups used above). The `ghtrends` run of 15:14Z is ✔ in
Zulip and `resolved` on the relay.
