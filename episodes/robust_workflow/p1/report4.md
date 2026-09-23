# Step 4 report — Observer finds stalls nobody registered, asks for recovery, records incidents

Plan: [plan.md](plan.md) step 4. 2026-09-24 (JST), by the Omni Agent. Deployed on this host
16:56–17:13 UTC; one live trial (T1) with an injected fault.

## Design

The `observe` role stays what it was (one watch, three-valued answer). Observer gets a
**second clock**, `agobserver.monitor`, beside the watch worker, and a **judgment role**,
`triage`. Code decides what records can decide; the model only decides what they cannot.

| Stage | What does it | How |
|---|---|---|
| Discover | code | every open `#front › front-…` conversation with activity in the last 12 h is an active request — off Observer's mirror, no watch registration, no Zulip call |
| Look | code | `agag.trace` over the mirror (`MirrorReader`, pyagag `e0c3ff9`) and `stall_candidates`: `unstarted` (a task with no post and no start after every earlier task finished, mission started, not held), `unacknowledged` (a post the owner's listener did not pick up), `undelivered` (an answer the requester was never served), `failed` (a failure notice), `resolved_live`, `silent`. Each has a grace time; elapsed time makes a candidate, never a verdict |
| Judge | local model (`triage`) | only `resolved_live` (a ✔ on live work: correction or mistake?) and `silent` (long job or dead worker?): `stall` / `legit` / `unclear`, from the trace and the conversation's last messages |
| Ask | code | one post in **the conversation the request came from**: the fact, the expected next action, who is responsible, `agentchat trace <id>`. Front owns that conversation and holds every tool a recovery needs; the human reads it there. Max 2, 10 min apart, each after a fresh look |
| Verify | code | *rescued* only when a later look no longer finds the candidate |
| Report | code | when requests are spent, or there is nobody to ask (the origin is ✔, or its owner is the one not answering): the realm's owners by name, in the incident topic; then stop |
| Record | code | one `incident-<kind>-<id>` topic in Observer's channel: detection with evidence ids, each request, the outcome — "rescued … the cause is not removed by this" or "stopped" — and `[selfnote][incident]` / `[state]` notes. Incident topics are records, not watches (intake skips them) |

Duplicates and restarts: an incident is its `Candidate.key` (kind, conversation, evidence id)
in a local store; a lost store adopts the existing incident topic instead of opening a second
one and does not ask again blindly. Posts go through `agag.delivery` (read-back on an
ambiguous send). Observer's own unanswered request is never an incident of its own.
`AGOBSERVER_MONITOR=0` switches it off; `AGOBSERVER_MONITOR_SECONDS` (120) and
`_WINDOW_HOURS` (12) tune it.

**Why the origin conversation and not the stalled topic.** A post by Observer in a task topic
would make Observer that task's requester: autolab's report would be handed to Observer, not
to the asker (the "bot posts re-serve and misdirect" trap). The origin is Front's; its reply
there reaches the human, and a start Front then posts into the task keeps Front the
requester.

## A defect found on the first live look

Started at 16:56:36Z, the monitor at once opened `incident-silent-8309` on p3's twin
`✔ workplan-locations-2` — a conversation autolab had answered. Observer's mirror held the
pre-✔ topic name on every message before the resolve: **Zulip delivers moves only to
subscribers**, Observer's bot was in none of the project channels, so every ✔ there was
invisible to it (the trap already known for the relay's reader). The triage judged it
correctly (`legit`, "an explicit duplicate … directs all work to the original", 43 s on the
local model) and the incident was dismissed — but the facts were wrong. Fix: the monitor
keeps the bot subscribed to every public channel (checked on the first tick and every 10 min;
a resync when anything was added). It joined 49 channels and resynced at 16:58:35Z; the twin
then read correctly as ✔-while-answered, was judged `legit` again and dismissed. A re-judgment
that agrees with the previous one no longer posts.

## Live trial T1 — a missing start, found and recovered without the Omni Agent

A two-task mission through the ordinary entrance, the Omni Agent as the requester's stand-in
(`#front › front-robust-p1-t1`), in a new disposable project `pj-robustp1` that Front opened
on the relayed decision (`refactorp1` was archived; Front asked instead of guessing). One
fault: `agautolab/.local/faults/skip-next-start` (one-shot) armed while task 1 ran, so the
automatic start of task 2 would be skipped as a crash or bug would skip it.

| UTC | Event |
|---|---|
| 17:01:22 | request (#8721) |
| 17:02:57 | Front opened `pj-robustp1` (after the archived-project answer) |
| 17:03:28 | Front: `workplan-wordcount-lines` #8736, "plan and start in one pass" — its reply already stated the new contract correctly: "autolab starts task 1 itself" |
| 17:03:46 | autolab planned m8741, **started task 1 itself** (start note), and the listener served it from that note — no relay post |
| 17:04:35 | requester accepts task 1 (#8767, after running the four modes on the tree) |
| 17:04:40 | Front posts the acceptance into task 1 (#8770) |
| 17:04:58 | autolab closes task 1; **fault injected: the start of task 2 skipped** |
| 17:08:30 | **Observer: `incident-unstarted-8750` opened, recovery asked in the origin (#8785)** — 3 min 32 s after the fault, with no watch registered |
| 17:08:37 | Front (served by that post) ran `agentchat trace`, posted the start (#8789) and reported its id; autolab served task 2 the same second |
| 17:10:32 | Observer's next look: **rescued after 1 request**; "the cause is not removed" |
| 17:10:50 | requester accepts task 2; 17:11:08 autolab closes it: "every task of m8741 is finished; the mission waits for your acceptance" |

| Measure | T1 |
|---|---|
| Wall time, request → last task closed | 9 min 46 s (incl. opening the project) |
| Stall (injected) | 3 min 39 s, fault → task 2 served |
| Detection time / recovery time after the request | 3 min 32 s / 7 s |
| Omni Agent rescue interventions | **0** (the posts were requests and acceptances) |
| Front posts to other agents | 6: goal, setup, workplan, 2 acceptances, 1 recovery start (the fault). **No task-start relay on the normal path** (p3's pattern would have been workplan + permission + 2 starts + 2 acceptances) |
| Runs / cost | Front 11 ($1.09), autolab superdirector 2 ($0.23), supercoder 4 ($0.39): 17 runs, **$1.71**; Observer triage 2 runs before the trial (p3's twin), $0 (local model) |
| Zulip calls added by the monitor | 0 per look (mirror); posts: 3 for the incident, 1 request; 1 `subscriptions` read per 10 min; one subscribe + resync (~62 calls) when a channel appears |

## Other findings in T1

- **A false report, of a new kind.** autolab's reply for task 2 said "committed … and not
  pushed"; its own close-out line under it said "pushed main to Gitea (1 commit)", which is
  true (`origin/main` = `64d7e4a`). Front relayed the prose and offered to push. The supercoder
  guide now says pushing and the devlog record happen after its reply and that it must not
  state the push itself (agautolab `bd80a35` in pj-agdev). Not a stall; the receipt was right
  and the prose was wrong — the same lesson as F2, one level down.
- autolab's planning reply still said "opened …/workrun-task2 …; post there to start it" beside
  "task 1 starts now" — removed (same commit).
- Front's replies after Observer's request are addressed `@**agobserver-agstudio1**` (the last
  other speaker) before naming the Omni Agent; harmless (Observer has no mention route outside
  argues) but noise.
- The mission's own acceptance is still not recorded in the workplan (report3, remaining).

## Verification

| Check | Result |
|---|---|
| pyagag | 734 passed (`MirrorReader`, `stall_candidates`: the p3 F2 moment is a candidate at 6 min, not during the run, not before the grace; a mirror trace makes no call) |
| agobserver | 81 passed (`test_monitor.py`: found without registration, one request per interval, rescue verified on a later look, bounded then reported by name, restart and lost store do not re-ask, closed origin not looked at, incident topic is not a watch, subscription + resync) |
| agautolab | 257 passed (the one-shot fault is spent after one skip) |
| Deployment | pyagag `e0c3ff9` in agobserver (was `d20720c`), agautolab, agfront; Observer, autolab and Front restarted, recovery queued 0; Observer's introduction re-posted with the new section |

## Limits

- Discovery covers `#front` only: a request that did not come through Front (a routine run's
  delegations, a project room post) is not looked at yet.
- `unacknowledged` on Front's own conversation is reported, not recovered: when Front's
  listener is down there is nobody in the realm to ask.
- The judged kinds (`resolved_live`, `silent`) were exercised on real data only as `legit`; a
  judged `stall` is covered by fixtures, not yet live (step 5).
