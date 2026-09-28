# failsafe p6 ex1 — report: explicit work relations and shared retirement decisions

## Outcome

| completion condition | result |
|---|---|
| the required-results table passes | ✓ all seven rows by tests on pyagag `a42c711`, and all seven live except deliberate adoption, which is covered by tests only (report5) |
| the three legacy retirements have explicit dispositions | ✓ o11450 **completed**, with forge's plan and run ended with it. o11522 **completed**, after its run was ended with `agrun finish`. o14251: its stray unit **withdrawn**, and the request completed by its own records. All were recorded by Front on the Omni Agent's post #15455 (report4) |
| readers agree after restart | ✓ trace, panel and Observer were identical before and after two restarts: during L1's suppression, and over all seven cases of the phase |

Step reports:

- `report1.md` — reproduction and decisions;
- `report2.md` — relations;
- `report3.md` — dispositions;
- `report4.md` — migration and legacy settlement;
- `report5.md` — validation and deployment.

## Relation semantics

- **The return address and ownership are separate facts.** A root note's
  home is always where answers go. What the conversation *is* to that home
  is recorded, not guessed:
  - **work**: a delegation, or a deliberate move (`[rootchat-moved]`). It
    adopts the conversation's work, waits, acceptance holders and receipt
    decisions.
  - **reference**: a comment or a citation. It adopts nothing. The home
    lists it ("cites …"), and replies still come back to the home.
  - **unknown**: nothing records it. It adopts nothing, and it is listed
    with the command that resolves it. Missing evidence never makes an edge
    and never hides a conversation's own work.
- **Where a relation comes from**:
  - the note's own word (`rel=`);
  - the author's newest correction (`[selfnote][relation] #<note> …`);
  - the author's legacy record (`legacy upto=#… reference=… unknown=…`).

  Readers ignore how much history they read. p6's first-post rule is
  deleted.
- **Defaults**:
  - writers that open a conversation for work state `work`;
  - `agentchat send` decides from the conversation's real beginning, read
    oldest-first, and says what it recorded. A comment in somebody else's
    request becomes `reference` by itself, as in L2;
  - `--relation` overrides the default;
  - `agentchat relation` inspects and corrects.
- **Shared reader**: `agag.relations`. The trace, the acceptance holders,
  receipt decisions, and through the trace the panel and Observer all use
  it.

## Disposition semantics

- **Record.** `[selfnote][disposition]
  <suppressed|completed|cancelled|withdrawn> a<unit> upto=#<id> by <decision
  maker> #<evidence> — <why>`, in the request's origin conversation, plus
  `disposition-reversed`.
  - **monitoring suppressed**: the work stays open and visible, and the
    card explains why. Observer does not act on it.
  - **request ended**:
    - `completed` reads done;
    - `cancelled` and `withdrawn` read cancelled, never success;
    - unfinished work below reads *cancelled, ended with it* and is listed
      with its owner's own record, so the owner's path (`agrun finish`, a
      cancellation) can reconcile it;
    - a missing receipt under it is settled bookkeeping.
- **Boundary.** A decision covers what was substantive when it was made.
  - A later post in a covered conversation uncovers that conversation, so a
    new request, question or result is shown and monitored.
  - These do not uncover anything:
    - selfnotes (receipts, served marks, holds, these records);
    - ✔ and other notices;
    - restarts;
    - the recorder's own reply closing the serving that recorded it.
- **Convergence.** A repeat of a decision still in force writes nothing,
  and so does a second reversal. Readers derive everything from the
  records, so a restart reads the same state.
- **Tools**:
  - `agentchat disposition` for agents;
  - `python -m agobserver.disposition` for the operator, with `--by` in
    person.

  These replace `retired.json` and `agobserver.hold --retire/--unretire`.

## Migrations

- **Relations**: one legacy record per author, #15450–#15454, written with
  each author's own credential. Each was classified from the conversation's
  complete beginning in the mirror:
  - 534 notes are work;
  - one is a move;
  - five are references: #1574, #1752, #3033, #4825 and #15357;
  - none is unknown.

  The records were written immediately before the deployment, so no
  old-code note fell after them. Without them, 114 requests would have read
  unknown.
- **Replay** of all `#front` requests: the only change in owners, callbacks
  or scopes is o8512. Its mission m8519's acceptance holders went back from
  Front + Developer to **Front + Omni Agent**, its real requester. p6's
  cleanup citation had moved them.
