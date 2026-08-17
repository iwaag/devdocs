# lighter_agag_listen p1 — Step 3 report: subscription hygiene

AI-generated (Omni Agent, 2026-08-17).

## What changed

**`ZulipClient.unsubscribe_channels(names, principals=None)`** — `DELETE
users/me/subscriptions`, the counterpart of `subscribe_channels` and the only
way a listener's sweep cost has ever been able to go *down*. Same shape as its
twin: an empty list makes no request. Tested by
`test_unsubscribe_channels_sends_names_and_skips_an_empty_list`; the zulip
suite is 53 passing.

**The live pass** left every channel a finished experiment abandoned:

| bot | before | after | left behind |
|---|---|---|---|
| forge | **21** | **5** | FreeForge, general, ops, pj-assetpipe1, sandbox |
| autolab-agstudio | **17** | **4** | general, ops, pj-assetpipe1, sandbox |
| cagent | **17** | **4** | general, ops, pj-assetpipe1, sandbox |
| autolab-agautolab1 | 3 | 3 | nothing dead to remove |
| front | 2 | 2 | nothing dead to remove |

42 subscriptions dropped across three bots. What that does to the *startup*
sweep — the only place a full sweep still runs after step 2:

| bot | sweep cost before | after |
|---|---|---|
| forge | 30 calls | **14** |
| autolab-agstudio | 38 calls | **14** |
| cagent | 20 calls | **7** |

Against the unchanged 200/60s quota, forge's startup sweep went from 15% of
the window to 7%.

## How "dead" was decided, and the proof nothing was abandoned

A channel was treated as dead when its name is an experiment leftover
(`pj-*` or `create-<timestamp>`) and it is not in the keep set. The keep set is
the declared desired state — `nctl desired export`'s
`desired_agent.desired_zulip_channels`: `general`, `ops` for all,
plus `FreeForge` (forge) and `front` (front) — plus the two channels the
desired *minimum* does not name but which are live: `sandbox`, and
`pj-assetpipe1` (Assetpipe1 is a standing Plane project, and it holds real
awaiting work).

Names alone would not have been enough, so the decision was checked twice:

1. **Nothing in them is quiet-but-pending.** Every candidate's newest message
   is at least a day old, and several (`pj-flipproof`, `pj-p3smoke1`,
   `pj-phase1-subscribe-20260813`) have never held a message at all.
2. **Nothing in them was awaiting a reply.** A real `sweep_topics` run per bot
   — the last-poster rule included, not just prefix matching — was taken
   before and after. The two lists are **identical**:

   ```
   forge              1 awaiting: general/create-20260817-p2-asset-1
   autolab-agstudio   3 awaiting: general ×2, pj-assetpipe1 ×1
   autolab-agautolab1 7 awaiting: general ×7
   cagent             0 awaiting
   front              0 awaiting
   ```

   Every awaiting topic in the realm lives in `general` or `pj-assetpipe1`.
   Not one lived in a channel that was left.

Unsubscribing is reversible — `subscribe_channels` puts a bot back, and a
public channel's history is readable either way — so this needed evidence, not
a safety interlock.

The three scripts (`subscription_survey.py`, `awaiting.py`,
`unsubscribe_dead.py`) are under this phase's `.local/`. `devdocs` had no
`.gitignore`, so one was added with `.local/` per the AG standard style; no
`.local` file had been tracked before.

## Found while doing this

**The forge listener was still latched, live.** The first survey attempt died
on a real HTTP 429 within five channels, because forge-bot's quota was being
spent by its own spinning listener: **7,125** rate-limit failures in
`agforge/.local/out/zulip-listener.log`, still re-registering its event queue
about 200 times a minute. The problem.md diagnosis is not historical; it was
happening while this step ran.

So `com.agdev.agforge-zulip` was stopped (`launchctl bootout`) for the
duration of this step. It was answering nothing, so nothing was lost, and the
survey then ran with a full quota. **It is still down** and comes back in step
4, on the new pyagag.

**`general/create-20260817-p2-asset-1` is still unanswered** — the very topic
`problem.md` opens with. It is forge's one awaiting item and will be picked up
by the startup sweep the moment forge restarts on the fixed code. It is
therefore also the most honest end-to-end proof available in step 4.

## For the episode's closing

Experiment teardown should include unsubscribing the bots it subscribed.
Today's cleanup was 42 subscriptions across three bots, accumulated from
episodes that each left one channel behind — and, before step 2, each of those
channels was a call on every single sweep. Where that rule belongs (devpolicy,
the episode template, or an `nctl` reconciler that treats undeclared channels
as drift) is a decision for the episode's close, not something to slip in
here. The plan says the same.

Unsubscribed three in-system agents' bots on their behalf — handoff candidate.
