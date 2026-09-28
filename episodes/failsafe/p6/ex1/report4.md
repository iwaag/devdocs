# failsafe p6 ex1 — step 4: the existing cases migrated and settled

## Order of operations (2026-09-28 UTC)

| time | what | by |
|---|---|---|
| 02:40 | dry run of the legacy classification, each author's live identity, fresh mirror copy | Omni Agent (operator tool) |
| 02:47 | legacy relation records written, each with its author's own credential: Front #15450, autolab #15451, forge #15452, archsage #15453, Omni Agent #15454 | Omni Agent (operator tool) |
| 02:48 | pyagag `a81a2cc` locked and synced in agfront, agautolab, agforge, agobserver, the relay, archsage and cagent; every listener, the gateway, forge's service, cagent-api/-zulip and the relay kickstarted 02:48:37Z with nothing in flight | Omni Agent |
| 02:49 | request #15455 to Front: settle o11450, o11522 and o14251 | Omni Agent, standing in for the Developer |
| 02:50 | Front's report #15468 | Front |

The records were written right before the restart, so no note from old code
could fall after their `upto`. A check after the deployment found no root
note written since #15449. Old code ignores `[selfnote][relation]`, so the
minute between the records and the restart changed nothing for it.

## Relation migration

| author | legacy notes up to | reference | unknown |
|---|---|---|---|
| Front | #15425 | #1574, #1752, #3033, #4825, #15357 | none |
| autolab | #15409 | none | none |
| forge | #11465 | none | none |
| archsage | #11594 | none | none |
| Omni Agent | #7194 | none | none |

The mirror held every conversation from its beginning, so no relation was
left unknown.

## Replay of all #front requests

The replay covered all 135 requests of the mirror at #15449, read three
ways:

- old code (pyagag `7690f13`);
- new code without the records;
- new code with the records.

It was then repeated on the live realm after the settlement: 136 requests,
the records read from the realm.

| comparison | changed requests |
|---|---|
| new code **with** the legacy records vs old code | 1 |
| new code **without** the records vs old code | 114: every legacy delegation read `unknown` and adopted nothing. The explicit gap the records close, and why they preceded the deployment |
| live realm after step 4 vs new code before settlement | the three settled requests, plus the settlement request itself |

The one meaningful change is **o8512**: the acceptance holders of mission
m8519 (#8514) went from Front and **Developer** back to Front and **Omni
Agent**, who asked for it.

- p6's cleanup citation #15357 had made o11711's requester the holder
  through the acceptance chain.
- Owners, trees, callback destinations and card states are otherwise
  unchanged. The p6 structural rule had already dropped the four other
  citations, because their conversations are short.
- Callbacks follow the note's home whatever its relation. The trees are
  identical, so no callback destination changed.

After the settlement, the live replay shows **0** unknown relations, and
#8514's holders are [9, 15].

## The three legacy cases

Front settled all three in one serving: 88 s, 23 turns, $0.57. It used
`agentchat disposition` and `agrun finish`, on the Omni Agent's post #15455
as the evidence.

| case | recorded | card before → after | ended with it (remaining obligations) |
|---|---|---|---|
| **o11450** (context trial) | `completed a11450 upto=#11472` (#15457): "reading and plan-only reached, generation never asked for" | `queued` (active) → **completed** (recent) | forge's plan a11459 (awaiting requester) and its run r11466 (not started). Both read *cancelled, ended with it* and are never called completed. forge has no cancellation of its own, so its record keeps saying `planned`, and nothing more is owed for this request |
| **o11522** (study aisvgs) | first `agrun finish … --achieved` (end record, report delivered to the desk #15459, ✔), then `completed a11522 upto=#15459` (#15462) | `unknown` (active) → **completed** (recent) | none. The run reads finished by its own record. The two plain exchanges (archsage's study topic, autolab's setup plan) were answered and taken up, so they are not listed |
| **o14251** (growbox trial E) | `withdrawn a14329 upto=#14330` (#15463): "stray post #14330, corrected in #14410" | `queued` (active) → **completed** (recent). The rest of the request was already done by record | none |

- **o11522 needed both steps.** After `agrun finish`, the run was ended by
  its own record, but Front found the request still not complete: its two
  plain exchanges read `awaiting_requester` (#15468). Front then recorded
  the request completed, as #15455 asked for that case.
- **The boundary covers Front's own report.** o11522's `upto` is #15459,
  the close-out report that `agrun finish` had just delivered, so the
  decision covers it.
- **Observer.** `tracked.json` is `{}`, and no incident was opened for any
  of them.
- **Unfinished repository work** (checked before the request):
  - `aisvgs` main `f57eed1` = `origin/main`;
  - `growbox` main `9d34067` = `origin/main`;
  - both are clean with no mission copies, and forge's kero workspace
    holds plan files only.

## Retirement removed

- `agobserver/.local/incidents/retired.json` is deleted. A copy is in the
  replay kit, `pj-agdev/.local/failsafe-p6ex1/fixture/retired.json`.
- No reader has used it since step 3.
- `agobserver.hold --retire/--unretire` exit and name
  `agobserver.disposition` instead.

## Holds and receipts preserved

- The p6 hold records are untouched and still read settled on every reader:
  - o11711's #15378;
  - the nine reconciled receipts #15379–#15387.
- **o8512's hold backfill** is documentation debt, not an open approval.
  - The hold it would record ("accept after reviewing task 2", 2026-09-27)
    was settled by the acceptance #15318/#15323 before it was ever a
    record.
  - p6's report documents it. The auto-mode classifier refused a record
    attributed to the Developer without their words.
  - Nothing was invented and no live hold was created: o8512 has no hold
    record, and none is owed. The Developer can write it from their own
    terminal with p6's command if they want it on record.

## Commits

- **Pins**:
  - agfront `7b04c33`, agautolab `916eba1`, agforge `656618e`, agdevworld
    `8f49a61`, archsage `87eeb7a`;
  - pj-clusterintent `e7115f3` (cagent);
  - pj-agdev `62b5ac6` (agobserver lock, submodule pointers).
- pyagag `a81a2cc`: a completed plain exchange is not listed as ended with
  a request. The o11522 simulation showed that noise before the live
  settlement.

## Interventions

- **Omni Agent as the operator** wrote the five legacy records with each
  author's credential through `python -m agag.relations legacy`. This is a
  one-time migration.
- **Omni Agent as the Developer's stand-in** posted #15455. The Developer's
  delegated cleanup authorization covers it. Front's dispositions record
  the Omni Agent as the decision maker.
- **Front** did the settlement itself, with ordinary tools.
- No hand edits of records were made.
