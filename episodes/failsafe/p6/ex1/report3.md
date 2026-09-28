# failsafe p6 ex1 — step 3: monitoring suppression separated from ending a request

## Result

"Retired" meant three different things in three readers (report1). It is
replaced by explicit **dispositions**. A disposition is a record in the
request's origin conversation, next to its holds. The origin conversation is
never archived.

```
[selfnote][disposition] <kind> a<unit> upto=#<id> by <user id> (<name>) #<evidence> — <why>
[selfnote][disposition-reversed] #<disposition> by <user id> (<name>) #<evidence> — <why>
```

| kind | meaning | the unit | unfinished work below it | card |
|---|---|---|---|---|
| `suppressed` | **monitoring suppressed** | keeps its state | keeps its state | stays in its group with its real state, and the reason says "monitoring suppressed by … : why" |
| `completed` | **request ended**: requested outcome reached | `done` | `cancelled`: "ended with …; its own record: …", listed as `remaining` | `completed`, "ended: completed by …", "ended with it: …" |
| `cancelled` | **request ended** on a decision, without the outcome | `cancelled` | as above | `cancelled` |
| `withdrawn` | **request ended**: taken back by whoever asked (a stray post, an abandoned trial) | `cancelled` | as above | `cancelled`. Never "completed" |

Every record states:

- the affected work: `a<unit>`, the unit and everything below it;
- the decision maker and the evidence: `by`, and their post `#evidence`
  (0 when the decision is recorded in person from the operator's terminal);
- the reason;
- the scope boundary `upto`.

Ended work leaves the active queue. Unfinished work below an ended unit
never reads as completed. Each such unit is listed with what its owner's
own record still says, so the owner's existing path can reconcile it (for a
routine run, `agrun finish`). A missing receipt under an ended unit becomes
settled bookkeeping.

## The activity boundary

`upto` is the newest **substantive** post in the scope when the decision was
recorded: speech that is not an ack, not a selfnote and not a system notice.
Per conversation (`Node.activity`), a substantive post after `upto`
uncovers that conversation:

- a new request, question or result is visible and monitored again;
- the conversations where nothing new happened stay covered.

These do **not** count as new activity:

- receipts, served marks, holds and these records (all selfnotes);
- ✔ and other system notices;
- restarts;
- the recorder's own reply that closes the serving the decision was
  recorded in: `end=` of an ack older than the record. When Front records a
  decision and then says "recorded", nothing new has happened.

Anything else the recorder says afterwards is new activity.

p3's retirement lapsed on **any** message, a bookkeeping note included,
and covered the whole request or nothing.

## Code

- **pyagag `4727dd4`**:
  - `agag.dispositions` (new) holds the notes, the parsers, `read`,
    `apply`, `suppressed_anchors`, and the tools `record`, `reverse`,
    `lines` and `command`.
  - `agag.trace` reads the two tags as records and gives each node
    `last_substantive` and `activity`. It applies the dispositions once per
    trace (`Trace.dispositions`, `Node.disposition`) and then decides the
    holders again for what is still open. `trace_lines` says "monitoring
    suppressed …" and "new activity since #… : not covered".
  - `agag.progress`:
    - a unit ended by a disposition shows the decision, not its own
      completion;
    - a suppressed unit's reason says so;
    - a request ended on its root, with nothing new in it, takes the
      decision as its state and reason. Its stages are history, and its
      remaining units are listed;
    - cards carry `dispositions` and units carry `disposition`.
  - `agentchat disposition [<message id>] [suppressed|completed|cancelled|withdrawn|reversed]
    --evidence <post> [--unit <anchor>] [why]` lists, records and reverses.
    Recording prints what ended with the decision.
- **agobserver (pj-agdev `9ad3e64`)**:
  - `retired.json`, `load_retired` and `retired_now` are gone.
  - Candidates on anchors a suppression still covers are dropped, and a
    request whose root is covered is skipped like a request held as a whole
    (no probes, no incidents).
  - `retain` no longer counts covered anchors as outstanding.
  - Ended units are `done`/`cancelled` in the trace, so they are finished
    for Observer as for everyone.
  - Requests with a disposition record are discovered like held ones, so
    new activity anywhere under a decision is seen.
  - `python -m agobserver.disposition` (`--list`, record, `--reverse`, `--by`
    in person) replaces `agobserver.hold --retire/--unretire`. Those two
    options now exit 2 and name the replacement.
- **Relay (agdevworld `0720880`)**:
  - `retired.json` is not read, and `recovery_for` no longer adds
    `retired`/`retired_why`.
  - Origins with a disposition record are discovered.
  - Group and scope rules are unchanged. A suppressed request's card is
    unfinished, so it stays **active** and explains itself. An ended one is
    `completed`/`cancelled`, so it moves to **recent** and ages out.
- **agfront `93a133c`**: Front's `front` and `desk` guides get "When a
  request is ended, or should not be chased". The section covers:
  - which kind to use;
  - never calling a cancelled or withdrawn request successful;
  - `agrun finish` for a routine run that ended with a request;
  - what counts as new activity;
  - reversal.

## Convergence

The records live in the conversation, so every reader, and every restart,
reads the same state.

- **Repeat.** `record` finds the same kind, unit and decision maker still in
  force and still covering the unit, and writes nothing. That holds after
  the recorder's own reply too.
- **Reversal.** A reversal of a reversed disposition writes nothing.
- **Interruption.** An interrupted recording is either the one post or
  nothing, and running it again gives the same final state.

## Tests

- **pyagag**: 1106 passed, 10 more than step 2 in `test_failsafe_p6ex1.py`:
  - suppression keeps every state and the card state, covers the scope for
    Observer, and is said in the trace and on the card;
  - a suppression on one unit leaves the rest monitored;
  - completed, cancelled and withdrawn: the root, the unit below ("ended
    with …; its own record: awaiting delivery"), `remaining`, the card state
    and reason, the missing receipt settled, and no stall candidate;
  - a receipt note, a served mark and a ✔ after the decision change
    nothing, while a new post in the request uncovers the root and leaves
    the untouched task ended;
  - a new result under a suppression is monitored again;
  - a repeat writes nothing, a fresh reader agrees, and a reversal applies
    once;
  - the recorder's reply reporting its own decision is not activity, and
    its later unrelated post is;
  - `agentchat disposition` refuses without evidence, records and prints
    what ended with it, and lists.
- **agobserver**: 177 passed.
  - p3's retirement test became "suppressed monitoring leaves tracking until
    the request moves again": a bookkeeping note keeps it untracked, and a
    new post tracks it again.
  - One new test: an ended request leaves tracking and opens no incident
    long past every grace.
  - Fixtures state `rel=work`.
- **Relay**: 363 passed. Fixtures state `rel=work`.

## Still to do (step 4)

- `retired.json` still holds o11450, o11522 and o14251. No reader uses it
  any more.
- Their dispositions, the legacy relation records, and the deployment come
  next.
