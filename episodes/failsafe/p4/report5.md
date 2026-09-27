# failsafe p4 — step 5: the two inaccurate task records, corrected

## What was wrong

| | task 1 completed | result | the mission's cancellation (legitimate) | written in error |
|---|---|---|---|---|
| m9349 (`workplan-search-icon`) | `[state] completed` #9367, 2026-09-23 22:03 | `## Result` #9366 | `[state] cancelled` #12603 (requester's cancel, #12599) | task 1 `[state] cancelled` #12602, 2026-09-27 04:40 |
| m9697 (`workplan-shortest-icon`) | #9724, 2026-09-23 23:17 | #9723 | #12608 (#12600) | task 1 #12607 |

Both errors came from the `cancel_tasks` defect, fixed in agautolab
`32ad9b9` during p3. Both work channels are archived. These tasks predate
mission copies, so they have no `[change]` notes; their completion is the
state note and the posted result.

## The mechanism

`python -m agautolab.correct_state <task anchor> --note <wrong note> --because "…"`
(agautolab `27b1bbe`) is a narrow operation:

- It refuses unless the record shows exactly this case:
  - the note is this agent's `cancelled`;
  - the state before it is `completed`;
  - no state and no speech of this agent came after it.
- It appends one more note of the kind every reader already reads (the
  newest `[state]` wins: autolab's `own_state`, `agag.trace`, Observer,
  the relay). That note is `[selfnote][state] completed`.
- Beside it, it appends `[selfnote][correction] #<wrong> state cancelled ->
  completed (as #<original>): <why>`, so the history keeps both the defect
  and the repair. Nothing is edited or deleted.
- It writes `completed`, this agent's own word, never `accepted` or `done`,
  which are the requester's. The mission's own state is not touched.
- Both notes are selfnotes, so nobody is served, nothing is integrated
  again and no worker wakes.
- An archived channel is unarchived for the two writes and archived again.

Tests: `tests/test_correct_state.py` (the case recognized and read back as
`completed`; five other shapes refused). agautolab 331 passed.

## Applied (07:59Z)

| | notes written | channel |
|---|---|---|
| m9349 task 1 (anchor #9352) | #13250 `[state] completed`, #13251 `[correction] #12602 … (as #9367)` | unarchived and archived again |
| m9697 task 1 (anchor #9700) | `[state] completed` and `[correction] #12607 … (as #9724)` | unarchived and archived again |

## Verification, including after a restart of autolab and Observer (07:59:46Z)

| reader | m9349 | m9697 |
|---|---|---|
| autolab (`read_mission` / `read_task`) | mission `cancelled`, task 1 `completed` | same |
| `agentchat trace` (#9345 / #9689) | mission CANCELLED, task 1 DONE `completed` | same |
| relay project room (`/projects`) | work `cancelled`; tasks: 1 finished, completed 1, cancelled 0 | same |
| Observer | tracked and held sets identical before and after (`o11711`, `o8512`); no incident, no request | same |
| autolab listener | no serving of either task since 2026-09-23. The startup recovery queued its usual index and served nothing there | same |
| channels | archived | archived |

The correction reopened nothing and recorded no acceptance. The missions'
legitimate cancellation still stands.

## Human choices left open, deliberately

- **o11711 / m11741**: held. The Developer decides to resume, take what
  exists, or cancel. The mission copy `missions/aisvgs/m11741` is kept.
- **o8512 / m8519**: held; task 2's agreement is the Developer's.
- **m6113, m7601, m7732**: their acceptance is pending the Developer.
- **Review topics** `review-autolab-stopped` (now 7 occurrences: p4 T1–T3
  are 5–7), `review-autolab-uncertain` and `review-front-unanswered`:
  their ✔ is the Developer's.

Correcting facts decides none of these.
