# Agent session p1 ex3 — step 4: behaviour evaluation and work accounting

Date: 2026-10-15. The evaluation tools and their output are in the ignored
notes `pj-agdev/.local/agent-session-p1-ex3/step4/`:

- `browser.cjs`: Playwright Chromium, the game served over HTTP so that
  `localStorage` has a real origin;
- per run: `probe*.js` and `steps*.js`, their outputs, the screenshots, and
  the decisions with their pause records;
- `corrupt-both.js` and its outputs.

Mechanism figures come from `step3/aggregate.py` and from ex2's `audit.py`
(output in `step5/`).

The evaluation used deterministic code only: no model call was made for
review. Each arm was judged from its own branch against the frozen
criteria. The agents' test claims were not used as evidence.

## Method

1. **The arm's own headless suite**: `node game/test/headless.js`. The
   baseline has 31 checks.
2. **Diff review** of `game/` against `7669fdb`.
3. **An independent probe per set.** These assertions are my own,
   accelerated with `RTS.step` and adapted only to the arm's API names:
   - A: refusals, spawn-to-rally movement, existing units unaffected,
     a changed point, a cleared point, restart;
   - B: ordered visits, replace, Stop, a dead unit, gather and attack
     still accepted;
   - C: a 4-minute two-bot match with research and a queue running →
     save → load:
     - compared state, rebuilt `byId` and restored bot state;
     - then 90 s of continued simulation compared with the original;
     - corrupt and incompatible inputs;
     - a save taken while a worker helps an unfinished building.
4. **A browser check** through the real UI: clicks, keys and
   `rtsDebug.fastForward`, with screenshots.
5. **The same checks for both arms of a set.**
   - In A, the in-game help was checked in both arms, because A's request
     asks for it.
   - In B, the request does not ask for help, so neither arm was judged on
     it.
   - In C, a corrupt-save battery was added during C run's evaluation. It
     was then run on C fresh's pre-repair commit (`49df865`), the state C
     fresh's own evaluation had judged, and all five cases were rejected
     there. Applying it earlier would not have changed C fresh's verdict.

## Quality

The final branches, after any repair:

| Criterion | A fresh | A run |
| --- | --- | --- |
| A1 command, owner/missing/non-production refusals | pass | pass |
| A2 trained unit reaches the point (probe 0 px; browser: right-click set → `Q` → worker at the flag) | pass | pass |
| A3 existing units unaffected, a change applies later only | pass | pass |
| A4 help and marker | marker pass; help **cut off at first evaluation** → repaired | the same (cut off → repaired) |
| A5 restart clears | pass | pass |
| A6 follow-up clear (`Z` + HUD button; next worker gathers) | pass | pass |
| A7 tests | 46 (+15) | 46 (+15) |

| Criterion | B run | B fresh |
| --- | --- | --- |
| B1 ordered visits (0, 1, 2), idle at the end | pass | pass |
| B2 plain move replaces, Stop clears | pass | pass |
| B3 dead unit with a queue | pass | pass |
| B4 route drawn for selected units | pass | pass |
| B5 other commands work; semantics stated | pass (README: any non-queued order clears the queue) | pass (README and in-game help) |
| B6 follow-up numbering | pass (1…n on the points) | pass (1…n plus a "残り N" counter) |
| B7 tests | 39 (+8) | 40 (+9) |

| Criterion | C fresh | C run |
| --- | --- | --- |
| C1 one slot, controls, versioned format | pass (HUD row, version 1) | pass (top-right buttons, version 1) |
| C2 round trip of time, ore, entities and ids, orders, queues, research and bot state | pass after repair; **the construction-helper save failed at first evaluation** | pass (the construction-helper case also passed) |
| C3 continued simulation identical to the original for 90 s; `byId` rebuilt | pass | pass |
| C4 corrupt or incompatible data rejected, live match unchanged | pass (all five corrupt cases rejected before and after the repair) | pass after repair; **partly corrupted saves loaded at first evaluation** (a crash after load, NaN ore) |
| C5 transient UI and timing reset | pass | pass |
| C6 browser reload then Load is exact | pass | pass |
| C7 follow-up saved time and disabled Load | pass | pass |
| C8 tests | 46 (+15) | 42 (+11) |

Every run is completed and accepted, and no original requirement was lost
through the follow-up or the repair.

**Defect pattern.**
- A was the same help-layout defect in both arms.
- In C, each arm had one different defect. C fresh's strict validator
  rejected a legitimate order shape. C run's lenient validator accepted
  broken data.

That is design variation, not a property of the session policy.

Two existing layout issues were not counted against either arm. Both are
also visible in the baseline game:

