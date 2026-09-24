# Step 1 report — the remaining gaps, reproduced, and the baseline

Plan: [plan.md](plan.md) step 1. 2026-09-24 (JST), by the Omni Agent. Nothing deployed in this
step. The reproductions are fixtures committed beside the code they exercise: pyagag
`tests/test_p3_reproductions.py` and agobserver `tests/test_p3_reproductions.py`.

## Where things stand

| Check | Result |
|---|---|
| Version set | p2's final set is still running: pyagag `deec513` in all seven consumers, agobserver `dc21380` (introduction `0bf46af`). Listeners have been up since p2's trials, and the E3 restart of autolab ran on the same revision |
| `nctl status` | Nautobot, worker and dumps are healthy. agstudio was observed 2.3 h ago, the other nodes 5.2 h ago |
| `nctl drift` liveness | Front, forge, archsage, cagent and Observer: `polling`. autolab on this host: `stale`, because the observation predates E3's listener restart (its status file was rewritten a minute before the check). autolab on the VM: `stale`, as in p2 |
| Monitor health (`/healthz`) | `ok`: cycle 68 completed 73 s before the check in 0.5 s, 18 requests looked at, 15 tracked, 0 open incidents, source `live`. The last judgment was E3's `silent` → `stall`, in 61.7 s |
| Incidents on record | 27 files: 18 p1-era incidents withdrawn in p2, 4 rescued, 3 reported (B's undelivered, C3's `origin_closed`, E3's silent), 2 dismissed (`resolved_live` on C3 task 1 and on p3's `workplan-locations-2`) |

