# failsafe p3 — step 5: residue retired, retention bounded

## 1. Plain conversations no longer stay tracked forever

**Owner: agobserver `monitor.py`** (pj-agdev `a1c09c2`).

**Dry run first.** The new rule was run against a read-only copy of
Observer's mirror before rollout, request by request. It dropped exactly
the four requests with nothing owed (o11538, o11539, o11771, o8721) plus
o8286 (below). Every request holding a unit of work stayed.

The rule (`monitor.satisfied`, used by `retain` and `origin_closed`):

- **A plain conversation** (no mission, task, asset or run in it) no
  longer keeps its request tracked when:
  - its agent answered, the answer was taken up by whoever asked, its
    serving ended, and the next move is the requester's; or
  - somebody ✔'d it after its serving ended. A plain exchange has no
    other record, so its ✔ is how it is closed.
- **These still keep a request tracked:**
  - a unit of work that has not recorded `done`/`cancelled`: its record
    is still the only end;
  - an answer not yet taken up;
  - an open or queued serving;
  - **the requester's own conversation owing an answer** (a new post,
    a serving in progress, a failure notice). Only child conversations
    counted before;
  - an open incident;
  - **a held request** (`held.json`), always. Held requests are also
    looked at from the hold itself, so a lost index does not drop them.
- **Age is not completion.** Nothing in the rule reads time.
- **Rediscovery.** A request that left tracking is found again by the
  12-hour window over `#front` as soon as its conversation moves. The
  test does this across a restart.
- **`python -m agobserver.hold --retire <why> o<id>`** is a person's
  explicit disposition for residue that owes nothing but cannot say so.
  Typically that is a finished run from before `end=` markers, which
  still reads "open".
  - It is kept in `retired.json`, beside `held.json`.
  - A retired request leaves tracking and gets no incidents.
  - **The retirement lapses by itself** when anything new is posted in
    the request, and the ordinary rules apply again.
  - `--release` undoes a hold or a retirement.

Tests (`test_p3_failsafe.py`):

- a satisfied exchange leaves tracking, and comes back when it moves,
  across a restart;
- an answer not taken up keeps the request tracked;
- a held request survives outside the window;
- a retired request leaves tracking until it moves again, across
  restarts;
- a closed plain exchange is not unfinished work.

## 2. Bounded caches

| store | before | now |
|---|---|---|
| Observer's incident records (`incidents/*.json`, `~N` archives) | kept for ever | `Monitor.prune`, every 60th look: a closed record or archive is removed 14 days after its last change, unless its request is tracked or held, or its review or a later outcome is owed. **Episode counts are kept** (`episodes.json`), so a later episode never reuses an incident topic's name. The Zulip topics are the record and stay. Test: `test_closed_incidents_are_pruned_…` |
| `tracked.json` | 11 entries, 7 of them plain or residue, some 78 h old | bounded by the rule above: now `o11711`, `o8512` (both held) |
| probe state (`health.json`) | 24 h TTL (p2) | unchanged: already bounded |
| live execution records | newest 200 (p2) | unchanged; their `.injected` markers are pruned with them, and orphans are removed |
| listener journals (`listener.sqlite`) | — | **not bounded, by decision**: 50–680 rows and ≤ 2.1 MB per listener after weeks. Their rows are the serving evidence that `previous()`, redelivery and receipts read |
| `agobserver/.local/incidents-p1/` | robust_workflow p1's name-keyed store, marked disposable | deleted |

A restart does not re-create retired work. The monitor's only index is
`tracked.json`, plus the origins of open incidents and the holds. A
retired or satisfied request is in none of them.

## 3. The p2 reviews, reconciled

Both reviews now show their final outcomes. The step-4 rollout appended
them automatically:

| review | occurrence | trial | later (auto) | status recorded (`review_status`, by the Omni Agent) |
|---|---|---|---|---|
| `review-autolab-stopped` | 1 | A, `silent-exit` | work `done` | reviewed: injected trial; 129 s to Front; no product defect |
| | 2 | A2, `silent-exit` + `review-exit` | work `done` | reviewed (as above; 123 s) |
| `review-autolab-uncertain` | 1 | C, `freeze-after-tool` | moved again 02:14 UTC, then `done` | reviewed: injected trial; SIGCONT by hand after the escalation |
| | 2 | D, `probe-fail` | moved again 02:23 UTC, then `done` | reviewed; **fixed**: P2-1 (agobserver `5004bf3`), re-tried in D2 |
| | 3 | D2, `probe-fail` | `done` | reviewed |

- Remaining fix from these reviews: **none**. The listener still does
  not report a harness exit itself; Observer's probe covers it (129 s),
  and that is recorded as a finding.
- **Both review topics stay open for the Developer's ✔.** The ✔ is the
  Developer's own record that they looked. The statuses above are the
  Omni Agent's follow-up notes, and say so.

## 4. Legacy items: an owner and a disposition each

