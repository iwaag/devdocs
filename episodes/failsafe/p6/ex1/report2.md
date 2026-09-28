# failsafe p6 ex1 — step 2: what a conversation relation means

## Result

A root note still says **where answers go**. It now also says, on the
record, **whose work the conversation is**. Only a work relation adopts.
Readers no longer decide ownership from how much history they happened to
read.

| relation | recorded by | return address | adopts (trace, waits, acceptance, receipts) |
|---|---|---|---|
| **work**: delegation | `[selfnote][rootchat] <home> #<anchor> rel=work` | home | yes |
| **reference**: citation or comment | `… rel=reference` | home | no; listed on the home as "cites …" |
| **adoption**: deliberate move | `[selfnote][rootchat-moved] <home> #<anchor>` (`agentchat anchor`, `agrun adopt`) | the new home | yes |
| correction | `[selfnote][relation] #<note> <work\|reference> — <why>`, by the note's author (newest wins) | unchanged | as corrected |
| legacy | `[selfnote][relation] legacy upto=#<id> reference=#… unknown=#… — <why>`, one per author | unchanged | each note of that author up to `upto` without a word is work unless listed |
| **unknown**: nothing records it | — | home | no; listed on the trace and the card with the command that resolves it |

A missing relation never makes an ownership edge, and it never hides work.
The unowned conversation stays traced from its own request.

## Code (pyagag `e72b6a2`, agfront `75d739f`)

- **`agag.selfnote`**:
  - `rootchat_note(home, relation="work")` writes the word;
  - `rootchat_relation` reads it;
  - `effective_rootchat_note` returns the note message that
    `effective_rootchat` reads;
  - `parse_conversation` ignores a trailing `rel=` word, so every existing
    routing reader keeps working unchanged (the mirror index, Front's
    routine reader, the relay's closing and ops views).
- **`agag.relations`** (new) is the shared writer and reader:
  - `Relations.of(note)` gives the relation and what decided it (`note`,
    `move`, `record`, `legacy`, or nothing);
  - `load(client)` makes one realm-wide note search, which a mirror answers
    for free;
  - `beginning_relation` is the default a writer applies;
  - `classify_legacy` and `python -m agag.relations legacy --mirror …
    --channel … [--apply]` write the migration record.
- **Writers.** Every writer that opens a conversation for work states
  `rel=work` through the default argument: autolab's tasks, forge's asset
  runs, `agproject` setup topics. They need a re-pin, not a code change.

  `agentchat send` (`ensure_rootchat`) reads the conversation's **beginning
  oldest-first** (`ZulipClient.topic_beginning`) and decides:
  - work for a new conversation, one the sender began, or one opened for
    work (a note first, or an identity note);
  - reference for one that began as somebody else's request, printing what
    that means and how to say otherwise;
  - an unreadable beginning writes no word (unknown) and says so.

  `--relation work|reference` overrides the default. On a conversation that
  already has the sender's note, it records a correction instead, and a
  repeat writes nothing.
- **`agentchat relation <channel> <topic> [work|reference] [--because
  <post>] [why]`** lists every root note there: whose it is, where it
  returns answers, its relation and what decided it. With a kind, it
  corrects the caller's own note. A move is refused, because a move is
  already work.
- **Readers.** Every reader now takes ownership from the same relation:
  - **`agag.trace`**:
    - only links whose relation adopts are edges;
    - citations and unknowns are carried on their home
      (`Node.relations`), and unknowns also on `Trace.unknown_relations`;
    - `trace_lines` prints "cites …" and "? relation unknown … — <command>";
    - `_reference` and its `len(messages) >= HISTORY` rule are deleted;
    - a failed relation-record search reads notes without a word as
      unknown, and says so in `problem`.
  - **`agag.acceptance.decision`**: the requester comes only from a root
    note that adopts, and the holders chain stops at a citation. #15357
    had made o11711's requester the holder of m8519's acceptance in place
    of the Omni Agent.
  - **`agag.receipt._decision_for`** climbs only work relations. It now
    uses the effective note, so moves count as well; before, it read plain
    notes only.
  - **`agag.progress`**:
    - each unit carries `relations`;
    - the card carries `unknown_relations`;
    - the reason names them.

    Observer and the relay consume the same trace, so they need no code of
    their own for this.
  - **Callbacks** are unchanged by design. `rootchat_home` follows the home
    whatever the relation, so a reply to a comment reaches the commenter.
- **Guides.** Front's `front` and `desk` guides say what the two relations
  mean, what `send` picks, and how to inspect or correct it (a cleanup
  comment is a reference). `Bash(agentchat:*)` already grants the new
  subcommand.

## Migration evidence

`python -m agag.relations legacy` was run offline (`classify_legacy`) on
the step-1 mirror copy, which covers every conversation completely. It
reproduces step 1 exactly:

| author | legacy notes up to | work | reference | unknown |
|---|---|---|---|---|
| Front | #15425 | 349 | #1574, #1752, #3033, #4825, #15357 | 0 |
| autolab | #15409 | 153 | 0 | 0 |
| forge | #11465 | 27 | 0 | 0 |
| archsage | #11594 | 4 | 0 | 0 |
| Omni Agent | #7194 | 1 | 0 | 0 |

(Front's single deliberate move, #6581, reads as work without a record.)

The records are written right before the deployment in step 4, each with
its author's own credential, so no old-code note falls after `upto`. A
note written later by an unredeployed writer reads unknown and is listed.
Old code ignores `[selfnote][relation]`.

## Tests

pyagag passes 1096 tests: the previous 1079 plus 17 in
`test_failsafe_p6ex1.py`. The new tests cover:

- a citation adopts nothing at 199, 200, 201 and 450 posts;
- a legacy citation nothing classifies is **unknown** at 199–201 posts,
  listed, and prints its resolving command;
- a truncated read (40 posts) neither adopts the citation nor drops a real
  task;
- the legacy record classifies its author's notes only, and a later note
  without a word is unknown;
- only the author's correction counts, and the newest wins;
- `send` records work, reference and work for a new conversation, somebody
  else's request and a task, and prints the reference advice;
- `send --relation` overrides the default, corrects the sender's own note,
  and writes nothing on a repeat;
- an unreadable beginning writes unknown and says so;
- `agentchat relation` lists what decided each note;
- delegation, a citation with a reply, and a deliberate adoption each give
  the right owner, callback destination and acceptance holders;
- a receipt decision does not climb a citation;
- rename, resolve and a reused name keep the relation on its request.

The existing suites needed their fixtures updated to the new note shape:

- the synthetic realms now write `rel=work`, and p6's cleanup citation
  `rel=reference`;
- the three realm-export replays (`trace_p3`, `progress_p1`, and the
  trials) get the legacy records their authors would have written
  (`tests/legacy_relations.py`, built with the same `beginning_relation`);
- three exact-string assertions now expect `rel=work`.

No behaviour assertion changed.