A mirror snapshot (a `.backup` of Observer's store in the scratchpad) was used for every count
below, with no Zulip call.

## Reproductions

Each fixture states the outcome p3 requires and is `xfail(strict=True)` today, so the fix has to
remove its mark. With the mark removed, each one fails for the stated reason (checked). The suites
are otherwise green: pyagag 757 passed, agobserver 108 passed.

| # | Scenario | What happens now | Expected | Path |
|---|---|---|---|---|
| **W1** | The only conversation opened for a request is a delegated one whose owner asked the human a question. Front relayed the question (its receipt is written) and the human has not answered. The human then ✔s the conversation, and the judge rules it a legitimate wait | The incident is `dismissed`, correctly, and nobody is asked. The request **leaves the tracked index in the same look**, so after the 12 h window it is never looked at again | No nudge, and the request stays tracked across inactivity and a restart until it is completed, cancelled or replaced | `Monitor.retain`: `closed_by_decision` treats every node of a dismissed `resolved_live` incident as finished, and `open_origins` excludes `dismissed`. "Not a stall" is being read as "finished" |
| **J1** | A ✔ conversation's judgment is queued. While the worker judges that snapshot, the human answers in the conversation (a cancellation in words) | The worker's verdict (`stall`) is filed under the incident key. The next look pops it for the new candidate (different `since`, different state) and **posts a recovery request on it** | A verdict about superseded evidence is rejected and the new state judged, with no request and no end of tracking on the old verdict | `Monitor.judged`: `_verdicts` is keyed by incident only, and no snapshot identity travels with the job. The same cache also keeps a verdict whose candidate vanished, to be applied to a later candidate on that conversation |
| **S1** | A delegated answer is posted, and the requester's home is then served on its **owner** route (a human post, or Observer's request about a *different* answer). The serving reads the answer in its threads and replies | No receipt is written. The mention entry for the same answer is then **served again**, and the run says "nothing new" | One serving. After the reply is confirmed delivered, the answer it read gets a served mark | `Listener._after_delivery`: the owner route marks only answers named by an `[owed]` note (`_mark_owed`), and the mention route only its trigger. Nothing records which remote posts a serving's threads contained (`topics.write_threads`, `agfront.evidence.write_evidence_threads`) |
| **S2** | A callback naming the bot arrives while its listener is down, and the owner ✔s the topic right after (autolab's close-out) | Startup recovery reads open topics only (`mirror.topics(include_resolved=False)`), so **the callback is never queued**. It comes back only if Observer asks for it | Found at startup by the delegation's anchor, served once, with no twin | `Listener.recover` |

Live and historical evidence behind S1 (Front's serving journal, `listener.sqlite`):

- A3 `front-robust-p2-a3-renamed`. Serving 211 (mention for the plan's answer #9584) read task 1's
  report #9590 in its threads and relayed it (#9605), marking only #9584. Serving 212 then served
  #9590 and answered "Nothing new has come in" (#9609).
- A3 again. Serving 216 (owner route, Observer's owed request for #9629) relayed task 2's report
  #9642 (#9656), marking only #9629. Serving 218 served #9642: "Nothing new" (#9664).
- A2/B2: servings 201 and 200, the same shape after owner servings 198 and 199.

### Mission acceptance, traced (no fixture: step 3 defines what the acceptance is)

1. autolab closes the last task (`serve_run` → `record_result` → `[state] completed`, ✔) and
   `start_next_task` finds nothing left. It says "every task of m… is finished; the mission waits for
   your acceptance in `pj-…/workplan-…`". The mission stays `started`.
2. Two things happen next, and neither records a completion:
   - **Nobody posts in the workplan** (C3 m9738, A3 m9570, E3 m9828 and the rest of p2). The human's
     acceptance of the last task already said "That completes this mission from my side" (#9674,
     #9804), and Front relayed it into the *task* topic only.
   - **Front relays the mission acceptance into the workplan** (p1 N1 #8916, p3 #8460). That buys
     autolab a superdirector run that writes no flag and answers "Noted"/"Understood" (#8921, #8462),
     naming Front, which buys Front a callback serving. That is two paid runs, and the mission is still
     `started`.
3. The writes that exist: `python -m agautolab.mission_done` (autolab's entrance, and only when
   asked in its own channel) and the relay's completion door (`agentroom.close._accept_mission`, the
   human's button), which writes `[state] accepted` per task and `done` on the mission. Neither
   records *which* acceptance it rests on, and neither is on the requester's normal path.
4. Consequence for Observer: the trace reads the workplan as `awaiting_requester` for ever, and each of the
   15 tracked requests is held by such a workplan (11 of them by nothing else).

## Baseline

| Measure | Now | Evidence quality |
|---|---|---|
| Tracked requests | **15**. Every one holds a `workplan-` node whose mission is `started` with all tasks finished, and 11 are held by nothing else. The other four also hold p3's ✔ `workplan-locations-2` (dismissed, so `retain` skips it), a p3 task awaiting its requester, the setup plan of `pj-robustp1` (no mission), and B's unreceipted #9368. W1's gap is masked the same way: C3 carries a dismissed `resolved_live` on task 1 and stays tracked only through its workplan | measured (trace of `tracked.json` off the snapshot) |
| Missions `started` with every task finished | **22** of the realm's 41 missions, all 12 robust-trial missions of p1 and p2 among them. No task anywhere carries `accepted`. The 6 `done` missions with tasks were closed by the relay's button or `mission_done` | measured |
| Redundant servings (a mention serving whose answer was already posted when an earlier delivered serving of the same home began) | Front: **10 of 142** delivered mention servings in the whole journal, **4 of 34** since p2's monitor started (#9187). autolab and forge: 0 | measured, with a heuristic. "Posted before the earlier serving's ack" means the answer was readable, not that it was proven to be in the threads. The four p2 ones were checked by hand against the posts |
| Answers without a receipt | **1** under p2's rule (`receipts_from` #9187): B's #9368, relayed by an owner-route serving before `[owed]` existed and reported in attempt 1. **27** under the strict rule (every answer needs a mark) | the 1 is reproduced (S1's shape). The 27 are p1-era records whose listeners skipped marks (p2 report5), so they are history and not evidence of a current defect |
| Recovery false positives on the final p2 revision | 0 | p2 report5 |

## Reproduced, hypothesised, historical

- **Reproduced** (fixture fails for the stated reason; live instances cited): W1, J1, S1, S2, and the
  mission gap (live, above).
- **Code facts, not reproduced**: the verdict cache (`_verdicts`) keeps a verdict whose candidate
  vanished and applies it to a later candidate on the same conversation. `retain` and the
  incident loop run in one look, so a judgment on superseded evidence could also end tracking
  (W1's path fed by J1's verdict). `Listener._mark_owed` reads home by its live name, so a home ✔'d
  between the reply and the receipt gets no receipt. The relay's `_accept_mission` and
  `mission_done` write `done` without evidence of the acceptance they rest on.
- **Historical, insufficient evidence**: the 27 strict-rule unreceipted answers (p1 listeners), and
  whether `legit` on a human ✔ is judged consistently (p1 said stall, p2 legit; step 6).

## Expected outcomes, as the later steps' acceptance

- Step 2: W1 and J1 pass. A dismissal suppresses a nudge and ends nothing. The request leaves
  tracking on completion, cancellation or replacement evidence only. A verdict is applied only to
  the snapshot it evaluated, and a superseded one is re-queued without spending a request. Backlog
  and invalidations are visible in the health record.
- Step 3: a mission is accepted by an explicit, evidenced operation. The workplan reads `done`, and
  the request leaves tracking on the next look. A repeated or interrupted acceptance creates neither
  a second completion nor a planning run. The 22 stuck missions are reconciled only where the
  acceptance is on record.
- Step 4: S1 and S2 pass. Receipts cover what a serving actually read, after confirmed delivery, on
  every route, and a callback under ✔ is recoverable at restart by anchor.