| item | current record | disposition | owner |
|---|---|---|---|
| **o11711 / m11741** (aisvgs round 2, sage p2) | task 1 open. Its harness ended on 2026-09-26 13:43 after posting "still running: the local kit run". The copy `missions/aisvgs/m11741` holds 3 commits and uncommitted reports (`strand2b-reinforcement.md`, `strand5a-people.md`, `strand5b-products-practice.md`) and kit scripts. It sits on agstudio's disk only | **held** (unchanged since p1) | the Developer: see below |
| **m8519** (mediagen `flux2_scenery`, adventure_game p3), request o8512 | task 1 completed; task 2 showed its result on 2026-09-23 (#8565: verified end to end, `21bcd70`/`31c0ba1` pushed) and waits for its requester's agreement | **held** (`hold o8512`, new) | the Developer: agree to task 2 in `work-m8519 › workrun-task2-m8519`, then record the mission's acceptance, or rework or cancel |
| **m7601** (protoprey research round), **m7732** (protoprey first build), adventure_game p1 | every task `completed` (4 and 5); mission `started`; no acceptance on record. p1 was ended by the Developer after step 4; ProtoPrey v0.1.0 was delivered unplayed | **pending the Developer's acceptance**, documented. Not tracked; not replayed | the Developer: `agentchat accept <id> --evidence <their post>` if they accept, or cancel |
| **m6113** (studyrealworld publish, 2026-09-12) | task 1 `completed`. The plan topic was ✔'d on 09-12 after autolab found the stage already done (#6177). No mission state note: it predates them | **pending an acceptance record**, documented. The work itself was found already done | the Developer (the original requester) |
| **m9349, m9697** (robust_workflow p2 trials) | tasks completed; acceptance never recorded | **cancelled.** Their requester (the Omni Agent's stand-in) retired them without acceptance through autolab's own plan path (#12599, #12600). autolab wrote `[state] cancelled` on each mission and archived the work channels (`work-m9349`, `work-m9697`). The planner read the new `status.md` and said the task was `completed` and the mission not accepted. **Defect found:** `cancel_tasks` also rewrote each completed task 1 as `cancelled` (#12602, #12607). It is fixed in agautolab `32ad9b9`, and a finished task now stays finished. The two records were **not** restored, because their channels are archived and posting there needs an admin unarchive. Their history keeps `completed` (#9367, #9724) and the integration, and the last note reads `cancelled` | done (record inaccuracy documented) |
| **a11459** (forge, give_context_easier p1 trial), request o11450 | forge plan shown and awaiting its requester; run never started | **retired** in Observer (`hold --retire`). The trial's requester is not continuing it, and **forge has no cancellation record**, so forge's own record keeps reading "awaiting requester" | Omni Agent (trial); forge's missing cancellation is a remaining limitation |
| o11522 (sage p2 aisvgs round 1) | mission m11579 done and accepted; its routine run conversation predates `end=` and reads open | **retired** (`hold --retire`), with that reason | done |
| o11538, o11539, o11771, o8721, o8286 | plain exchanges, satisfied or ✔'d | left tracking under the new rule | done |

**Decision-ready: m11741 (o11711).** The run's results so far are in its
mission copy, not integrated. Choices:

1. **Resume.** Release the hold (`hold --release o11711`) and post in
   `work-m11741 › workrun-task1-m11741` asking it to continue from what
   the copy holds. The next serving sees the copy and finishes the
   strands. It is then an ordinary recovery, and the monitor watches it.
2. **Take what exists.** Ask the task to commit and show the current
   reports as its result, then agree to it there. The close-out
   integrates it into `aisvgs` `main`.
3. **Cancel.** Say so in `workplan-aisvgs-round2`. The copy is released
   and its work kept on `autolab/m11741`.

Until one of these happens it stays held, and no one is asked about it.

## 5. Other cleanup

- **p1 and p2 trial conversations**: all twelve `#front ›
  front-failsafe-p{1,2}-*` conversations ended with Front's report of a
  recorded acceptance (p1-t1: nothing needed). They are ✔'d by their
  requester, the Omni Agent. Their missions were all `done` already.
- **Fault files and timing overrides**: none present in autolab or
  Observer. The hooks themselves stay (`silent-exit`,
  `freeze-after-tool`, `probe-fail`, `review-exit`, `stop-mid-task*`,
  `timing.json`).
- **Temporary processes**: none. The one `vite` process belongs to the
  developer's own terminal and was not touched.
- **Mission copies**: only `aisvgs/m11741` (held) remains. `robustp1` and
  `protoprey` hold nothing but their locks.
- **Obsolete configuration**: the `.local/refs.toml.pre-catalog` files of
  agfront, agautolab, agforge and archsage were deleted. They were
  superseded by the shared catalog on 2026-09-26, and no code reads them.

## 6. Dependency pins

These follow actual consumers:

- pyagag `27b85d9` in agautolab and agobserver, the writer and reader of
  live records;
- `b2cdf75` in agfront, agforge, archsage and cagent, which have
  conversational roles and do not keep live records;
- `f5c4359` in the relay and comfynotify, which use no changed module.

Superseded code removed in this phase:

- the fenced-mark splitter and its two slip rules;
- Observer's per-kind hypothesis tables;
- the serial probe loop.

## Tests

agobserver 168 passed (`test_p3_failsafe.py` 18); agautolab 318 (+1: a
cancelled mission keeps its completed task completed).

## Rollout

- Observer kickstarted at 04:39:32Z with no run in flight.
- Holds, retirements and review statuses were recorded after it, and the
  two trial cancellations went through autolab's live plan serving.
- autolab's listener and gateway were kickstarted for the `cancel_tasks`
  fix, with no run in flight.
