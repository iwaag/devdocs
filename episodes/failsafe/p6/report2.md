# failsafe p6 — step 2: one reading of completion and outstanding obligations

## Adopted semantics

A unit of work carries separate facts: its **execution record**, the
**requester's decision**, the **receipt** of each answer, and **open
questions**. The actionable state comes from those facts.

| fact | record | who writes it |
|---|---|---|
| execution completion | `[state] completed` / `delivered` / `finished` | the producer (owner) |
| requester acceptance | `[state] accepted`/`done` in the unit, `[change] accepted … +shown=<id>`, `[acceptance] #<evidence> … after=#<shown>` | the requester's side (the holder, or its recorder) |
| cancellation | `[state] cancelled`/`replaced`/`retired` | the owner, on a holder's request |
| receipt | `[served] <remote> <id>` (up to the id); `[selfnote][receipt] #<answer> by #<evidence> (<why>) in <remote>` (exactly that answer) | the agent the answer is owed to |
| open question | a response request (`agag.outstanding`) | anybody |

### The rule (`agag.trace`)

- **An answer without a receipt is owed** (`awaiting_delivery`), whatever
  its producer's record says. A producer's `completed`/`delivered` is still
  checked for a receipt, so the p1 capability is kept.
- **A decision recorded after it settles it.** `trace.decisions` reads, per
  conversation, the records that cover an answer id:

| decision | covers |
|---|---|
| the requester's (non-owner's) `[state] accepted`/`done` in the unit | the unit's answers before the note |
| `[change] accepted … +shown=S` | answers up to S, the result shown for review. The close-out report after the agreement is **not** covered, so it still needs its receipt |
| `[acceptance] #E … after=#S` in a mission | the mission's and its tasks' answers up to S (up to E for notes written before `after=` existed) |
| `[state] cancelled` above the unit | answers before the cancellation |

- The decision is bound by id and by position:
  - it belongs to the unit, or to a unit above it in **the same
    request's** tree;
  - it was written after the answer existed;
  - it names a bound the answer is within.
- A later answer, another request's acceptance and a ✔ settle nothing.
- A settled answer leaves the unit reading as its record does (`done`).
  `Node.receipt` keeps the fact: `{answer, to, home, state: "settled",
  settled_by: {kind, id, covers, what}}`, and the detail says
  `#8557 to Front has no receipt in … — settled by …`. An owed one has
  `state: "missing"`.
- **Speech at home is no receipt for any reader.** The `receipts_from`
  leniency (p1-era answers counted as taken up once the requester spoke at
  home) is removed from `agag.trace` and from Observer. Only Observer
  applied it, which is why it and the panel disagreed. The decision rule
  above replaces it, with evidence.
- **A reconciled receipt** (`[selfnote][receipt]`, new in `agag.selfnote`)
  counts only when:
  - the agent the answer is owed to wrote it;
  - it names exactly that answer id;
  - that answer is in the conversation.

  It claims no serving. Step 3 writes it.

### Requesters wherever the trace starts

- A unit's requesters are every root-note author in it, whether or not the
  home the note names is inside the traced tree.
- `agentchat trace 8519` (from the mission) and `trace 8512` (from the
  request) now give task 1 the same state and the same receipt fact.

### References adopt nothing

- A root note written into a conversation that **began as somebody else's
  request** is a citation. That means the conversation's first post is
  speech by another sender, and it carries no identity note. Such a note
  does not hang the conversation, or its work, under the note's home
  (`trace._reference`).
- Opened-for-work conversations still adopt:
  - a first post that is a note (`[task]`, `[mission]`, a root note);
  - a conversation the note's author began;
  - a deliberate move (`[rootchat-moved]`, `agrun adopt`).
- A history that does not reach the beginning decides nothing.
- Only conversations the tree reaches are read for this (one breadth-first
  pass; the realm has about 1000 root notes).

### Cards, and Observer (`agag.progress`, agobserver)

- An owed answer names whom it is owed to, from the trace's `owed_to`:
  "answer #9368 is not yet taken up by Front" instead of "by autolab". Each
  unit carries `receipt`.
- A card lists `settled_receipts`. A finished card's reason says
  "N answer(s) settled without a receipt (#…: bookkeeping,
  `agentchat receipt`)": settled history, not active work.
- Observer traces with the same call. `undelivered` is raised only for an
  owed answer. A settled task is no longer an obligation in
  `tracked.json`, so a request whose only remainder is bookkeeping leaves
  tracking.
- The panel, Observer and `agentchat trace` now call the same function with
  the same arguments.

## Replay (the step-1 fixture, new code)

| case | before (panel / Observer) | now, every reader |
|---|---|---|
| o11711 | waiting / completed | completed; m8519's request no longer inside it (the #15357 citation is dropped) |
| o8512 | waiting / completed | completed; #8557 settled by Front's `[state] accepted` #15321 |
| m8519 from the mission | completed / completed | completed; the same settled receipt |
| o8816 | waiting / completed | completed; #8866, #8891, #8914 settled (#9944–#9946) |
| o9010 | waiting / completed | completed; #9060 settled (#9954) |
| o8721 | waiting / completed | completed; #8778, #8811 settled (#9939, #9940) |
| o9227 | waiting ("by autolab") / waiting | completed. m9349 stays **cancelled**; task 1 keeps its corrected `completed`; #9368 settled by the cancellation #12603 (asked by the holder at #12599) |

Whole realm: 133 `#front` requests, traced under both old rules and the new
one (`replay_all.py`). The differences:

- The 10 above are false waits settled, or dropped for a citation.
- **Three old requests keep the waits the panel already showed**, and
  Observer's rule had hidden them:
  - 6566, `anchorcheck-c`, 2026-09-12;
  - 3161, the `mediagen` routine's standing conversation (9 plans);
  - 7596, `protoprey-handover`: m7601/m7732. Their missions are unaccepted,
    and failsafe p3 recorded them as the Developer's pending decisions.

  None has had activity in days, so none is in Observer's window or the
  panel's active scope. If one moves again, its answers read as owed, which
  is the truth.
- **Two citations dropped** in the Developer's standing-routine
  conversations. In 2458 (`routine-ghtrends`) and 1550 (`routine-rtnotes`,
  plus autolab's `how-to-request`), Front had posted status questions into
  conversations the Developer began. Their cards move from `queued` to
  `answered`.

## Tests

- pyagag, `tests/test_failsafe_p6.py` (13, synthetic realms shaped like
  m8519):
  - completion alone stays owed, and the card names Front;
  - acceptance settles, and the receipt remains bookkeeping;
  - the mission's acceptance settles its tasks up to the shown result;
  - the same state whether traced from the mission or the request;
  - a bound acceptance does not cover the later close-out;
  - a later result stays owed;
  - another request's acceptance settles nothing;
  - cancellation settles only answers before it;
  - a reconciled receipt covers exactly its answer, and only when written
    by the requester;
  - speech at home is no receipt;
  - a citation adopts nothing;
  - opened-for-work conversations and deliberate moves still adopt.
- `test_identity`'s p1-boundary test is replaced by the decision rule.
- pyagag 1050 + 13 green.
- agobserver 176, with its `receipts_from` test replaced: speech is not a
  receipt, and a decision removes the obligation.
- agentroom 363, against the new source.

## Commits

- pyagag `dad4ced`.
- pj-agdev `0dc798d` (agobserver).
- Not deployed yet. Pins and restarts follow step 4, before step 5 uses the
  tools live.
