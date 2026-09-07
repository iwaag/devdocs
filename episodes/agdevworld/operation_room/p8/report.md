# Operation room p8 — report

**Verdict: the two-window arrangement works today, without a build.** Open
`/` for instructions, `/?routine=<name>` in a second window (another display
is fine) for watching, `/?parts=gauge` in a third if wanted. Tick Show
resolved in the watching window. That is the whole recipe.

## What was proved on the real system

- **Everything shown is the relay's memory.** A change in Zulip reaches the
  relay before the HTTP reply of the request that made it reaches the caller
  (8–16 ms, i.e. the event queue is faster than the response path). Another
  window shows it on its next 5 s tick: 1.6–3.7 s observed over eleven
  toggles, 5 s plus a fetch as the worst case (report1).
- **One real run, started from window A** (the board's Start, the path p7
  had only mocked): fire, Front's ack in the same second, Front's one-line
  answer at +8 s; window B, never touched, ended on the same run with the
  same posts, chip and graph (report2).
- **A second window costs the realm nothing.** 40 loopback requests a
  minute per window, `sweep_calls` unchanged over ten minutes, the
  agentroom view's 30 s cache answering repeat reloads in 4–6 ms (report3).
- **Per-window, by design:** routine/session selection, the pending-fire
  card, and the display preferences until reload (shared `localStorage`,
  one copy per browser profile).

## Caveats the reader must know

- **Default preferences hide a ✔'d run**; the list says "resolved and
  hidden". A watching window needs Show resolved on, or it loses the run at
  the moment somebody closes it.
- **The Phaser views are static** — `?view=ops` and `?view=routines` made
  zero requests in a minute after load. The monitoring window is the
  dashboard, not a world view.
- **Host observation (`/inflight`) can miss a short run entirely**: an 8 s
  Front run fell between 15 s polls and the line read "without an observed
  run" throughout.
- The fire → B-lists-it latency was not captured as a number: A's "Fire #…
  posted" line is replaced within the same refresh because the relay is
  already ahead of the reply. Step 1's bound (one tick) stands in for it;
  a second paid run to recover the number was not taken.

## What went wrong

**Three unauthorised Front runs, USD 0.32.** The plan's premise that ✔ /
un-✔ "buys nothing" is false for un-✔ made more than 60 s after a ✔: Zulip
posts a Notification Bot line into the now-unresolved `#front` topic and
Front serves it. Rapid toggles inside Zulip's 60 s undo grace period post
nothing, which is why the eleven-toggle series was free and the mistake
looked like a confirmation. Runs 0557, 0558 (Step 1 setup, twice) and 0560
(Step 3) each answered with a one-line "nothing new". Recorded in report3
with the costs; the rule is now in `agdevworld/README_DEV.md`: **✔ is free,
un-✔ is not.**

## Handoff candidates (Front agent)

Found while looking at the dashboard, not sought: (1) Front's answer said
"resolving this topic (✔)" and did not ✔ — say it after doing it. (2) In the
run it did ✔, Front then posted its report into the *bare* topic name two
seconds later, leaving an open twin topic beside the ✔ one; the relay read
it, truthfully, as `open`. Report first, ✔ last. Both belong to `agfront`,
neither was changed here ("did X for agent Y": the stray topic was folded
into the ✔ one by hand as the Developer).

## Built

Nothing in the product. `agdevworld/README_DEV.md` gained a "Two windows"
subsection (pj-agdev `agdevworld` pointer bumped). The measurement rig —
`twowin.mjs` (two visible windows over CDP), `toggle.py`, `zq.py`, the step
scripts and screenshots — lives under `agdevworld/.local/p8/`, ignored.

Deferred, with the reason: a `?mode=watch` that hides the posting panes and
a URL parameter for Show resolved would make a monitoring window safer to
leave unattended, but no check failed for want of them and the two-line
recipe above is enough for one operator on one machine. Periodic refresh
for the Phaser views is not worth building while the dashboard is the
monitoring surface. Server push (SSE) is not warranted: nothing observed
exceeded 3.7 s and the human-visible delay is the tick, which is already
under the "does anyone notice" line this plan set.
