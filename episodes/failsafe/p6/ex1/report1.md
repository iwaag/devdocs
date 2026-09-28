# failsafe p6 ex1 — step 1: the gaps reproduced, the decisions defined

## Deployment observed (2026-09-28 02:05Z)

- `nctl status` ok (every submodule clean); `nctl drift` converged=46.
- Pins are p6's final set: pyagag `7690f13` in agfront, agautolab, agforge,
  agobserver, archsage, cagent and the relay; comfynotify on its old pin;
  agautolab1 (VM) not redeployed since before p6.
- Observer: `tracked.json` `{}`, `held.json` `{}` (unread since p6),
  `retired.json` holds o11450, o11522 and o14251.
- The panel (`/progress?fresh=1`) shows all three retired requests in the
  **active** group: o11450 `queued`, o11522 `unknown`, o14251 `queued`.
  Nothing on the cards mentions the retirement.

The replay kit is in ignored files under `pj-agdev/.local/failsafe-p6ex1/`:

- `fixture/mirror-before.sqlite` and `fixture/listener-before.sqlite`:
  `.backup`s of Observer's mirror and journal, newest message #15449 (20:39Z).
- Observer's `retired`, `tracked` and `held` files.
- `repro_citation.py` and `repro_scope.py`: the synthetic reproductions
  below, run against pyagag's test realm.
- `replay.py`: traces the legacy cases off a mirror copy.
- `dump.py`: prints one conversation.
- `classify_legacy.py`: sorts every root note by its conversation's real
  beginning.

## The citation gap

### Reproduced at 199, 200 and 201 posts

This is the p6 cleanup shape (`test_citing_another_request_during_cleanup…`).
A desk's run writes its root note into another request's conversation. That
conversation is padded to N posts, and the desk is then traced.

| cited conversation | history the trace reads | cited request adopted by the desk? |
|---|---|---|
| 199 posts | complete | no |
| 200 posts | the newest 200 | **yes**: the cited request, its mission and its task hang under the desk |
| 201 posts | the newest 200 | **yes** |

`_reference` returns false once `len(messages) >= HISTORY`. The traversal
then keeps the link as work. Raising `HISTORY` only moves the step.

### A bounded history missing its beginning (limit 40, below `HISTORY`)

The trace assumes any read shorter than 200 reached the beginning, so a
truncated read gets it wrong in both directions:

| case | result |
|---|---|
| (a) cited conversation whose visible window begins with the citer's own post | **adopted**: the ownership edge is manufactured |
| (b) a real task whose `[task]` note fell outside the window | **dropped from its mission's tree**: established work is silently erased |

### The same note also changes acceptance scope

`repro_scope.py` checks the citation's effect beyond the trace:

- **Acceptance.** `agag.acceptance.decision` climbs the requester's
  `effective_rootchat`. After the citation, the cited mission's acceptance
  holders become Front and the citing desk's requester (Developer). Before,
  they were Front and the Omni Agent. The real requester **loses** the
  decision, and the citing desk's conversation joins the acceptance scope.
- **Receipts.** `agag.receipt._decision_for` climbs plain `[rootchat]`
  notes. It never reads a move, and it treats a citation like a delegation.
- **Callbacks.** A reply naming Front in the cited conversation is served in
  the citing desk. That is the right **return address**, and it has to stay:
  a cleanup comment may need an answer.

So one note carries two meanings that must separate. Its return address is
correct. Its ownership meaning is only guessed, at read time, from a history
of whatever length the reader happened to get.

### Legacy records

The mirror holds every conversation completely: 776 of 776 coverage rows
are `complete`. Every root note can therefore be classified from the
conversation's real beginning:

| relation | notes |
|---|---|
| work: the conversation was opened for the work, carries an identity note, or was begun by the note's author | 534 |
| adoption (`[rootchat-moved]`) | 1 |
| reference: the conversation began as somebody else's request | 5 — #1574, #1752, #3033, #4825, #15357 |
| unknown: beginning not held | 0 |

- #15357 is p6's cleanup link.
- The other four are Front relaying into topics the Developer or the Omni
  Agent had begun. p6's full replay found two of them (2458 and 1550).

## The three retirements

Each case was traced off the mirror copy and read in full.

