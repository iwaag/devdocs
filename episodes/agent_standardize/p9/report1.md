# p9 step 1 — pyagag: the served marker

AI-generated (Omni Agent, 2026-08-21). Commit `d14a8e0` in `pyagag`, pushed
to GitHub; `8209af7` in `agfront`, which is the first consumer to write the
mark.

## The bug, live before the fix

p8 left `sweep_rootchats` with no notion of "already served" and predicted
what that costs. It was reproduced against the running realm before a line
was changed — Front's own credentials, p8's finished exchange:

```
anchored ('agforge-agstudio1', 'assetplan-paper-airplane-icon') -> front/front-20260821-p8-icon
anchored ('agforge-agstudio1', 'assetrun-paper-airplane-icon')  -> front/front-20260821-p8-icon
marks:   {}
waiting: [('agforge-agstudio1', 'assetplan-paper-airplane-icon'),
          ('agforge-agstudio1', 'assetrun-paper-airplane-icon')]
```

Both exchanges finished more than an hour earlier — forge delivered, Front
told the developer, nobody was waiting for anything. They read as "waiting"
because since p8 the answer goes **home**: Front never becomes the last
poster in the topic that named it, so "somebody else spoke there and named
me" is true for the rest of that topic's life. Every listener restart would
have re-served both. In p9 that same shape runs a supercoder against a live
repository, so it is a prerequisite and not an open item.

## What was built

### The note

`agag.selfnote` gains a second tag beside `rootchat`:

```
[selfnote][served] <channel>/<topic> <message id>
```

`SERVED_TAG`, `served_note(remote, message_id)`, `parse_served(content)`.
The id is the last word, so a topic name may contain slashes and spaces the
same way a root note's may.

The note is written **into home** — the agent's own topic. That is the same
choice p8 made for the reply, for the same reason: a post in the remote topic
would be a post in somebody else's conversation, and would serve them. In
home it triggers nobody, because a selfnote is never somebody speaking.

The plan allowed keeping the id in the home topic's workspace instead and
asked for a reason if the chat was not used. The chat was used, no deviation
to explain: a workspace file does not survive a node being rebuilt from
GitHub, it is not visible to the developer reading the conversation, and it
would put the *one* memory that guards against duplicate paid work in the one
place p8 deleted memories from.

### Reading it back

- `ZulipClient.own_notes(tag, num_before)` — the `sender:me search:<tag>`
  narrow, generalized out of `own_rootchat_notes`. `own_rootchat_notes` and
  the new `own_served_notes` are one line each on top of it.
- `served_marks(client)` → `{(channel, topic): newest served id}`. The
  highest id per topic wins; a later note supersedes an earlier one.
- `sweep_rootchats` skips a topic whose newest naming post is **at or below**
  its mark. A newer post there is a new question, and is served.

One extra API call per sweep, whatever the number of topics.

### Writing it

- `mark_served(client, home, remote, message_id)` — the post.
- `note_served(client, home, channel, topic, message_id=None)` — what a
  listener calls after serving a callback. With no id it reads the remote's
  newest **real** message (`last_real_message`, so a root note that arrived
  after the question is not what gets marked) and returns the id recorded, or
  `None` when there was nothing real there to mark.

### The lookback bound

`LAST_SPEAKER_LOOKBACK` 10 → 30. p8 wrote this down as "not reachable today —
an agent writes one note per topic". The served note makes it reachable from
the other side: a home now collects one `[served]` note per callback *and*
one `[rootchat]` note per topic it reaches out to, and a supervision of three
tasks is several callbacks deep. A conversation buried under its own
bookkeeping must still resolve its real last speaker, or a topic that awaits
nobody is never served again. The cost is unchanged — it is the `num_before`
of a call that was already being made.

## Tests

361 green in pyagag (was 349), 26 in agfront. The new ones:

| test | what it pins |
|---|---|
| `test_the_served_note_round_trips` | the format |
| `test_a_served_topic_may_hold_a_slash_and_the_id_is_the_last_word` | parsing |
| `test_anything_that_is_not_a_served_note_parses_to_none` | four near-misses |
| `test_a_callback_already_answered_is_not_swept_again` | the p8 bug, twice: the same sweep before and after the mark |
| `test_a_post_newer_than_the_mark_calls_us_back_again` | the mark bounds one exchange, it does not retire the topic |
| `test_the_newest_mark_wins` | and a human quoting the word is not a mark |
| `test_the_mark_is_written_at_home_and_names_the_post_it_answered` | into home, and the newest *real* post |
| `test_a_topic_with_nothing_real_in_it_is_not_marked` | nothing served, nothing claimed |
| `test_a_home_buried_under_its_own_notes_still_knows_who_spoke_last` | the lookback bound, with `LAST_SPEAKER_LOOKBACK - 1` notes on top of the request |
| `test_a_mention_serves_the_front_topic_it_was_sent_on_behalf_of` (agfront) | extended: the mark is posted, and it is posted at home |
| `test_a_mention_front_never_anchored_leaves_no_mark` (agfront) | a dropped mention claims nothing |

## agfront, wired in this step

agfront is not a step of its own in the plan but it is a live participant in
the p9 mission — Front supervises the whole thing — and it is the consumer
that already had the p8 callback path. `handle_mention` now calls
`note_served` after `serve_topic` and logs the id. pyagag re-locked to
`d14a8e0`.

## The realm, before the restart

The two p8 topics above were **resolved** rather than marked by hand: the
exchange they hold is finished, and `rootchat_notes` already drops resolved
topics. `sweep_rootchats` for Front then returned `[]`, and the listener was
restarted:

```
06:43:13Z full sweep: 0 awaiting, 1 mentioning, 29 calls spent, 976 left in the window
06:43:13Z serving mention in 'pj-simpleshooter'/'workplan-shield-pickup-icon'
06:43:13Z mention in ... carries no root note of ours; ignoring
```

Zero runs, which is what a restart after a finished exchange should cost.

*Did the resolve of two forge topics for the agents involved — housekeeping
of a finished p8 exchange, not a handoff candidate.*

## What is not done here

agforge and agautolab do not call `note_served` yet; that is steps 2 and 3.
Their listeners must not be restarted with anchored, finished topics until
they do.