- **Retirements**: the three became dispositions, and `retired.json` is
  deleted. A copy is in the replay kit.

## Validation

- Tests on the final pin:
  - pyagag 1108 (+29 in `test_failsafe_p6ex1.py`);
  - agobserver 177;
  - relay 363;
  - agfront 193;
  - agautolab 331;
  - agforge 265;
  - archsage 39;
  - cagent 204.
- p6's checks are unchanged and pass: unreceived results, result-bound
  acceptance, receipt recovery, holds, and the archived-source listener
  timing.
- Live: the step-4 settlement and trials L1–L3 (report5).
  - Front used every new tool without guidance beyond its guides: `send`'s
    reference default, `agentchat disposition` of all four kinds,
    `agrun finish`.
  - Observer posted nothing during the trials.
- Two defects found live were fixed:
  - the card named the owner, not the requester, as owing an agreement;
  - a closed plain exchange read `resolved_live`.

## Interventions (not autonomous agent recovery)

- **Omni Agent as operator**: the five legacy relation records, the
  deployment and restarts, and the deletion of `retired.json`.
- **Omni Agent as the Developer's stand-in**: requests #15455, #15473,
  #15542 and #15569, and the confirmations #15478, #15547 and #15574. The
  Developer's delegated cleanup authorization covers the legacy settlement.
  Front's records name the Omni Agent as the decision maker.
- **The Developer account**: #15580 confirmed L3 after Front refused the
  stand-in's confirmation. The L3 cancellation is recorded as the
  Developer's. *Corrected in p6 ex2:* #15580 was posted through the API
  client (`Python-urllib`), not a Zulip UI, so the record evidences the
  Developer **account**, not a human typing; it was not "in person".
- **Front and autolab** did all the record-keeping in the trials and the
  settlement, with ordinary tools. No record was edited by hand.
- **Cost**:
  - the settlement: Front $0.57;
  - L1–L3: Front $3.03 over 26 runs, memo renders included, and autolab
    $0.78;
  - steps 1–3 ran no model.

## Limitations

- **Deliberate adoption was not exercised live** in this phase. Its path
  (`[rootchat-moved]`, `agrun adopt`) is unchanged from p5/p6 and is
  covered by tests.
- **Front's harness memory** (`feedback_confirm_before_contacting_agents`,
  2026-08-20) makes Front ask for the Developer's own go-ahead before
  contacting another agent. It was applied inconsistently to the stand-in:
  accepted in L1 and L2, refused in L3. Keeping or changing it is the
  Developer's call. One of its questions (#15576) was addressed to Front
  itself.
- **An old writer's notes read unknown.** A root note written without
  `rel=` after the legacy records, by an unredeployed writer, is visible,
  not silent. Two writers are excluded from the deployment:
  - comfynotify keeps its old pin, and writes no root notes;
  - agautolab1 (VM) runs only its HTTP gateway, with no listener and no
    root notes. It was checked and deliberately not redeployed. A listener
    placed there must be redeployed first.
- **Relation corrections count only from the note's author**, and a legacy
  record classifies only its sender's notes. A person who wants a relation
  changed asks the agent that wrote the note.
- **Old incident records are untouched.** o11522 and o14251 keep incident
  records from p3/p5 in `reported`/`dismissed` states. The requests read
  ended on every reader. The records are history, pruned by their TTL.
- **o8512's hold backfill** remains documentation debt (p6 report): the
  hold was settled by #15318/#15323 before it was ever a record. Nothing
  was invented and no hold is open.

## Where it lives

- pyagag:
  - `e72b6a2` (relations);
  - `4727dd4` (dispositions);
  - `a81a2cc`, `559f206`, `a42c711` (fixes from the migration and the
    trials).
- Observer: pj-agdev `9ad3e64` (dispositions; `agobserver.disposition`).
- Relay: agdevworld `0720880`.
- agfront guides: `75d739f` (relations), `93a133c` (dispositions).
- Final pins:
  - agfront `56369f3`, agautolab `1c09364`, agforge `18ab361`, agdevworld
    `bc1e3d9`, archsage `7c46dd7`;
  - pj-clusterintent `491ac0e`;
  - pj-agdev `24eccbc`.
- README_DEV *Work relations and dispositions*.
- Host notes in `pj-agdev/.local/devenv.md`.
- Replay kit in `pj-agdev/.local/failsafe-p6ex1/`.
