# failsafe p7 — step 4: implementation

All changes are on `failsafe-p7` branches: pyagag `e39956c`, pj-agdev
`4356863` (agobserver) and pj-clusterintent `6afea1d` (cagent). They are
merged and pinned in step 5's roll-out.

## pyagag

### `agag.claims` (new)

- **The records.** `records_of(message)` turns one post into the acts it
  records (report1's table):
  - a note by tag: `hold`, `hold-release`, `disposition`,
    `disposition-reversed`, `relation`, a `rootchat` that states `rel=`,
    `acceptance`, `[state] accepted|done`, `approval`, `receipt`, `served`;
  - a post of speech outside the reply's own conversation is a `send`;
  - memo channels and acks are nothing.

  Each record keeps every id it names (`a15835`, `#15837`, `m20402`,
  `upto=#…`), so a claim's target can be matched to it.
  `window_records(mirror, self_id, after, upto)` reads the serving's own
  posts off the mirror (`Store.messages_by_sender` gained `since_id` and
  `upto_id`).
- **The reader.** `OllamaReader` makes one `/api/chat` call: JSON format,
  temperature 0, no thinking, and the system prompt measured in step 2
  plus "posting this reply itself is not a send". `parse_reader_output`
  accepts fences and drops unknown acts. An unreadable answer is a
  `ReaderError`, never an empty list. `reply_words` removes the handoff
  mention and the `ag-post` line.
- **The judgment.** `judge(claims, window, …)` returns `(missing, found)`:
  - a record of the claimed kind in the window counts;
  - a send is matched to the named conversation when it can be, and to any
    send of the serving otherwise;
  - failing both, a record already made counts: one naming the target
    (always required for `release` and `receipt`), or for the other acts
    one in the target's conversation, or, with no target, one in the
    reply's own conversation.
- **The notes.** `[selfnote][claim] {json}` holds reply, serving, attempt,
  window, requester, missing, found and `of`. `[selfnote][claim-settled]
  #<claim> recorded|corrected|dismissed [#<reply>] [— why]`. `claims_of`
  gives each claim its state: `repairing`, `escalated`, `superseded`,
  `recorded`, `corrected` or `dismissed`.
- **The notice.** `notice(open claims)` quotes each claim, names the record
  it lacks and the tool that makes it, and states the two ways out: do it
  now, or say plainly that the earlier reply was wrong.
  `notice_for_current()` reads the open claims in the bound serving's home
  off the running listener's mirror.
- **Configuration.** `ClaimCheck.from_host()` reads
  `~/.config/agag/claims.toml` (`AGAG_CLAIMS_CONFIG`). Its `[reader]` holds
  `url`, `model` and `timeout`; the optional `[check]` holds `attempts`,
  `retry_seconds` and `mirror_wait`. No file means no reader, and the
  problem is kept for the outcome.
- **The operator's door.** `python -m agag.claims <reply> --mirror <copy>
  [--after <ack>]` checks one posted reply the way the listener does and
  writes nothing. `--settle <claim> <why>` closes a claim a person decided
  to leave (`dismissed`), under `AGENTCHAT_ZULIP_ENV`.

### The listener (`agag.listen`)

- `Listener(claims=ClaimCheck | None)`. With a check, opening a serving
  records `extra.window_from` (the mirror's newest id); the ack narrows the
  window on the owner route.
- After a delivered serving's receipts, `_check_claims`:
  1. skips a serving with no reply words of the agent's (the failure line,
     nothing delivered);
  2. waits up to `mirror_wait` for the mirror to hold the reply;
  3. reads, judges and settles;
  4. for a new mismatch, writes `[claim]` and, when the conversation is
     one this listener serves (`topic_filter`), `[start] #<claim> for
     <requester>`.

  The outcome is kept in the serving record (`extra.claims`: state, tries,
  read, found, missing, window, note).
- `_settle_claims`. An open claim written before this serving began (so
  its prompt carried the notice) is settled `recorded` when every missing
  act now has a record. Otherwise:
  - an attempt 1 is settled `corrected` when this reply does not claim the
    same act again;
  - when it does, it becomes an attempt-2 `[claim]` with `of` and no start
    note.

  An escalated claim closes only on its record.
- An unchecked serving (no reader, a reader error, the mirror behind) is
  tried again while the executor is idle (`_retry_claims`, from the
  journal, at most `attempts`). A last failure stays `unchecked` with
  `final`, and is logged. It is never `clean`.
- `current_listener()` beside `current_mirror()`.

### Serving and prompts (`agag.topics`)

- `serve_topic` keeps the run's own reply words in the journal
  (`extra.reply_words`) when the reply mark was usable.
- `prompt_with_guide(reply=True)` appends the claims notice after the
  retry notice: every conversational role of every agent, with no guide
  text.

### Readers of the records

- `agag.trace`: `Node.claims` (open claims). `trace_lines` prints `!
  claim #…: reply #… says release #15837 — no record (…)`, and
  `next_actions` lists it. `stall_candidates` adds a `claim` candidate:
  - an escalated claim after 60 s;
  - a `repairing` one after `claims.REPAIR_SECONDS` (600 s).

  Its evidence is `(claim, reply)`.
- `agag.progress`: each unit carries `claims`, and the card carries
  `claims`. A card that would read completed, cancelled or answered reads
  `waiting` with the claim as its reason.
- `agag.agent`: every AgentSpec listener (Front, autolab, forge, archsage,
  Observer) gets `ClaimCheck.from_host()`. In log-only mode it gets none.
  The start-up log says whether the check is on and with which model.

## agobserver

`Monitor.handle` reports a `claim` incident to the owners at once and asks
nobody, like `unanswered`: its own listener already served the agent once
with the mismatch, or could not. `Monitor.recovered` closes it only when
the claim is no longer open in the trace: its settlement is on record.

## cagent

`zulip_window.main` passes `ClaimCheck.from_host()` to its listener, and
none in log-only mode.

## The proxy case

A release on the Omni Agent's words with the Developer's authority is
written by the agent's own bot as `[hold-release] #15837 by 9 (Omni Agent)
for 8 (Developer) #15842`. The check looks at who posted it and at the
kind and target. The `by … for …` is the tools' business (`acts_for`) and
stays that way. Pinned by
`test_a_reply_whose_records_are_in_its_window_is_clean_and_writes_nothing`.

## Tests

pyagag `tests/test_failsafe_p7.py`, 16 tests over the real listener,
`serve_topic` and mirror on the fake realm, with a stub reader:

- **run-0183 reproduced**:
  - the mismatch is recorded, with the Omni Agent as requester;
  - the start's `because` is the claim;
  - exactly one repair serving, whose `trigger_id` is the start note;
  - its prompt carries the quote and `agentchat release`, and the first
    prompt carries nothing;
  - its records settle the claim `recorded`, and its own check is `clean`;
  - the repair is addressed to `@**Omni Agent**`;
  - nothing further is served.
- A repair that says it again: attempt 2 (`of` the first), `superseded`
  then `escalated`, one start note in all, and no third serving.
- A repair that corrects: `corrected`.
- Records in the window, including the proxy's `for`: `clean`, nothing
  written.
- A restatement of a record already made, read by an over-reading reader,
  and a quoted command: `clean`, `on record`.
- A send claim with a post elsewhere in the window is `clean`; without one
  it is `missing: send`.
- A reader that fails once: `unchecked`, then detected on the retry.
- No reader configured: `unchecked`, with the reason.
- **The known limit**: a reader that extracts nothing leaves the false
  reply `clean`.
- A failure line is never read.
- The rules without a listener: reader parsing, words, `records_of` per
  act, `judge`, claim states, the notice, the trace line, the candidates
  and the card.

agobserver `tests/test_p7_claims.py`: an escalated claim is reported once
to the owners by name and Front is not asked, and recovery happens only on
settlement.

Suites on the branch (the consumers with `PYTHONPATH` set to the branch's
`src`, before the pin):

| suite | passed |
|---|---|
| pyagag | 1186 |
| agobserver | 180 |
| agfront | 198 |
| agautolab | 334 |
| agforge | 265 |
| archsage | 41 |
| cagent | 204 |
| agentroom relay | 363 |

## Guide text

None. Option 1 was not chosen, so `REPLY_GUIDE` is unchanged. The only
words a run sees are the notice, and only in the serving that answers a
mismatch.
