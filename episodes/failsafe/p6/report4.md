# failsafe p6 — step 4: a complete lifecycle for human holds

## What a hold was, and why that failed

A hold was a line in Observer's private `incidents/held.json`: a request
key, a free-text why and a time.

- Observer skipped the whole request.
- The relay read the file and showed "held by a person".
- `python -m agobserver.hold --release` deleted the line.

Nothing named the decision the hold protected, so nothing could tell when
it was made. o11711's hold, "the Developer decides how to resume it",
outlived the resume (#15264), the acceptance (#15273/#15284) and the
integration (`f57eed1`). It ended at 18:43Z on 2026-09-27 when somebody ran
`--release` by hand. That call also removed o8512's hold, and it left no
record of who released either, or why.

## What a hold is now (`agag.holds`)

A record in the **request's own conversation**, which is the origin, and
the origin is never archived:

```
[selfnote][hold] <purpose> a<unit> by <user id> (<name>) #<evidence> — <why>
[selfnote][hold-release] #<hold> by <user id> (<name>) #<evidence> — <why>
```

| field | meaning |
|---|---|
| purpose | the decision it protects: `acceptance`, `resume`, `decision`, `indefinite` |
| unit | the anchor of the work it covers; the request's own anchor covers the whole request |
| by | the person who holds it; evidence is their post (0 when recorded in person at the operator's terminal) |
| why | their words |

### How a hold ends

It ends by exactly one of two things, both on record:

| purpose | settles by itself when | otherwise |
|---|---|---|
| `acceptance` | the unit's `[acceptance]` record, or its cancellation | released on the holder's words |
| `resume` | the unit's owner serves it again after the hold (a later ack that worked or answered), or the unit's record says finished or cancelled | released on the holder's words |
| `decision` | never | released on the holder's words |
| `indefinite` | never: kept on purpose | released on the holder's words |

- Settlement is **derived from the records** by every reader, so a retry,
  a restart or a second reader reaches the same answer, and no tool has to
  remember to release anything.
- The authorized paths settle the hold without extra steps: `agentchat
  accept`, autolab's cancellation, `agrun continue` or a resume post the
  owner serves, and completion.
- **Unrelated activity settles nothing** (tested): a new post, a ✔, a
  `[state]` word, work that began before the hold, another unit's
  acceptance.

### While a hold is in force

- It says what it waits for, in words. For example: "Developer's decision
  on how task 11741#1 goes on (its owner serving it again, or its end on
  record)". A hold whose unit is not in the trace says so, instead of an
  unexplained "needs you".
- It covers only its unit and what was opened below it:
  - Observer opens no incident about that work;
  - new work in the same request (another plan, a new question) is looked
    at as usual (tested: an owed answer in a second plan is still
    `undelivered`);
  - a hold on the request's anchor covers all of it.
- An intentional hold stays visible on finished work. The card is
  `awaiting_you` until the holder releases it.

### History

`Hold.history` holds the placement, then the settlement or the release,
each with its post id and words. `agentchat hold` prints it, and cards
carry it (`card["holds"]`).

## Tools

| who | command |
|---|---|
| any agent (Front) | `agentchat hold [<msg>]`: the request's holds, in force, settled or released, with history |
| | `agentchat hold <msg> --for <purpose> --unit <any message of the work> --evidence <their post> "…"`: places a hold. The holder is whoever wrote the evidence, never the caller, and never the agent doing the work. The unit must be in the request. A repeat writes nothing |
| | `agentchat release <hold> --evidence <their post> "…"`: releases only on the holder's own words. An already settled or released hold writes nothing |
| operator | `python -m agobserver.hold --list o<id>` / `o<id> --for … --by <user id> "…"` / `--release <hold> --by <user id>`: the same records, written with Observer's credential for a person recording in person |
| operator | `--retire` / `--unretire`: unchanged (failsafe p3; `retired.json`, lapses on new activity) |

Front's `front` and `desk` guides gained *When a person keeps a decision
for themselves*, next to the receipt section.

## Readers

- **Trace**: `hold` and `hold-release` are record tags, so the root node
  carries them.
- **Progress** (`agag.progress.card`):
  - `holds_of(result)` decides each hold;
  - a hold in force shows on its unit (or the root) as `awaiting_you`:
    "held by Developer for resume (why): waits for …". `next` is `you`
    when the viewer is the holder;
  - `card["holds"]` lists them all with history.
- **Observer**:
  - reads holds from the trace every look, and finds held requests by
    their `hold` notes in the mirror index, so tracking survives a lost
    `tracked.json`;
  - suppresses only the candidates a hold covers;
  - keeps a request tracked while a hold is in force;
  - no longer reads `held.json`, and says so where the constant is
    defined.
- **Relay**:
  - no longer reads `held.json`;
  - discovers held requests from the hold notes;
  - `observer_held` and the active group come from the card's holds in
    force;
  - the frontend needed no change: it shows the `observer_held` tag and
    the unit's reason.

## Tests

- pyagag `test_failsafe_p6.py`, +5:
  - a hold on acceptance shows what it waits for, is not duplicated, and
    settles with the acceptance (history held → settled; the card
    completes);
  - a hold on a resume ignores a post, a ✔ and a state word, and settles
    when the owner serves the work again;
  - an intentional hold stays on finished work, is refused a release on
    somebody else's words, releases on the holder's, and a repeat writes
    nothing;
  - a hold covers only its work, and a second plan's owed answer stays an
    `undelivered` candidate;
  - a hold on work outside the request, or on the working agent's words,
    is refused.
- `test_progress`'s held-request test now places a hold record.
- pyagag 1078 green.
- agobserver 176: two tests moved from `held.json` to records, one with
  an explicit release.
- relay 363: the hold test uses a record.
- agfront 193.

## Commits

- pyagag `d07b7d5`.
- agfront `4939026`.
- agdevworld `a42670a` (relay).
- pj-agdev `ac4173c` (agobserver).
- Deployment follows, before step 5 uses these tools on the real records.
