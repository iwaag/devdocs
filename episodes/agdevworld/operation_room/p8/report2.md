# Step 2 — the real run, started from window A

Same rig as Step 1 (two visible 960×1040 windows, `:8090`, CDP). Window A on
`/`, window B on `/?routine=ghtrends`, both with default preferences. One
paid Front run, as authorised. Instruction typed in A:

> p8 two-window check: this run is a verification of the operation room
> only. Answer here with one line and dispatch no work.

## What happened

| | wall clock (UTC) | from the click |
|---|---|---|
| A's Start clicked | 15:14:35.27 | 0 |
| fire 5158 in `#front › front-routine-ghtrends-2026-09-07T15:14Z` | 15:14:35 | < 1 s |
| Front's ack 5159 | 15:14:35 | < 1 s |
| Front's answer 5160 | 15:14:43 | 8 s |

Front did as told: one line, no `workplan-` opened, so the run's graph is
the run topic alone (1 node). Fire, ack and answer times are the Zulip
message timestamps read back from the relay (`/routines/ghtrends`).

Both windows agreed at the end (`06-end-*.png`): the same run selected in
each, the same three posts (fire · ack · Front), the same `○ open` chip,
the same one-node graph, health lines a tick apart. B had never been
touched.

## What was not timed, and why

The number this step was built for — A's "Fire #… posted" to B listing the
run — was lost. The driver waited for A's result line to read `Fire #… posted`
and never saw it in 30 s of 250 ms polls, while the fire was already in Zulip
and on the relay. The likely mechanism is the one Step 1 measured: the relay
learns of the post from the event queue *before* the HTTP reply reaches A, so
A's very next refresh finds the run listed, clears the pending card and
replaces the result line with "… is listed below" within the same few hundred
milliseconds. The driver only logged what it was waiting for, not the line's
final text, so this stays an inference. The fire was not repeated — a second
paid run to recover a latency number that Step 1 already bounds (B trails the
relay by one 5 s tick, 1.6–3.7 s observed) was not worth it.

A second pass (`watch.mjs`) opened both windows fresh at +96 s and confirmed
the end state above; it could not see the ack and answer arrive because Front
had finished in 8 s.

## Observations worth keeping

- **Host observation missed the run entirely.** B's in-flight line read
  `0 in flight · 0 unknown · 1 without an observed run` on every 15 s poll
  from +96 s on. An 8 s Front run is shorter than the poll interval, as the
  plan anticipated; but "without an observed run" also says the host
  directories held no record keyed to this topic at all by the time we
  looked, which is a question for `/inflight`, not for this phase.
- **Front said it resolved the topic and did not.** Its answer ends
  "resolving this topic (✔) as the operation-room check", yet message 5158
  still carries the bare topic name in Zulip and the relay reads `open` with
  the evidence "the run topic is not resolved". The relay is right; Front's
  sentence is not. Handoff candidate for the Front agent: say ✔ only after
  the rename succeeded. The run is left open for Step 3's toggles and ✔'d
  by hand at the end of that step.
- The relay's `sweeps` / `sweep_calls` stayed at 1 / 63 across the run: a
  new run topic costs the event queue, not a sweep.

## Files

`agdevworld/.local/p8/step2/` — `run.mjs` (the start), `watch.mjs`,
`step2.json`, `step2-watch.json`, screenshots `01-before-*`, `03-B-lists-run-B`,
`04-B-ack-B`, `05-B-answer-B`, `06-end-*`.
