# lighter_agag_listen p1 — Step 4 report: quota raised, rolled out, latch proven gone

AI-generated (Omni Agent, 2026-08-17).

## The quota raise

`UserProfile.rate_limits` is a `"seconds:count,…"` string; empty means the
server default (200/60s here). All five listener bots were `''` and are now
`60:1000`. The command, so it survives a Zulip rebuild — the value is in the
database, but the knowledge should not live only in one head:

```sh
cd pj-agdev/.local/zulip-selfhost
docker compose exec -T -u zulip \
  -e BOTS=forge-bot@agstudio.local -e RATE_LIMITS=60:1000 \
  zulip /home/zulip/deployments/current/manage.py shell \
  < .../p1/.local/set_bot_rate_limits.py
```

(The repo's `./manage.py` wrapper does **not** forward environment into the
container — `docker compose exec -e` is required. The script defaults to all
five bots when `BOTS` is unset, and prints every bot's current value first.)

Verified from the client side, not just the database: `x-ratelimit-limit` now
reads **1000** for forge, autolab-agstudio, autolab-agautolab1, front and
cagent.

The plan asked to *check* whether autolab's two nodes share one bot. **They do
not** — `autolab-agstudio-bot` (id 11) and `autolab-agautolab1-bot` (id 12)
are separate Zulip users with separate credentials. So five bots, not four.

## The rollout

pyagag `3b289fc` (steps 1–3). All four consumers pin
`git+https://github.com/iwaag/pyagag.git?branch=main`, so each was bumped with
`uv lock --upgrade-package pyagag && uv sync` and the lock committed and
pushed to GitHub — never the gitea mirror.

| repo | commit | listener |
|---|---|---|
| agforge | `d76994f` | `com.agdev.agforge-zulip` restarted |
| agautolab | `2c3b7a8` | `com.agdev.agautolab-zulip` restarted |
| agfront | `1578487` | `com.agdev.agfront-zulip` restarted |
| pj-clusterintent (cagent) | `b8bd8d1` | `com.clusterintent.cagent-zulip` restarted |
| pj-agdev (submodule pointers) | `80982da` | — |

The `agautolab1` node was updated through its playbook
(`nctl render production` → `setup_autolab_node.yml --limit agautolab1`,
20 ok / 3 changed / 0 failed) and now holds agautolab `2c3b7a8` with pyagag
`3b289fc`. Its Zulip listener tasks **skipped**, correctly:
`autolab_node_zulip_listener` is `false` in the rendered inventory and
`autolab-zulip.service` is inactive and disabled on the node. See the finding
below.

No consumer handler code was edited, as the plan required — `sweep_serve`'s
signature never changed.

## Latch proof, live

forge's quota was temporarily set to **`60:10`** rather than the plan's
`60:30`: after step 3 a forge sweep costs 14 calls, not 30, so 30 would no
longer have been "below one sweep". The listener was then restarted.

The whole log from that restart (`.local/latch-proof-agforge.log`):

```
06:00:19 agforge zulip listener starting (pull sweep, prefixes ('runcreate-', 'create-') + DM thread)
06:00:19 listening as user_id=13 (forge-bot@agstudio.local)
06:00:19 sweeping as user_id=13 (forge-bot@agstudio.local)
06:00:19 registered event queue a94e6bb6-… (last_event_id=-1)
06:00:19 registered event queue c9841fcb-… (last_event_id=-1)
06:00:19 full sweep deferred: 6 requests left in the window
06:01:10 full sweep deferred: 4 requests left in the window
06:01:13 rate limited: GET events -> HTTP 429: API usage exceeded rate limit; … (retry after 2s); backing off 5.2s (strike 1, queue kept)
06:01:59 full sweep deferred: 0 requests left in the window
06:02:49 full sweep: 1 awaiting, 14 calls spent, 989 left in the window
06:02:50 serving 'general'/'create-20260817-p2-asset-1'
06:02:50 create topic 'general'/'create-20260817-p2-asset-1'
```

Against the plan's expectations:

- **one 429 → one logged backoff honouring `Retry-After`.** Zulip asked for
  2s; the backoff was 5.2s — the `RETRY_SECONDS` floor plus jitter, which is
  the designed behaviour, never shorter than the server asked.
- **zero `registered event queue` lines during the backoff.** Exactly **2**
  registrations exist in the entire run, both at startup, one per loop. The
  old code produced ~200 per minute in this state.
- **recovery and a served topic afterwards.** Quota restored to `60:1000`; the
  next poll refreshed the budget, the deferred sweep ran for exactly the 14
  calls predicted, and it served
  `general/create-20260817-p2-asset-1` — **the very topic `problem.md` opens
  with, unanswered for 10 hours.**

An unplanned result worth more than the planned one: at `60:10` — a quota it
*cannot* complete a sweep within — forge did not 429 at all for the first
minute. `SWEEP_BUDGET_RESERVE` saw 6 requests left and declined to start.
The 429 above had to be **forced** by burning the window with an external
burst of 11 calls as forge-bot. The loop no longer walks into the limit on its
own; it had to be pushed.

## Steady-state proof

Six messages posted into `general/mission-p1-burst-test`, served by
autolab-agstudio (`.local/burst-proof-agautolab.log`):

- **zero full sweeps.** Not one `full sweep:` line during the burst — the
  thing the old loop did 6 times, at 38 calls each.
- **4 targeted checks for 6 messages.** Not the 1 the plan hoped for, and the
  honest reason is worth recording: coalescing happens *per poll batch*. A
  long poll returns as soon as the first event lands, so a burst spread over
  several poll round-trips costs one check per batch, not one per burst. Six
  messages arrived as four batches. Full coalescing needs the messages to land
  inside one batch; the set guarantees *at most* one check per topic per
  batch, which is the bound that matters.
- **`x-ratelimit-remaining` stayed at the ceiling** — 999 of 1000 for every
  bot afterwards.

Old cost for that burst: 6 × 38 = 228 calls, over the *old* 200/min quota on
its own. New cost: 4 calls.

## Where the numbers ended up

| bot | subs before | after | sweep before | sweep after | quota before | after |
|---|---|---|---|---|---|---|
| forge | 21 | 5 | 30 | **14** | 200 | 1000 |
| autolab-agstudio | 17 | 4 | 38 | **14** | 200 | 1000 |
| cagent | 17 | 4 | 20 | **7** | 200 | 1000 |
| autolab-agautolab1 | 3 | 3 | 11 | 11 | 200 | 1000 |
| front | 2 | 2 | 10 | 10 | 200 | 1000 |

And the sweep is no longer what a message costs: it runs at startup and queue
re-registration only.

## Findings this step surfaced

**agautolab1 runs no Zulip listener, and its bot has a backlog.**
`autolab-agautolab1-bot` (id 12) has **7 awaiting topics in `general`**
(`run-game-idea`, `run-20260817-advance-work`, `run-1`, `run-2`,
`run-assetpipe1`, `create-20260817-p2-asset-1`, `mission-stray-in-general`),
and nothing serves them: the node's listener is disabled by desired state.
Either the desired state is right and that bot should not be addressable, or
the listener should be on. This is not a rate-limit problem and was not
touched here.

**`general` collects stray topics that match agents' prefixes.** Several of
the awaiting items above are strays in `general` that autolab answers with
`ignoring …: 'general' is not a project channel` — served, declined, and left
awaiting forever, so they are re-served on every restart sweep. Cheap now, but
it is permanent noise in every sweep. The burst-test topic was resolved
afterwards to avoid adding to it.

**The problem was live while this phase ran.** forge had accumulated **7,125**
rate-limit failures and was still re-registering ~200×/minute when step 3
began; its listener was stopped there and only came back here, on the fixed
code.

## What the phase leaves

The plan's three goals:

1. **A rate-limited listener backs off honouring `Retry-After`, keeps its
   queue, and recovers on its own** — proven live above, and proven with the
   *old* cost structure in the sense the plan meant: the quota was lowered
   below one sweep, which is the same stress at a smaller scale.
2. **One incoming message costs ~1 API call, not a full sweep** — proven: 4
   calls for a 6-message burst, zero sweeps.
3. **The quota raise is headroom, not the fix** — the latch proof ran at
   `60:10`, a twentieth of the old quota, and held.

Out of scope and still open, per the plan: the stale-`agag-status.json` reader
(problem.md item 6), merging the DM and sweep loops, `update_message` as a
sweep trigger, and a recurring unsubscribe janitor. Plus the two findings
above.

Restarted and reconfigured four in-system agents' listeners on their behalf —
handoff candidate.
