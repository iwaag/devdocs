# p8 step 1 — pyagag: selfnotes and the root

AI-generated (Omni Agent, 2026-08-21). Commit `6d5853d` in `pyagag`, pushed
to GitHub.

## What was built

`agag/selfnote.py` is the whole convention, in one place:

- `is_selfnote(content)` — a message whose content starts with `[selfnote]`.
- `note(tag, value)` / `parse_note(content, tag)` — the general
  `[selfnote][<tag>] <value>` shape. `rootchat` is the tag this module names;
  a consumer adds its own (step 2 gives agforge a `work` note).
- `rootchat_note(home)` / `parse_rootchat(content)` / `own_rootchat(history,
  self_id)` — the root note, and reading a topic's anchor off its history.
- `Conversation`, `parse_conversation`, `home_from_environment`,
  `HOME_VARIABLE` — moved here from the deleted `participation.py`, because
  they are what the note is made of.
- `last_real_message` / `last_real_sender` / `without_selfnotes` — the crux,
  below.

### Hiding it

`topics.format_chatlog` drops selfnotes **for every sender**, not just other
people's. That is a deliberate reading of the braindump, which asked only
that they be hidden from everyone but the author: an agent that sees its own
notes starts writing them by hand, and a note written by a model is prose,
not a record. `write_threads` renders through the same function, so
`threads/` is covered by the same line. `agentchat read` hides them unless
`--all` is given.

### Not serving on it — the crux

Every place a listener asks "who posted last" now asks for the last
**non-selfnote** message:

| place | what it decides |
|---|---|
| `zulip.sweep_topics` | whether an owned topic awaits a reply |
| `zulip.sweep_serve`'s pre-serving re-check | whether a pending entry is stale |
| `zulip.topic_from_event` | whether an arriving message schedules a serving |
| `zulip.is_mention_for_us` / `mention_from_event` | whether it is a mention |
| `zulip.sweep_mentions` | startup mention recovery |
| `topics.serve_topic`'s post-run re-check | whether to serve the topic again |
| `topics.handoff_mention` | who the reply hands the turn to |
| `topics.TopicContext.humans_spoke` | whether there is anything to answer |

Miss one of these and the root note an agent writes into another agent's
topic *is* a post by somebody else, which serves that topic's owner — p7's
ack loop with no ack in it. Eight regression tests are written from that
trail (`tests/test_zulip.py`, "selfnotes must not buy anybody a run"), one
per row above plus the two recovery paths.

One behavior was deliberately kept: a topic with **no messages at all** still
matches the sweep, because `serve_topic`'s `empty_reply` is what silences it.
A topic holding only selfnotes does not match — nobody has spoken in it.
`LAST_SPEAKER_LOOKBACK = 10` is the cost of the distinction: the last-poster
check reads ten messages instead of one, the same single API call.

### `agentchat send`

`ensure_rootchat` runs before the real post: one `whoami` and one history
read, and if this bot has no `[rootchat]` note in the topic, it posts one
naming `AGENTCHAT_HOME`. The result is cached per process — a run that posts
twice pays once, and one `agentchat` invocation is one process, so the cache
lives exactly as long as the fact it caches. Without `AGENTCHAT_HOME` no note
is written (a run nobody will call back), and posting into the home topic
itself writes none either (it is not somewhere else). A note that cannot be
written prints and is not fatal, the same rule `ensure_subscribed` already
had: the message matters more than the callback.

### Callback and recovery

- `zulip.rootchat_home(client, channel, topic, self_id)` — the callback's
  lookup. A run named in somebody else's topic reads that topic, finds its
  own root note, and that note is the conversation to serve. `None` for a
  topic this bot never anchored: somebody else's business, as in p7.
- `zulip.remotes_for_home(client, channel, topic)` — the `threads/` list,
  replacing `participation.remotes_for_home`.
- `zulip.rootchat_notes` / `sweep_rootchats` — the `sender:<me>` +
  `search:rootchat` narrow. One call lists every topic this agent anchored;
  each is then checked for "last real speaker is somebody else, and names
  me". `sweep_serve` runs it beside `sweep_mentions` on every queue
  registration, so startup recovery is the union of the two.

**The narrow was verified against the live realm** before being relied on:
posting `[selfnote][rootchat] front/front-probe` as the Omni Agent bot and
querying `sender:<me> search:rootchat` returned exactly that message
(`#general` > `p8-selfnote-probe`, message 866, since deleted). Postgres
tokenizes `[selfnote][rootchat]` into the two bare words, so the tag is
searchable despite the brackets.

### Deleted

`src/agag/participation.py` and `tests/test_participation.py`.
`AGENTCHAT_LEDGER` no longer exists in pyagag. `HOME_VARIABLE` survives, in
`agag.selfnote`.

## Tests

349 pass (`uv run pytest`), up from 317. New: `tests/test_selfnote.py` (27),
the selfnote block in `tests/test_zulip.py` (8), the selfnote block in
`tests/test_topics.py` (4), and `tests/test_chat.py` rewritten around the
root note instead of the ledger.

## Deviations from the plan, and one consequence

- The plan put the callback under step 1 as pyagag work and step 3 said
  agfront needed "nothing else in code". Resolving a home from the chat
  instead of from a file *is* a code change in agfront, so the lookup was
  written as `zulip.rootchat_home` here to keep agfront's own change to three
  lines. Step 3 still touches `agfront/src`.
- **agautolab imports `agag.participation`.** Its behavior is not touched
  this phase and it is not being re-locked, so it stays on the pinned older
  pyagag and keeps working; its listener is not running. Adopting selfnotes
  there is the later phase the plan already names — but until then, agautolab
  cannot take a pyagag bump without that work.
