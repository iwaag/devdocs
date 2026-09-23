# Step 2 report — request progress is inspectable

Plan: [plan.md](plan.md) step 2. Evidence: [report1.md](report1.md). 2026-09-24 (JST), by the
Omni Agent. No agent was served; one listener (Front's) was restarted on new code.

## What was built

### `agag.trace` and `agentchat trace` (pyagag `22cbd74`)

One read that follows a request from any message down through every conversation opened on
its behalf, and says for each what its posts show. **No new store and no new record type for
the ordinary path**: it reads what already exists.

- **Tree** — a conversation's children are the topics carrying a root note
  (`[selfnote][rootchat] <channel>/<topic>`) that names it, *whoever* wrote the note, so the
  planner's own notes put a mission's task topics under its `workplan-` topic, and forge's
  put an `assetrun-` under its `assetplan-`. Each conversation is shown once, under the most
  specific conversation that anchors it; every anchoring note is listed (`anchored by Front
  #8331, autolab-agstudio1 #8302`), which is how a start that was *not* posted shows: the
  requester is missing from that list.
- **States**, from the posts alone (identity notes and acks decide who owns a
  conversation; `[state]` words are the owner's record; names are display only):

  | state | evidence |
  |---|---|
  | `not_started` | the owner opened it and nobody has posted since |
  | `queued` | a post after the owner's last word, not acknowledged |
  | `executing` | acknowledged, not answered; the age of the last ack or progress line is shown |
  | `awaiting_requester` | the owner answered last (and, when checkable, the requester has served it) |
  | `awaiting_delivery` | the owner answered the requester by name; the requester's `[served]` mark does not cover it |
  | `awaiting_human` | the conversation a human asked in: the agent spoke last |
  | `failed` | the newest owner post is a failure notice |
  | `done` / `cancelled` | `[state]` says so |
  | `unobservable` | the read failed — never read as "not started" |

  A `✔` on a conversation whose state is not finished is flagged ("resolved (✔) without a
  finished state").
- **Owed now** — mechanical facts only: a task whose predecessors are finished and that
  nobody has posted into, a post nobody acknowledged, an answer not served, a failure notice.
  Elapsed time is shown, not judged.
- **Freshness** — `observed_at`, per-conversation last activity, ages in the text.
- **Cost** — one message lookup, two realm-wide note searches, one history read per
  conversation (two when a name must be followed across `✔`). `calls` is reported: 13 for
  the p3 locations request (10 conversations), 10 for the forest-location request.
- `agentchat trace [<message id>] [--json]`; without an id it traces the conversation the
  run is serving (its anchor). `--json` is `agag.trace.v1`.

### Operation failures leave evidence where the request is (pyagag `22cbd74`)

A refused or uncertain `agentchat` write (`send`, `resolve`, `use`, `anchor`, `argue`) now
writes `[selfnote][opfail] <command> <channel>/<topic> refused|uncertain: <reason>` into the
conversation the run is serving. It is a selfnote — nobody is served by it — and the trace
shows it on that conversation (`! operation failed: …`). In F1 the refusal that led Front to
open a twin existed only in its transcript; now it is one read away from the request. A
timeout is recorded as *uncertain*, because the post may have landed.

### Defect D1 fixed (agfront `cc70b29`)

The ordinary `front` role's continuation read each remote conversation by its bare name, so
every task autolab had closed (`✔`) read as "awaiting a reply to your post" in every later
serving (report1, F2). It now follows the rename like the `threads/` files beside it. The
new test fails on the old code and passes on the new.

## Verification

| Check | Result |
|---|---|
| pyagag suite | 711 passed (14 new in `test_trace.py`, 2 new + 2 adjusted in `test_chat.py`) |
| agfront suite on the new pin | 182 passed (1 new) |
| **The p3 stalls, replayed** (fixture: realm messages 8280–8460, text shortened, cut at a message id) | at #8425 (13:50, F2): task 4 `not_started`, no Front anchor, listed as owed; at #8441 (F3): task 5 owed; at #8413 task 3 `executing`, not owed; the twin `✔ workplan-locations-2` flagged resolved-unfinished |
| Live, three credentials (Front, Observer, autolab) | the same 10-conversation tree, 13 calls each |
| **Front's own execution environment** | `agfront/.venv/bin/agentchat trace` with Front's credentials file and home variables, from inside a Front generation workspace: works, and defaults to the served conversation. `Bash(agentchat:*)` in Front's grant covers it |
| Front listener | kickstarted 16:23:56Z on `22cbd74`/`cc70b29`; startup recovery queued 0 |

The live trace of the p3 locations request shows what nobody had noticed: all four p3
missions still say `state note: started` although each was accepted — Front's acceptance post
in the workplan writes no `[state] done`, and `mission_done` is only run when somebody asks
autolab to close out. A trace is how that became visible; step 3 decides who writes it.

## What is not covered

- **Worker liveness.** `executing` says an ack exists and gives its age; it does not prove the
  run is alive (the plan's own caveat). The serving journals (`listener.sqlite`) hold that per
  host and are not read by the trace; each listener's `agag-status.json` is fresh (seconds) but
  describes the listener, not a serving.
- **Existing views** (agentroom's `/projects`, `/ops`) do not call the trace yet; nothing in
  this episode needed them to.
- **Observer** has the credential and the library; wiring it in is step 4.
- Deliveries that failed inside a listener (`FAILED after n attempt(s) … kept in the queue`)
  are still only in that listener's log and journal; the trace sees them as
  `awaiting_requester`/`executing` with an age.
- D2–D4 (resolve/un-resolve and the refusal's advice) belong with the handoff change in step 3.

## Handed forward

- To step 3: the trace is the check a progression change must satisfy — after it, a finished
  task followed by `not_started` must not be a state the realm can stay in.
- To step 4: "owed now" is the mechanical half of stall detection; its lines are candidates
  for Observer judgment, not verdicts.
