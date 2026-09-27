# failsafe p6 — report: completion, receipts and human holds reconciled

## Outcome

| completion condition | result |
|---|---|
| the required-results table passes on the final implementation | ✓ all seven rows by tests on pyagag `7690f13`. Four also live: the nine legacy repairs, the interrupted-receipt trial, the archived source (#8557), the settled hold and the dropped citation (report6) |
| the legacy cases have evidence-backed dispositions | ✓ m8519 accepted; m11741 integrated with its sage refresh bound; m8823, m9017 and m8741 accepted; m9349 cancelled. Nine receipts reconciled on the decisions that settled them. Both holds settled on record (o11711's history recorded; o8512's in this report only, see Interventions) |
| live readers agree | ✓ `agentchat trace`, the panel and Observer read the same state for every case, before and after a restart. Observer tracks nothing false, and the panel shows no false wait |

Step reports:

- `report1.md` — reproduction and causes;
- `report2.md` — semantics;
- `report3.md` — receipt repair;
- `report4.md` — holds;
- `report5.md` — reconciliation of the existing requests;
- `report6.md` — validation and deployment.

## Adopted semantics

- **Four separate facts**: execution record, requester decision, receipt,
  open question.
  - An answer without a receipt is owed, whatever its producer's record
    says.
  - It is **settled** by a decision recorded after it that covers it:
    - the requester's `accepted`/`done` in the unit;
    - `[change] accepted … +shown=` (up to the shown result);
    - the mission's `[acceptance] … after=`;
    - a cancellation above it.
  - Settled work reads finished. The missing receipt stays visible as
    bookkeeping (`Node.receipt`, a card's `settled_receipts`).
  - A later answer, another request's decision and a ✔ settle nothing.
    Speech is no receipt for any reader.
- **One interpretation.** The panel, Observer and `agentchat trace` call
  the same trace with the same arguments. Observer's private leniency
  (`receipts_from`) is gone.
  - A unit's requesters are every author of its root notes, so the answer
    does not depend on where a trace starts.
  - Every answer above the served mark is checked, not only the newest.
- **Citations adopt nothing.** A root note written into a conversation that
  began as somebody else's request does not make that request, or its
  waits, the writer's. Deliberate moves still adopt.
- **Receipts are repairable from evidence** (`agentchat receipt`):
  - the listener's `[served]` mark when the caller's journal shows a
    delivered serving was given the answer;
  - otherwise a reconciled receipt (`[selfnote][receipt]`) naming exactly
    that answer and the decision or post it rests on;
  - otherwise nothing.
  - Only the owed agent's own receipts count, and the listener skips
    reconciled answers.
- **Holds are records** in the request's conversation (`agag.holds`):
  - each names its purpose, unit, holder and evidence;
  - `acceptance` and `resume` settle by the path their purpose names;
  - `decision` and `indefinite` end only on the holder's words;
  - a hold in force covers only its work, says what it waits for, and
    keeps its history;
  - tools: `agentchat hold/release`, and the operator's
    `agobserver.hold`.

## Proven causes

| symptom | cause | evidence |
|---|---|---|
| seven answers never received (m8519, m8741, m8823, m9017, m9349, 2026-09-23) | the p1 listener's ✔ race: the closing report arrived with its ✔ during or right before a serving; the serving marked only its trigger; the re-look by name said "nothing owed; skipped" | Front's listener log for every one (report1). Fixed since 87ac87e / 61df5ee; today's queue keeps the newest trigger |
| panel "waiting" vs Observer silent | Observer alone applied `receipts_from`; neither read the requester's decisions | replay under both rules (report1) |
| `trace 8519` DONE vs `trace 8512` AWAITING_DELIVERY | requesters known only from root notes whose home is inside the traced tree | report1 F3 |
| m8519's wait inside o11711 | Front's cleanup post into m8519's request wrote a root note naming o11711 (#15357) | report1 F4 |
| Front could not repair | no receipt tool; the hand-written marker (#15362) was unparsable; unresolve of the archived channel returned HTTP 400 | report1 F5 |
| a hold outlived its purpose | `held.json` recorded no purpose or settlement; release was a manual file edit with no history | report1 F6 |
| "not yet taken up by autolab" | the panel took the actor from root-note authors, not `owed_to` | report1 F7 |

The resolve/archive race was a hypothesis at the start. It is now proven
for the ✔ part. The archive played no part in losing the receipts: it
only blocked Front's unresolve.

## Repaired records

- **Receipts**: nine reconciled by Front with `agentchat receipt --repair`
  (#15379–#15387), each on the decision that settled it: Front's
  `accepted` notes #15321, #9954 (two), #9939, #9940, #9944–#9946, and the
  cancellation #12603. No work was rerun or re-accepted, and nothing was
  posted in a work conversation.
- **Holds**:
  - o11711's history was recorded as hold #15378 (`resume` of task 11741#1),
    which every reader shows **settled** by #15283;
  - o8512's (`acceptance` of m8519) is settled by #15323 and documented
    here.
- **Links**: #15357 stays in the record. It is read as a citation, so
  o11711's tree is its own work again.
- **Unfinished repository work**: none.
  - `aisvgs` main is at `f57eed1` and `localize` at `31c0ba1`, both on
    `origin/main` and clean.
  - The mission copies were removed at close-out.

## Validation

- Tests:
  - pyagag 1079, of which +26 are p6's (`test_failsafe_p6.py` 25, plus
    listener interruption tests) and updated identity/progress tests;
  - agobserver 176;
  - relay 363;
  - agfront 193.
- A replay of all 133 `#front` requests found:
  - the intended changes;
  - three old requests whose real unaccepted answers Observer had hidden;
  - two dropped citations.
- The live trial (m15405) covered:
  - ordinary completion through Front, with resolution;
  - an interrupted receipt: the listener was killed after the delivery,
    and the restart wrote receipt #15438 with no rerun and one report;
  - a restart comparison across trace, Observer and the panel, with every
    reader unchanged.

## Interventions

- **Omni Agent as the Developer's stand-in**:
  - the two requests to Front (#15376 repair, #15392 trial) and one
    confirmation (#15397);
  - resolving both conversations.
- **Developer-side repairs (not agent successes)**:
  - the code and deployment;
  - the backfilled hold record #15378, written with the operator tool in
    the Developer's name, in person, and marked as a backfill.
- **Refused and not pursued**: the o8512 hold backfill. Claude Code's
  auto-mode classifier refused a record attributed to the Developer without
  their words. The Developer can write it from their own terminal if they
  want it on record:
  `python -m agobserver.hold o8512 --for acceptance --unit 8519 --by 8 "<why>"`.
  It would read as settled by #15323.
- **Before this phase**, by the Developer and Front on 2026-09-27: m8519's
  task-2 acceptance, the m11741 continuation, the sage refresh, and the
  manual `held.json` release at 18:43Z.
- **Cost**:
  - step 5, one Front serving: $0.53;
  - the step-6 trial: Front $1.06 and autolab $0.31;
  - steps 1–4 ran no model.

## Limitations

- **Retirement is still Observer's file.** o11450, o11522 and o14251 were
  retired in p3/p5 ("nothing owed"). The panel does not read
  `retired.json` and shows them `active` with their real next actions:
  - forge's plan awaiting its requester;
  - a routine run never ended with `agrun finish`;
  - a stray post.

  Closing them is the Developer's call.
- **Archiving was not exercised live** on new work: autolab archives only
  cancelled missions, and the Omni Agent may not archive. It is covered by
  the listener test and by the live repair in `work-m8519`.
- **Reconciled receipts do not repair older, never-given answers under
  them by themselves.** Each answer needs its own repair, which the tool
  lists (m9017's #9039 was found this way).
- **The reference rule is structural.** It needs the conversation's
  beginning. A citation into a conversation longer than the trace's
  history (200 posts) is still adopted.
- **Legacy acceptance notes without `after=`** cover answers up to their
  evidence post only. m8823 task 3's close-out needed the requester's
  task-level `accepted` to settle.
- **Pins**: comfynotify keeps its old pin; agautolab1 (VM) was not
  redeployed.

## Where it lives

- pyagag:
  - `dad4ced` (settlement, citations, one rule);
  - `fb76e2b` (`agentchat receipt`, listener);
  - `d07b7d5` (holds);
  - `7690f13` (trial fault).
- agobserver:
  - `0dc798d` (no `receipts_from`);
  - `ac4173c` (holds from records);
  - `f4f6579` (CLI).
- relay: `a42670a`.
- agfront guides: `3207a39` (receipts), `4939026` (holds).
- Final pins: `c761a86`, `5339f6e`, `b042c9c`, `2c4e78d`, `1815459`,
  `29b55cc`; pj-agdev `ab0d251`.
- README_DEV *Completion, receipts and holds*; host notes in
  `pj-agdev/.local/devenv.md`; replay kit in `pj-agdev/.local/failsafe-p6/`.
