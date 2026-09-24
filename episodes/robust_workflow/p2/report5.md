# Step 5 report — a fixed final revision under combined failures

Plan: [plan.md](plan.md) step 5. 2026-09-24 (JST), by the Omni Agent as the requester's stand-in
and trial operator.

## Version set fixed before the first trial (attempt 1)

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

Three attempts. Attempts 1 and 2 each found defects that needed code, so the acceptance record is
**attempt 3**, on a revision that did not change during it. The failed attempts are kept below,
because what they found is most of what this step taught.

### The final revision (attempt 3)

| Component | Revision |
|---|---|
| pyagag | `deec513`, installed in all seven consumers, every listener and the relay restarted 2026-09-23 23:01:52Z (venv `direct_url.json` and process start times checked) |
| agobserver (`pj-agdev`) | `dc21380` (superproject `c24ec32`) |
| agfront / agautolab / agforge / agdevworld | `243c9dd` / `a507617` / `62d7409` / `e6643b4` |
| archsage / cagent | `645969a` / `c27f04a` |
| Models, settings, watchdog | unchanged from the table above |

| Trial | What ran | Result |
|---|---|---|
| **A3** normal cycle 1 + combined rename/recovery (rep 1) | 2-task mission; origin renamed after planning; **Front's listener stopped** right after it relayed task 1's acceptance, for 7 min 10 s, while task 1's closing report (in a ✔ topic), task 2's automatic start and task 2's report arrived; then an unrelated request under the origin's old name | Observer: the ✔ on task 1 first judged a deliberate close, then — same incident, same allowance — `undelivered`, asked **in the renamed origin** with an owed note at +6 min 51 s. Front back: its first serving took the answer up and wrote the receipt 15 s after it returned; the mention entries were then correctly *not* served again; task 2's report served by the mention route. **Rescued** on the receipt. The reused name got its own answer; nothing of the mission went there. Mission completed (`eb0e56a`, `067f50a`) |
| **B3** normal cycle 2: forge → human review → autolab, concurrent with A3 | icon request sent without asking (as told); forge's result arrived while Front was down | `undelivered` asked in B3's origin at +6 min 40 s; receipt 34 s after Front returned; **rescued**. Icon checked and accepted; autolab committed `b60eade` |
| **C3** normal cycle 3 + combined rename/recovery (rep 2) | 2-task mission; **task 1's topic renamed by hand** while it ran; a human ✔ on task 1 while it waited for acceptance; later the **origin ✔'d by mistake** while task 2 waited | autolab replied under the renamed topic ("destination … is renamed"). The ✔ on task 1 was judged `legit` (the worker was waiting on the human) — nobody asked; Front un-✔'d it when the human said the ✔ was a mistake. The ✔ origin: `origin_closed` **reported to the owners at +7 min 43 s**, nobody asked in the closed conversation; the human un-✔'d it (a human decision), the incident said *moving again*; mission completed (`76ca69b`, `c39561c`) |
| **D3** monitor failure (rep 2): judgment held, source stale | `triage-stall` armed before C3's task ✔; `mirror-stale` armed for 5 min 40 s | Judgment held from 23:21:53; the relay DM'd **`judgment_stalled` at +5 min 5 s** while looks went on (cycle 13 completed during the hold, 17 requests looked at); released, judged 6 min 39 s after it began. Stale: **`unable_to_observe` DM'd at +1 min 34 s** (the injected staleness has no `stale_since`, so it is reported at the first record; the 300 s bound applies to a real stale feed, fixture), no conclusion and no request during it, *ok again* 2 min after removal |
| **Monitor stop** (rep 1 on the final revision) | `monitor-stop`: thread ended 23:53:55, process and listener up | Relay DM'd **`stopped` at +5 min 10 s**: "its listener is polling, so the process is alive and the monitor alone is not moving". Restarted: first look resumed with 0 posts and 0 judgments — no replay |
| **E3** real long wait across the silence threshold; dead worker | 1-task mission; **autolab's listener stopped** 14 s after it acknowledged the task (the run died with it) | `silent` at **+45 min 17 s** after the last sign of work; judged `stall` in 61.7 s on the judge worker; asked in the origin twice (00:39, 00:49); Front each time said it would not poke autolab without the human's say-so and asked the human; **reported to the Developer at +67 min**. Operator ended the fault: autolab's restart served the interrupted task **once**, answered 38 s later, Front relayed it, the incident said *moving again*. Requester accepted |

Normal cycles: **3 / 3** (A3, B3, C3; E3 as well), results at the requester each time, two requests
running at once in A3/B3.

### Against the targets

