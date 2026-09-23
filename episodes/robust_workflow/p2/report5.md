# Step 5 report — a fixed final revision under combined failures

Plan: [plan.md](plan.md) step 5. 2026-09-24 (JST), by the Omni Agent as the requester's stand-in
and trial operator.

## Final version set (fixed before the first trial)

| Component | Revision |
|---|---|
| pyagag | `85cea0e` — installed in all seven consumers (agfront, agautolab, agforge, agobserver, archsage, cagent, the agentroom relay), verified from each venv's `direct_url.json` |
| agobserver (`pj-agdev`) | `059e5a2` |
| agfront / agautolab / agforge | `709f720` / `e91b785` / `c800c50` |
| agdevworld (relay) | `66236c9` |
| archsage / cagent (`pj-clusterintent`) | `b070560` / `2b4f9f4` |
| Models | Front, autolab, forge: Sonnet 5 through claude_code (every role on record); Observer's intake, observe and triage: `qwen3.8:27b-mxfp8` through agcode on this host's ollama. Unchanged through the run |
| Monitor settings | look every 120 s; discovery window 12 h; `receipts_from` #9187 (first start of this revision); retry 600 s, two requests; `origin_closed` grace 300 s; unobservable reported after 1,800 s |
| Watchdog | the relay, every 30 s; alerts by DM to the realm's owners |

## Targets (set before the run)

| Measure | Target | Basis |
|---|---|---|
| Normal multi-task cycles completed, results at the requester | 3 / 3, including a forge → autolab handoff and two requests running at once | plan |
| Missing start: fault → recovery request | ≤ 6 min | p1's target; grace 180 s + one look (120 s) + slack. Looks no longer wait for judgments |
| Recovery request → work served | ≤ 2 min | p1 |
| ✔ origin with unfinished work → report | ≤ 8 min | grace 300 s + one look + slack |
| Monitor thread stopped (process alive) → visible and DM'd | ≤ 7.5 min | 3 × 120 s + 60 s, watchdog 30 s |
| Judgment held → visible | ≤ 5.5 min | 2 × 150 s timeout, watchdog 30 s |
| Monitor's source stale → visible | ≤ 5.5 min | 300 s, watchdog 30 s |
| Dead worker, silent past 45 min → request, then report | candidate at 45 min after the last sign of work; report after the second request | p1 thresholds |
| False `rescued`, duplicate work, lost tracked requests, Omni Agent rescue | 0 each | plan |

## Trial plan

All requests through `#front`, the Omni Agent posting as the requester's stand-in, on the
disposable project `pj-robustp1`. Faults are the operator's, never repaired by hand; human
decisions (acceptances, an un-✔ of a ✔ the human set by mistake) are recorded as such.

| Trial | Scenario rows covered | What |
|---|---|---|
| A | normal cycle 1; origin rename; missing start; old name reused; concurrent requests | 2-task autolab mission. While task 1 runs: the origin is renamed and a one-shot fault skips task 2's automatic start. After recovery a new, unrelated request takes the origin's old name |
| B | normal cycle 2 (forge → autolab); concurrent requests; monitor thread stopped | forge makes an icon, autolab puts it into the README, running beside A. The monitor thread is stopped mid-trial and restarted |
| C | normal cycle 3; child renamed; origin ✔'d while work remains; human decision | 2-task mission. The running task's topic is renamed by hand; the origin is ✔'d by mistake; the human un-✔s it after the report |
| D | judgment held; stale source; other requests still covered | a human ✔ on a running task (a judged kind) while the next judgment is held by a fault; the monitor's source marked stale for a while |
| E | long real wait across the silence threshold; dead worker | autolab's listener stopped right after it acknowledges a task and kept down past 45 min; then brought back |

Time-compressed (fixtures, controlled clocks): retention past the 12 h window across a restart,
unobservable reported after 30 min, a request whose origin is deleted, a lost store adopting a
renamed incident topic.

## Before the run: two defects found at deployment, fixed, and a restart of the version set

(Recorded here because they changed the version set; neither was in a trial.)

1. **The first live look opened nine false `undelivered` incidents** on finished p1 tasks and posted
   a recovery request into five `#front` conversations; Front answered each correctly ("nothing is
   waiting on anyone") in five short runs and did no work again. Cause: step 3 made a served mark the
   only receipt for an answer, and the listeners before pyagag `87ac87e` (p1's defect 6) had skipped
   a callback whose topic was ✔'d right after it, leaving the mark one post short of an answer the
   serving had in fact read (N1 task 2: mark 8885, a progress line echoing `@**Front**`; answer 8891,
   one second later). The monitor thread was stopped by the operator fault within two minutes, the
   incidents withdrawn (`python -m agobserver.withdraw`, a new operator command), and the trace given
   `receipts_from`: answers older than the monitor's first start are read as p1 read them. The
   restart in between showed the same records a second time as ✔-while-awaiting-delivery (nine
   `resolved_live` episodes, withdrawn, no request posted) — which is why the boundary went into the
   trace rather than into one candidate kind.
2. **A false `judgment_stalled` alert** (one DM to the Developer): the health record was written only
   per cycle, so a judgment that ended after the last write read as still running once the thread
   was stopped. The judge now rewrites the record when a judgment starts and ends. The same stop
   produced the first live `stopped` verdict: thread stopped 21:23:46Z, relay verdict and DM at
   21:28:58Z (**5 min 12 s**), "its listener is polling, so the process is alive and the monitor
   alone is not moving".
3. While writing the trial plan: a ✔ origin with unfinished work produced no candidate at all
   (Front does not deliver into a finished conversation). Added `origin_closed`, reported once after
   a 300 s grace.

## Results

(filled in as the trials run)
