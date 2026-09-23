# Step 3 report — unfinished work is retained, and recovery is verified positively

Plan: [plan.md](plan.md) step 3. 2026-09-24 (JST), by the Omni Agent. Code and fixtures; the
listeners are restarted at the step 5 rollout.

## Retention is separate from discovery

| | Discovery (p1, unchanged) | Retention (new) |
|---|---|---|
| What | open `#front › front-…` conversations with a post in the last 12 h | every request the monitor has looked at that still has **unfinished work below it** (any conversation not `done`/`cancelled`) or an **open incident** |
| Keyed by | — | the origin's first post (`o<id>`, step 2) |
| Survives | — | inactivity past the window, a ✔ or rename of the origin, a restart (`.local/incidents/tracked.json`); a lost index is rebuilt from the open incident records, whose origins are ids |
| Let go | — | when nothing below it is unfinished and no incident of it is open; or when its origin no longer exists (open incidents are then reported, not dropped) |

The index holds only what the monitor has looked at. It is **not** a rescan of the realm, which
today would adopt 74 historical candidates (report1). A ✔ conversation judged a deliberate close
counts as finished by decision.

A ✔ origin is looked at like any other; its incidents take the path p1 wrote and could never
reach — "the conversation the request came from is ✔ closed, so there is nobody there to ask" —
and are reported to the realm's owners (R4).

## Recovery is a transition on record

When an open incident's candidate is not produced on a look, the monitor no longer concludes
anything from that. It reads the stalled conversation **by its anchor** in the fresh trace:

| The look shows | Outcome |
|---|---|
| the mirror is stale | nothing concluded, nobody asked; no new incident opened on a stale look |
| the conversation is missing from the trace, or unreadable (a child whose own root notes are indexed but whose history reads empty is `unobservable` — pyagag trace) | *cannot see* posted once; reported after 30 min (`UNOBSERVABLE_REPORT_SECONDS`) if it stays so; never rescued (R1) |
| `cancelled` / `replaced` / `retired` recorded | closed **cancelled** — a decision, not a recovery |
| the state this kind waited for: `unstarted`/`unacknowledged`/`failed` → acknowledged or further (`executing`, answered, `done`); `undelivered` → no longer awaiting delivery; `silent` → answered, or progress newer than the incident; `resolved_live` → no longer ✔ | **rescued**, with the state and its detail in the post |
| anything else — e.g. posted into but not yet overdue for acknowledgement | still open (`waiting: queued`), no request until a candidate says it is owed |

A reported incident whose work later moves is marked *moving again*; the same work stalling after
a rescue, a decision or such a clearance opens a **new episode** (`e2`, its own topic) instead of
being ignored for ever or re-asking a closed one. A reported incident is otherwise never re-asked.

**An answer is taken up only by a served mark** (pyagag `_taken_up`). The mark is the receipt
the requester's listener writes after a *delivered* serving, bound to the post that serving
processed. p1 also accepted "the requester spoke at home since", which let a reply to something
else consume an answer nothing had read (R8). An answer that arrives during a serving stays owed
until the listener's next pass marks it.

The p3 replay fixture recorded marks written by pre-journal code (task 2's mark names a progress
post, 8382; the answer naming Front, which that serving read, is 8386). The fixture is read with
each mark moved to the newest post naming Front before it — what today's writer records — and the
loader says so.

## Bounded and restart-proof

- The allowance of two requests belongs to the incident — `(request, stalled conversation)` —
  and survives a change of kind (step 2), a restart (the store) and a lost store (adopted from the
  incident's note, counting the requests on record).
- A request retained past the window after a restart gets its *second* request once, then the
  report; no fresh allowance (fixture below).
- **Cost trap found in the dry run and removed:** with retention, p3's ✔ twin (judged a deliberate
  close) would have been re-judged on the local model every hour for ever — 30–130 s of a
  single-threaded loop each time. A `resolved_live` judged `legit` is now re-judged only when its
  conversation has changed since the judgment.

## Fixtures

| Reproduction / case | Result |
|---|---|
| R1 unreadable target | passes: *cannot see*, no rescue |
| R3 13 h of silence with an incident open | passes: still looked at |
| R4 origin ✔ with work outstanding | passes: reported to the owners |
| R8 unrelated reply at home | passes (monitor) and in pyagag's trace |
| new blockage inside its grace | open, `waiting: queued` |
| cancelled task | closed *cancelled*, no rescue post |
| stale mirror | nobody asked, nothing posted |
| unobservable for 30 min | one *cannot see*, one report, no rescue |
| retained a day past the window, across a restart | second request once, then reported |
| nothing outstanding | let go from the index |
| stall again after a rescue | new incident, episode 2 |

agobserver 100 passed (18 in `test_p2_reproductions.py`, no `xfail` left); pyagag 752 passed;
agautolab 258, agforge 265, agfront 182 on pyagag `48b2879`.

## Dry run on the live realm

The new monitor ran one look over a copy of Observer's mirror with a client that sends nothing:

| Measure | Value |
|---|---|
| Look duration (no judgment) | 0.18 s |
| Requests in the 12 h window | 8 |
| Retained after the look | 7 — every one because a mission or task is `awaiting_requester` (missions are never marked done: p1's open mission-close item) |
| Would post | 3, all one incident: p3's ✔ twin under its new key, judged by the model again |

Two rollout consequences, taken at step 5: p1's incident store is keyed by names and is moved
aside (disposable; the incident topics stay the record), and the p3 twin is judged once more under
its new key. Retention grows by roughly one request per mission left unaccepted — about 20 ms per
look each, and no candidates — until missions record their close.

## Evidence rules as implemented

| Outcome | Evidence the monitor now requires |
|---|---|
| rescued | fresh look; the stalled conversation read by anchor; the state its kind waited for |
| cancelled | the owner's (or requester's) terminal `[state]` word in that conversation |
| unobservable | the conversation missing from the trace or unreadable; reported after a bound |
| different blockage | same anchor, different kind or a non-recovered state: same incident, same allowance |
| reported | requests spent, origin ✔ or gone, owner not answering, unobservable too long, or unclear twice |
