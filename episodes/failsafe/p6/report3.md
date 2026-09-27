# failsafe p6 — step 3: receipt recovery completed, and a repair operation

## The gap in ordinary recovery

The ordinary path was already complete for everything since p3, and the
tests say so:

- `agag.listen` writes the receipt after a confirmed delivery, on both the
  owner and the mention route (`_after_delivery`, `_mark_inputs`,
  `_mark_owed`).
- After a restart, `_receipts_pending` finishes a delivered serving's
  receipts before judging anything else.
- The mention route judges its trigger by id.
- The queue keeps the newest trigger of an entry re-armed during a serving.

Step 1 proved that the seven missing receipts come from the pre-87ac87e
race. Adding another retry mechanism would have fixed nothing.

What the plan's cases still exposed:

1. **No way to finish a receipt that ordinary recovery can never reach.**
   An answer from before the listener's ✔ horizon, or one whose serving
   predates journaled inputs (Front's serving 109 for #8557 records only its
   trigger, #8553), stays unreceived for ever. Front's only tool was a
   hand-written line (#15362), which no reader parses.
2. **A per-answer receipt had no place in the "newest answer" reading.**
   The trace looked only at the owner's newest answer. A receipt that
   covers exactly one answer (step 2's reconciled receipt) would therefore
   have hidden an older answer that no serving was ever given. The trace
   now walks every answer above the served mark, newest first, and skips
   only the ones reconciled. Replayed over all 133 `#front` requests, this
   changes nothing on the realm as it is.
3. **The listener would have bought a run for a reconciled answer.**
   `unanswered_mention` and the trigger-by-id check only knew served marks.
   A reconciled answer above the mark and past the horizon would have been
   served again for "nothing new". `Listener.reconciled()` now reads the
   agent's own `[selfnote][receipt]` notes from the mirror index, and both
   checks skip those ids. A newer post naming the agent is still served.

## The repair operation: `agentchat receipt`

`agag.receipt`, exposed as `agentchat receipt <answer id> [--repair]
[--because <post>]` for the agent that runs it. No new grant is needed:
Front's roles already allow `Bash(agentchat:*)`.

**Inspection** (writes nothing) reports:

- the answer: where it is now, its author, whether it names the caller;
- the receiving agent;
- the **destination**: the caller's home for that conversation, found with
  the callback's own lookup (`rootchat_home`: own root note, else the
  replaced or parent conversation's), with its live, ✔ or renamed name;
- the receipt state: `RECEIVED` (a served mark covers it), `RECONCILED`,
  `MISSING`, or `NOT_OWED` (not a callback of the caller's, or not naming
  it);
- earlier posts naming the caller above the mark, and later ones (left
  untouched);
- the **evidence**, strongest first:
  1. **journal** — the caller's listener journal (`AGENTCHAT_JOURNAL`, now
     set for every run by `agag.agent.chat_environment`) shows a delivered
     serving triggered by the answer, or handed a span of that
     conversation holding it;
  2. **decision** — a decision recorded after it covers it (step 2's
     `trace.decisions`, in the answer's conversation and up its owner's
     root notes);
  3. **`--because`** — the caller's own later post that took it up.
     Refused if the post is not the caller's or not later than the answer.

**`--repair`** writes one selfnote into home, under its live name:

| evidence | note | coverage |
|---|---|---|
| journal, and every earlier post naming the caller above the mark was also given to a delivered serving | `[served] <remote> <answer>`, the mark the listener would have written | up to the answer |
| a decision, `--because`, or journal evidence with an earlier answer nobody was given | `[selfnote][receipt] #<answer> by #<evidence> (<accepted\|cancelled\|relayed\|given>) in <remote>` | exactly that answer; claims no serving |
| none | nothing. It says what would count: read the answer and deal with it, and the listener writes the receipt | — |

Properties:

- **A repeat converges.** A second run finds `RECEIVED`/`RECONCILED` and
  writes nothing.
- **An interruption leaves one note or none.**
- **Nothing is served.** A selfnote buys nobody a run.
- **An answer that arrived after the repaired one stays owed**: marks are
  bounded by the journal's spans, reconciled receipts by their one id.
- **Stable references.** Answers are matched by id, not name, and must be
  in the conversation the note names. Only the owed agent's own notes
  count.
- **Archived sources are fine.** The note goes into home, so an archived
  source channel (m8519's `work-m8519`, where Front's unresolve failed with
  HTTP 400) is never written to.

## Guidance

- `agentchat --help` has the example.
- Front's `front`, `desk` and `routine_run` guides have *When an answer
  reads as not taken up*:
  - what AWAITING_DELIVERY and "no receipt … settled by" mean;
  - the command, and that a repeat writes nothing;
  - that a receipt changes no work and needs nobody's approval;
  - not to write a receipt line by hand.
- The guides do not use the word "selfnote": an agent that sees the notes
  starts writing them, and agfront's prompt tests enforce that.

## Dry run on the real records (read-only, fixture mirror, as Front)

`pj-agdev/.local/failsafe-p6/dryrun_receipt.py`, output in
`fixture/receipt-dryrun.txt`. For all seven answers the tool:

- finds the right home (e.g. #8557 → `front/front-protoprey-p3-flux2-capability-20260923`,
  existing mark #8553);
- finds no journal evidence: Front's journal starts on 2026-09-20, and
  serving 109 recorded only its trigger;
- finds the settling decision: Front's `[state] accepted` #15321, #9939,
  #9940, #9944–#9946, #9954; the cancellation #12603 for #9368;
- would write a reconciled receipt.

It also lists **#9039** (m9017 task 1's shown result, 2026-09-23). No mark
at all exists for that task, so #9039 is unreceived too; its settlement is
the same #9954. Step 5 repairs it the same way.

## Tests

- pyagag `tests/test_failsafe_p6.py`, +7:
  - reconciled on a decision, and the repair converges;
  - journal evidence writes the listener's mark (the crash-after-delivery
    case);
  - no mark over an earlier answer nobody was given;
  - no evidence writes nothing, and `--because` must be the caller's own
    later post;
  - an answer arriving after the repaired one stays owed;
  - a resolved home, and an answer not naming the caller;
  - the CLI prints, then repairs.
- `tests/test_serving_lifecycle.py`, +3 on the real listener:
  - a restart after delivery writes the mark in home although the source
    channel was ✔'d and **archived** meanwhile;
  - it writes the mark under a home **resolved** meanwhile, with no twin;
  - a **reconciled** answer buys no run after a restart, while a newer post
    is served and marked.
- pyagag 1073 green; agfront 193.

## Commits

- pyagag `fb76e2b`.
- agfront `3207a39`.
- Not deployed yet (after step 4).
