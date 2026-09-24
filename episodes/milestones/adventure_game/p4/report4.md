# p4 step 4 — Measure, classify, decide the loop

2026-09-24, after the mission's close (14:00:28 UTC). Sources: run records under
`.local/agent/<role>/` diffed against step 1's counts, the mirror, `agentchat trace 11283`,
relay `/healthz` and the Front Desk close-plan preview (read-only `GET`).

## The table

| | p3 (2026-09-23) | **p4** (2026-09-24) |
|---|---|---|
| Request | locations + a forge-made location (two requests, forge + autolab) | respawn biome select scene (one request, autolab only) |
| Entrance | Omni Agent relays into `#front` topics | Developer, Front Desk conversation, one line |
| Request → last integration | 1 h 48 min, incl. a 24-min stall | **13 min 31 s** (3 min 57 s of it the Developer reading the plan) |
| Request → mission `done` on record | Omni Agent backfilled acceptances | 45 min 55 s — 31 min of it waiting for the Developer's play check |
| Runs / cost | 79 / $16.56 | **27 / $3.86** |
| — by role | Front 50 / $6.27, autolab 26 / $9.71, forge 3 / $0.57 | Front desk 13 / $1.56, Front memo rendering 6 / $0.45, autolab superdirector 2 / $0.33, supercoder 6 / $1.53 |
| Model | Sonnet 5 | Sonnet 5 (every run) |
| Tasks | — | 3, each accepted once, none returned |
| Stalls | 1 (24 min, found by the Omni Agent) | **0** |
| Observer incidents | — (monitor did not exist) | **0**; request tracked 5 → 6 while open, released to 5 after `done` |
| `[opfail]` / faults / returned changes | — | 0 / 0 / 0 |
| Receipts | — | Front's `[served]` marks on every autolab callback it took (#11315, #11331, #11341, #11355, #11366, #11380, #11392, #11404, #11408) |
| Acceptances by the Omni Agent | 9 | **0** |
| Relays by the Omni Agent | 5 | **0** |
| **DEM events** | 20 (5 relays, 9 acceptances, 2 review corrections, 1 repair, 3 start nudges) | **0** |

Mission close record (14:00:28, Front, on the Developer's #11410 "checked it, no problem; please
resolve the related topics and clean up"): `[state] accepted` on each task (#11412–#11414),
`[selfnote][acceptance] #11410 by 8 (Developer)` (#11415), `[state] done` (#11416), workplan ✔.
Trace at 14:03Z: mission `DONE`, tasks `DONE/accepted`. autolab released the mission copy
(no worktree under `.local/missions/protoprey/`, branch `autolab/m11294` kept at `3bf3e90`,
project `main` clean at `origin/main`).

Left open by design: the Front Desk topic itself and the `#work-m11294` channel. The close-plan
preview says both are `ready` for the Developer's `finish ✔` button (2 ready, 5 already done);
Front's "all related topics are resolved" (#11418) is true of the topics it names.

## Classification

| Class | Count | Note |
|---|---|---|
| rescue follow-up | 0 | |
| missed completion record | 0 | Front ran `agentchat accept` itself on the Developer's word |
| change contamination | 0 | only `autolab/m11294` commits entered `main`, all fast-forward |
| duplicate work | 0 DEM; 1 observed | one redundant Front serving on task 3's two callbacks (#11406, ≈$0.12) |
| silent stall | 0 | |
| returned change | 0 | |
| false claim | 0 | Front's stale "not integrated" (ex1) did not recur; every claim matched the notes |
| missing tool/permission | 0 | |
| other | 0 DEM; 1 observed | a conditional acceptance (#11388) closes the task without the acceptor seeing the result of its conditions — only `VERIFY.md` changed after it this time |

## Verdict: **stable**

The rule stated in the plan before the numbers: zero DEM events on the normal path, every stall
found by Observer and recovered in-system, mission `done` recorded without a human follow-up.

- Zero DEM events: yes.
- Stalls: none happened, so the second clause holds vacuously — **Observer's detection was not
  exercised by this request**.
- `done` recorded by Front from the Developer's own acceptance post; the Developer's message was
  a requester's act, not a follow-up chasing a missed record.

What the verdict does not show: this was the easiest shape a request can take — one agent
(autolab) behind Front, three tasks, no forge, no long job, no callback from a notifier, no
returned change. The Developer also delegated task acceptance to Front (#11317), so the
Developer-in-each-task path the plan described was not exercised either.

## Next episode: **p4/ex1**

Stable → the next request, measured the same way. Exit condition progress: **1 of 3** consecutive
zero-DEM requests. Recommended request for ex1: one that puts a second agent in the loop, since
p4 did not — p3's scale test (several locations from one forge request, integrated by autolab)
or the Developer's next `todo.md` entry if it needs forge. The two observed candidates (redundant
serving on a double callback, conditional acceptance without a second look) are watched there,
not fixed now.
