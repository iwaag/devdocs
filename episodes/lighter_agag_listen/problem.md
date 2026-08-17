# Problem — the sweep listener rate-limits itself and never recovers

Observed 2026-08-17 while reconciling `modernize_agdevworld` p2. The agforge
Zulip listener had been dead-but-alive for **10 hours**: the process was up,
launchd was happy, and it answered nothing. It never picked up the `create-`
topic p2's Front had just opened for it.

Every 5 seconds, since `2026-08-16T19:13:52Z`, both of its loops logged:

```
zulip call failed: GET users/me/<id>/topics -> HTTP 429: API usage exceeded
rate limit; …; retrying in 5s
registered event queue <new id> (last_event_id=-1)
```

6627 such failures in one log. This is not an agforge bug — it is the shape of
`agag.zulip.sweep_serve`, which every listener in the system shares.

## Measurements

| Fact | Value |
|---|---|
| Zulip per-user quota (`x-ratelimit-limit`) | 200 requests / 60 s |
| forge-bot's remaining budget while spinning | **7** |
| developer / front-bot remaining, same moment | 199 |
| One agforge sweep | `1 subscriptions + 21 topics + 8 topic_history` = **30 calls** |
| Retry interval on error | fixed 5 s, no backoff |
| Sustained demand once spinning | 30 calls / 5 s = **360/min against a 200/min budget** |
| Channels subscribed: forge / autolab / cagent / front | 21 / 17 / 17 / 2 |

## Root cause, in three layers

### 1. The sweep is expensive and is redone from scratch on every event

`sweep_topics` costs `1 + (subscribed channels) + (matching unresolved
topics)` API calls, and `sweep_serve` runs a whole one whenever a message
event sets `dirty`. There is no debounce, no coalescing, and no incremental
state. At 30 calls per sweep the quota affords **six sweeps a minute** — a
burst of seven messages is enough to cross it. Nothing in the loop knows what
a sweep costs.

### 2. The 429 handler is the thing that sustains the 429 (the latch)

`sweep_serve` treats every `ZulipError` alike: sleep a fixed 5 s, **drop and
re-register the event queue**, and set `dirty`, which restarts the full sweep
from the first channel. So the remedy for "you are calling too much" is to
call 360 times a minute against a 200-a-minute budget. The `Retry-After` and
`x-ratelimit-reset` headers Zulip sends are never read; a partly-completed
sweep is never resumed, so the budget is always burned on the first ~15
channels and the sweep never reaches the end. Once entered, this state is
stable forever — 10 hours of evidence.

The 5-second-retry-and-re-register behaviour is *correct* for what it was
built for: a dropped connection or a Zulip restart, proven in
`better_communication/zulip_receive/report4.md`. The defect is that errors are
not distinguished. For a transient disconnect, retrying hard is the cure; for
a rate limit, retrying hard **is the disease**.

A second copy of the same loop makes it worse: agforge runs the topic sweep
and the DM `serve` loop as the same bot user, so two loops re-register and
re-sweep against one shared budget.

### 3. Cost grows quietly with accumulated test channels

17 of forge's 21 subscriptions are dead `pj-*` channels from finished
experiments and hold no matching topic at all. Sweep cost is linear in
subscriptions, so **every experiment that leaves a channel behind pushes every
listener closer to the threshold**. Nothing ever unsubscribes.

## Why only agforge, and why that is not reassuring

front-bot sweeps 2 channels (~4 calls per sweep) and is far from the limit.
But **autolab and cagent sit at 17 channels each** — the same cost structure
as forge's 21. They are not safe; they simply have not been tipped over yet.
One busy minute puts them in the same latch. The user reports having seen a
similar problem before, which fits: this is a recurring failure mode of the
shared loop, not an incident local to one agent.

## Also missing: it looked healthy the whole time

launchd reported the service as running. The status file and the log recorded
every failure, and nobody read them. A listener that has answered nothing for
10 hours should be loud — the failure was found only because a human noticed
an unanswered request.

## What a fix should establish

1. **Rate limits are their own error class.** Honour `Retry-After` /
   `x-ratelimit-reset`, back off exponentially with jitter, and do **not**
   re-register the event queue — nothing is wrong with the queue.
2. **A sweep is debounced and coalesced.** Bursts of events collapse into one
   sweep; the first 429 stops happening.
3. **A sweep can resume.** A sweep interrupted part-way continues from where
   it stopped instead of re-charging the whole cost.
4. **Sweep cost is bounded and visible.** The loop should know, and log, what
   one sweep costs against the quota — before it spends it.
5. **Subscriptions are not immortal.** Finished experiment channels get
   unsubscribed, or sweeps stop paying for channels that never match.
6. **Silence is an alarm.** A listener stuck in a retry state must surface
   somewhere a human or an agent already looks, not only in its own log.

Items 1–3 remove the latch, which is what turned a passing rate limit into a
10-hour outage. Items 4–6 are what keep the next one short.