| Measure | Target | Final revision |
|---|---|---|
| Normal cycles | 3 / 3 incl. forge → autolab and concurrency | **3 / 3** (+E3) |
| Answer never served → recovery request | (grace 300 s + a look) | **6 min 40 s, 6 min 51 s** |
| Recovery request → verified by its receipt | — | receipt 15 s / 34 s after Front returned (its listener was down); rescued on the next look (≤ 2 min) |
| ✔ origin with unfinished work → report | ≤ 8 min | **7 min 43 s** |
| Monitor thread stopped → visible and DM'd | ≤ 7.5 min | **5 min 10 s** (5 min 24 s in attempt 1, 5 min 12 s at deployment) |
| Judgment held → visible | ≤ 5.5 min | **5 min 5 s**; looks continued throughout |
| Source stale → visible | ≤ 5.5 min | **1 min 34 s** (injected; see D3) |
| Dead worker | candidate at 45 min, report after two requests | **45 min 17 s**; report at 67 min; one serving on restart |
| False `rescued` | 0 | **0** — both rescues read `done` with a receipt |
| Duplicate work | 0 | **0**. The interrupted E3 task ran once more after its listener died mid-run (its first run never finished); no completed work was redone |
| Lost tracked requests | 0 | **0**; 15 tracked at the end, none dropped with work open |
| Omni Agent rescue interventions | 0 | **0** (see below) |
| Missing start | ≤ 6 min | not reached by Observer: in attempt 1 the injected skip was recovered by **Front itself** 25 s after it happened (it noticed the start missing in autolab's closing report and posted it); p1's two live detections stand |

**Human decisions and operator actions, recorded apart from rescue.** Requests, acceptances and
quality checks on the working tree; one ✔ on a task and one on an origin "by mistake" and their
un-✔ — the human's own actions and decisions; listener stops and fault files — the operator's
fault injection, each ended by the operator. None of them moved stalled work on the system's
behalf; every recovery was Front's, asked for by Observer or triggered by its own listener.

**What was time-compressed.** Retention past the 12 h window across a restart, a request whose
origin is deleted, an unreadable conversation reported after 30 min, a lost store adopting a
renamed incident topic, cancellation closing an incident as a decision, and a stale mirror with a
candidate pending: fixtures on controlled clocks only (`agobserver/tests/test_p2_reproductions.py`,
`test_monitor_health.py`). The 45-minute silence and every other time above ran live.

### Cost and calls

| Period | Runs | Cost |
|---|---:|---:|
| Deployment's first looks (Front answering the nine false requests) | 13 | $0.47 |
| Attempt 1 | 27 | $2.64 |
| Attempt 2 | 29 | $2.53 |
| Attempt 3 (final) | 51 | $4.58 (Front 30, $2.80; autolab 15, $1.47; forge 3, $0.31; Observer 3 judgments, $0) |

Zulip calls added by the monitor: 0 reads per look (mirror); posts only for incidents and requests
(9 in attempt 3 after the monitor's restart); one `subscriptions` read per 10 min. The watchdog
reads files only; its alerts were 9 DMs over the whole step. Local-model judgments: 29–128 s in
p1, 29–62 s here once no longer held.

### Attempt 1 (revision `85cea0e` / `059e5a2`) — two defects

- **A** (2-task mission, origin renamed, a skipped start): both task reports reached the renamed
  origin; the reused name got its own answer; the missing start was recovered by Front in 25 s.
- **B** (forge → autolab): the task's closing report was **lost**. Front's listener read the task
  topic's listing, then its messages, while autolab's ✔ moved them in the same second, found
  nobody speaking and logged "nothing owed". And the monitor could not see it: autolab had
  started the task itself, so Front had no root note there, and the trace did not follow the
  parent hop that callback routing follows. Fixed in pyagag `61df5ee` (the mention route judges
  its trigger by id first; children inherit the parent's requesters). After that deployment the
  first look found the lost report and asked Front, which relayed it 12 s later — but an
  owner-route serving left no receipt, so the next look could not verify it; the second request
  went out 29 s before the next rollout and the incident was **reported to the Developer
  unnecessarily** (22:38). Fixed in `4475a1b`: the request carries `[selfnote][owed]`, and the
  serving that processes it writes the served mark after its reply.
- The monitor-stop fault ran here too: DM at +5 min 24 s.

### Attempt 2 (revision `4475a1b` / `742660d`) — two more

- **A2** with Front down 7.5 min: the ✔ on task 1 was judged a deliberate close and the dismissal
  then **swallowed the `undelivered` that followed** on the same work (fixed `5a64a63`: a
  dismissal answers a judged question, never a mechanical one); its "Now undelivered" line was
  posted twice (`dc21380`). After the fix, asked in the renamed origin, receipt written, rescued.
- **B2** with the same outage: rescued, but the forge result was **served a second time** by the
  mention route eleven seconds after the owner route's receipt, because the receipt note had not
  reached the listener's own mirror yet (pyagag `deec513`: marks it has just written count at
  once). Both cycles completed.

### What this does not prove

Five trial requests on the final revision, on one small project, by one requester. The judged kinds
rest on a 27B local model: it judged a human ✔ on a waiting task *legit* here where it judged the
same shape a *stall* in p1 — both defensible, neither measured. Front's willingness to recover
(the missing start in attempt 1, the owed answers) is behavior, not a guarantee. The staleness
path was exercised with an injected flag, not a real expired event queue.