- the long key line of the help is cut off;
- the selected worker's Stop button overlaps the help text.

## Phases (reported cost, $)

| Run | Plan | Implement | Follow-up | Repair | Close-out | Total |
| --- | --- | --- | --- | --- | --- | --- |
| A fresh | 0.51 | 1.65 | 2.30 | 1.71 | 0.39 | **6.57** |
| A run | 0.58 | 1.19 | 1.69 | 1.85 | 0.69 | **6.00** |
| B run | 0.54 | 1.10 | 0.98 | — | 0.40 | **3.02** |
| B fresh | 0.60 | 1.68 | 1.01 | — | 0.43 | **3.71** |
| C fresh | 0.55 | 1.64 | 1.47 | 1.41 | 0.50 | **5.57** |
| C run | 0.56 | 1.76 | 1.67 | 1.39 | 0.70 | **6.08** |

Each implementation attempt also verified, presented and asked. The
follow-up and repair attempts each repaired, verified, presented again and
asked. The plan attempt is a new session in both arms.

## Per-attempt evidence (summary)

| Run | Sessions | Steps | Cache write | Cache read | Output | Attempt wall time | Context at each start (k tokens) |
| --- | --- | --- | --- | --- | --- | --- | --- |
| A fresh | 5 | 218 | 442 k | 18.9 M | 104 k | 1,890 s | 44 / 44 / 44 / 44 / 44 |
| A run | 1 (`e8b777b9`) | 153 | 187 k | 22.1 M | 83 k | 1,249 s | 44 / 95 / 126 / 165 / 201 |
| B run | 1 (`afafd9be`) | 99 | 122 k | 10.5 M | 44 k | 811 s | 44 / 86 / 115 / 136 |
| B fresh | 4 | 130 | 263 k | 9.9 M | 67 k | 1,015 s | 44 / 44 / 44 / 44 |
| C fresh | 5 | 196 | 367 k | 15.7 M | 97 k | 1,487 s | 44 / 44 / 44 / 44 / 44 |
| C run | 1 (`cac7b8d6`) | 140 | 208 k | 22.3 M | 78 k | 1,196 s | 44 / 95 / 162 / 197 / 221 |

- **Token counts** are from deduplicated assistant messages (input and
  cache) and from the result events (output).
- **A fresh's implementation attempt** ran one background subagent (a
  "visual verification" agent, 0.34 M cache read). Its process therefore
  emitted two result events. `total_cost_usd` is still the process total,
  so the cost is unaffected. ex2's `measure.py` reads only the last result's
  usage, so its token figures undercount such an attempt;
  `aggregate.py` does not.
- **Compactions**: none. The largest context was 228 k (C run's
  close-out).
- **Costs** are client-side list-price estimates on a Max subscription.
  They are not bills or quota.

## Mechanism: what each policy spent tokens on

| After the plan attempt | Fresh: A / B / C | Run: A / B / C |
| --- | --- | --- |
| Full-history reads (`wfe run show --history`) | 8 / 3 / 4 (45 k, 22 k, 44 k chars) | 0 / 0 / 0 |
| `plan.md` rereads | 3 / 2 / 1 | 0 / 0 / 1 |
| Definition reads | 3 / 1 / 3 | 0 / 0 / 0 |
| Help pages | 10 / 4 / 9 (24 k, 12 k, 20 k chars) | 4 / 3 / 3 (12 k, 8 k, 9 k chars) |
| Source-file reads, whole run | 19 / 9 / 21 (162 k, 142 k, 174 k chars) | 9 / 7 / 13 (76 k, 55 k, 122 k chars) |
| All tool output, whole run | 341 k / 222 k / 329 k chars | 137 k / 97 k / 183 k chars |
| Tokens appended at a resume | — | 0.8–1.3 k (continuation 1.7–2.1 k chars + the harness's own additions) |

- Each fresh attempt starts from the 29 k-char bootstrap and rebuilds its
  picture of the run: the record, the plan and definition, help pages, and
  the source it is about to change.
- A retained attempt starts from the previous attempt's context plus about
  1 k tokens. It reads no history and almost no documents again; its
  rereads of source are verification after edits.
- The retained arm's per-step context is two to five times larger, and
  each of its steps rereads that context from cache.
- In A and B the two effects roughly cancel, with fewer, cheaper steps
  winning slightly. In C the larger prefix won slightly.

This pair also confirms that ex2's continuation behaves the same on larger
tasks:

- every resume carried the answer's exact text;
- every resume acted on that text without a record read;
- graph revisions (follow-up and repair) were handled correctly in all
  three retained sessions.

The harness added nothing stale on resume this time; ex2 had seen a stale
background-task notice.
