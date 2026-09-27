# progress_panel p1 — step 4: transitions and observability gaps

Date: 2026-09-27 (UTC 09:01–09:07).

## Read-model tests from recorded conversations

`pyagag/tests/test_progress.py` — 25 tests over `fixtures/progress_p1.json`
(three requests as the realm held them; cut at a message id, each cut is
the realm a reader would have found):

| case | where | checked |
|---|---|---|
| planning | o13116 at #13120 (no mission) and #13125 (mission, no task) | the request is `working`; the plan `planning`, no total, no percentage |
| tasks known | #13136 | 0/1, the task `working` (conversation evidence, band `claimed`) |
| changing totals | + a second plan doc and task 2; then task 2 cancelled | 1/2 with "plan revised 1×: 1 → 2 tasks"; then 1/1 with 1 cancelled |
| queued work | task 2 behind unfinished task 1 (relay test) | `queued`, "after task 1" |
| healthy quiet tool wait | a fresh `waiting` check on Bash `pytest -q` | `waiting` with the call and its age; band pulses |
| stale / unknown data | a check older than 120 s; an open serving quiet for 30 min; an unreadable origin | `unknown`, band `unknown`, never animated |
| a check of another serving | ack ≠ the node's | not applied, said in `health_note` |
| an `ended` check beside an unended serving | m11741's task | `unknown` with both facts; not counted in progress |
| approval wait | #13164 (result shown, served) | `awaiting agreement` segment and tag |
| delivery pending | #13162 (result naming Front, no served mark yet) | `waiting: not yet taken up by Front`; the same unit is in `trace.next_actions` |
| completion | #13189 | `completed`; `next_actions` empty and `stall_candidates` empty two hours later |
| an agreed task, plan not accepted | #13178 | `tasks_agreed` done, `plan_accepted` pending, card not completed |
| cancellation | task and mission `[state] cancelled` | plan `cancelled`, no stage owed, card `cancelled` |
| study stages | o11522 at #11708 | run ended / report delivered / knowledge refreshed pending; a `sagesync` note after the acceptance satisfies the last |
| held by a person | o11711 with Observer's hold | card `awaiting_you`; its task still `unknown` |
| stop then recovery | #13149 stopped check + open incident → `stopped`; #13156 resumed serving (new ack #13155) with the old check and a rescued incident | the old check is not applied; the resumed serving `working`; recovery `rescued` |
| rename + ✔ of the request | every origin post moved to a new name, then ✔ | same origin/anchor 13116, label follows, still `completed` |
| late result | autolab posts naming Front after the task is accepted | record still `accepted` (meter 1/1), unit and card `waiting` for the delivery |
| two requests, one agent | o11522 and o11711 (both autolab, both aisvgs) | their own plans and counts (1/1 vs 0/1) |

Relay side (`agentroom/tests/test_progress.py`, 9) covers the board: two
requests to one agent as two cards, probes once per serving per TTL and
within a budget, failing/slow probes → `unknown`, Observer's hold and
incident on the right card, the current conversation pinned even outside
the bounds, no Zulip call, a stale mirror, the cache invalidated by a new
post, and a route that writes nothing.

## Defects found and fixed at their owning layer

| defect | fix |
|---|---|
| a cancelled plan kept `tasks agreed`/`plan accepted` pending, so its request read `waiting` for ever | a cancelled plan owes no stage (pyagag `9f1f898`) |
| a request whose every unit was cancelled read `completed` | it reads `cancelled` (same commit) |

Both were caught by the new transition tests, fixed in `agag.progress`,
re-run with the whole pyagag suite (1006 passed), and deployed to the relay
(lock `9f1f898`, kickstarted).

## In the browser

`agdevworld/.local/pg/` (CDP driver on port 9337, vite on `:5179`):

- `s1-demo` (16 assertions) — meters, planning, revision, stale, stopped,
  completion beside a request that needs you, relay down (frozen, last
  board kept), reload, 390×800.
- `s3-switch` (7) — "open conversation" on another card switches the room
  (`?conv=demo-b`) and the pinned card follows; a reload keeps both; the
  history and the panel take turns in the right column; every chip has
  words.
- `s2-live`, `s4-restart` (live relay) — ten real cards; the relay
  kickstarted while the panel was open: the panel said "not answering —
  showing the last board, read 09:04:01Z (1 s ago); nothing below is
  current", kept all ten cards, stopped animating, and came back with the
  same request pinned.

## Passive viewing costs nothing and starts nothing

Measured on the deployed relay: 30 forced recomputations
(`/progress?fresh=1`, ≈1.5 s each) and the open panel polling every 8 s.

| measure | before | after |
|---|---|---|
| relay mirror Zulip calls | 6 | 7 (its own event-queue poll) |
| realm revision (newest message) | 13302 | 13302 |
| servings in agfront / agautolab / agforge / agobserver / archsage | 538 / 1154 / 50 / 704 / 14 | unchanged |

Health probes ran only for open servings of autolab (one: m11741's task),
at most once per 15 s.

## Observability gaps confirmed (not fixed here)

- **Only autolab is probed.** Front's routine runs and desk servings,
  archsage's and forge's are conversation-only; the panel labels them so
  and never animates them.
- **An open serving from before failsafe p1** has no `end=` and reads open
  for ever (both aisvgs routine runs, m11741's task). The panel shows them
  as `unknown` after 30 min without work.
- **Routine runs left unended** (G2) keep `run ended` / `report
  delivered` pending, correctly, with nothing in the system to end them
  from another conversation.
- **No determinate unit exists for a run** in any record; run bands are
  activity only.