| case | request | what the records show | what "retired" meant | repository work |
|---|---|---|---|---|
| **o11450** | Developer's context trial #11450: "read the reference, and have forge make a **plan only**" | Plan a11459 recorded (`planned`, #11468) and verified. Front reported it (#11472) with forge's six questions "if you want generation", `response_request` to the Developer. The asset run r11466 was never started, as requested. The Developer ✔'d the desk on 09-27 without answering | the requested outcome was reached. The optional continuation (generation) was never taken up, and forge has no record that ends the plan | none: forge's topic workspace holds `plan.md` only, and nothing was generated |
| **o11522** | Omni Agent #11522: set up study aisvgs, run one bounded round, refresh the sage | Setup done. m11579 accepted (#11707). Sage refreshed to `13e0d6e` (#11694). #11699 also asked Front to close `routinerun-20260926-2100` with its report. Front declined (#11707: "the run has to post its own report"), and the run never got an end record. Its serving #11649 reads open, 37 h later | completed. The one open unit is a routine run with no `agrun finish` | none: `aisvgs` main `f57eed1` = `origin/main`, clean; `13e0d6e` is on main; the mission copies are gone |
| **o14251** | p5 trial E: run study-growbox once | Routine run finished, achieved (#14435, `agrun finish`). m14270 done and accepted. The stray post #14330 went into an unowned topic literally named `pj-growbox › work-m14270/workrun-task1-m14270`, and Front's correction #14410 withdrew it | completed. The stray unit is withdrawn and owes nothing | none: `growbox` main `9d34067` = `origin/main`, clean; `04b094c` is on main |

So every retirement meant **request ended, completed**, plus one leftover
unit per case:

- o11450: an offer nobody took up;
- o11522: a routine run with no end record;
- o14251: a withdrawn stray post.

None meant "keep watching less". The three leftovers need different
dispositions (step 4):

- o11450's forge plan: **withdrawn**, as not pursued;
- o11522's run: ended through the existing path, `agrun finish`;
- o14251's stray topic: **withdrawn**.

## The baseline, corrected

p6's report said "the panel does not read `retired.json`". It does:

- agentroom's `ObserverRecords.read` loads the file.
- `recovery_for` adds `{retired: True, retired_why}` to the root unit's
  recovery dict.

The gap is that nothing interprets it:

- `agag.progress._display` never looks at those keys, so the card keeps its
  trace state (`queued`, `unknown`) in the **active** group.
- Observer's `retired_now` means something else: "no incident and no
  tracking until **any** node's `last_activity` is newer than the
  retirement".
  - `last_activity` counts every message, selfnotes and receipts included.
    A bookkeeping-only write, such as a receipt repair, an `[owed]` note or
    a hold record in the tree, therefore silently ends the retirement.
  - A retirement is also all-or-nothing for the whole request. It has no
    unit, no decision maker, no evidence and no outcome.
- `[state] retired` is a third meaning. The trace reads it as `cancelled`
  (`CANCELLED_WORDS`).

So one word means three things in three readers, and the private file
carries none of what a shared record would need.

## Decisions for steps 2–3

### Relations (step 2)

- **Every root note states its relation.** The return address stays the
  note's home. Ownership becomes an explicit word on the note:
  - `[selfnote][rootchat] <channel>/<topic> #<anchor> rel=work`:
    delegation. The conversation is work done for the home, which becomes
    its requester, and it hangs under the home.
  - `rel=reference`: a citation or comment. Answers still come back to the
    home, but the conversation, its work and its waits stay their own
    request's.
  - `[rootchat-moved]` (`agentchat anchor`, `agrun adopt`): deliberate
    adoption, always work.
- **Writers.**
  - `agentchat send` decides the relation when it writes the note, from the
    conversation's **real beginning**, read by a targeted oldest-first read
    rather than the newest window:
    - **work** for a new or empty conversation, one the sender began, or one
      opened for work (an identity note, or a note first);
    - **reference** for a conversation that began as somebody else's
      request.
  - `--relation work|reference` overrides the default. The command prints
    the relation it recorded and how to change it.
- **Correction and inspection.** `agentchat relation` shows the relations
  in a conversation and which record decided each one.
  - Its correction writes `[selfnote][relation] #<note> <work|reference>
    — <why>`, accepted from the note's author or a person.
  - A move remains the way to change the home.
- **Legacy.**
  - One migration record classifies every root note written before the
    change. It lists the references and the unknown ones from the evidence
    above, up to the newest legacy note.
  - A note without a relation after that point is **unknown**: it comes
    from an unredeployed writer. So is a legacy note the record does not
    cover.
  - Unknown relations are listed on the trace and the card, with the
    command that resolves them. They never adopt, and they never hide the
    conversation's own work, which stays traced from its own request.
- **Readers.** The trace, callbacks, acceptance holders, receipt decisions,
  Observer and the panel all use the same `relation_of` reader. The
  history-length rule (`_reference`) is removed.

### Dispositions (step 3)

- **Two kinds**, recorded like holds in the request's origin conversation.
  A request's origin is never archived.

  ```
  [selfnote][disposition] <suppressed|completed|cancelled|withdrawn> a<unit> upto=#<id> by <user id> (<name>) #<evidence> — <why>
  [selfnote][disposition-reversed] #<disposition> by <user id> (<name>) #<evidence> — <why>
  ```

  - `suppressed`: **monitoring suppressed**. The work stays open and
    visible, and the card explains the suppression. Observer does not act
    on the covered obligations.
  - `completed` / `cancelled` / `withdrawn`: **request ended**. The unit
    and the unfinished work below it leave the active queue:
    - `completed` reads done;
    - `cancelled` and `withdrawn` read cancelled, never completed.

    Unfinished children are listed as ended with it, with the owners'
    existing paths (`agrun finish`, a cancellation) to reconcile their own
    records.
- **Scope and boundary.**
  - `a<unit>` is the covered unit and everything below it.
  - `upto=#<id>` is the newest **substantive** post in that scope when the
    decision was made. Substantive means speech: not a selfnote, not a
    system notice.
  - A later substantive post makes new obligations that the disposition
    does not cover, so they are monitored and shown. Receipts, notes, ✔
    and restarts are bookkeeping and change nothing.
- **Convergence.** Records are in the conversation:
  - a repeat finds the same record and writes nothing;
  - an interrupted write either exists or does not;
  - a restart reads the same records.
- **Retirement goes.** `retired.json`, `--retire`/`--unretire`,
  `retired_now` and the relay's `retired` keys are replaced by these
  records once the three cases are migrated.
