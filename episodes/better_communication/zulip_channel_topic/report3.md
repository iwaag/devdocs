# Step 3 report — agforge reacts in `create-*` channels

Date: 2026-08-12. Result: **a channel message produces the same behavior as a
DM** — ack first, then the generated asset's URL, all inside the topic. Tests
71/71.

## Code

- `agforge/src/agforge/zulip_chat.py`: the DM-only pipeline was generalized.
  A new `conversation(client, message, self_id)` returns where the message
  lives — `(history_fn, send_fn, label)` — a topic narrow +
  `send_to_channel` for stream messages, the partner narrow + `send_dm` for
  DMs. `run_and_reply` now takes those callables; everything downstream
  (transcript formatting, ack, `agent_run.run_request`, reply extraction) is
  shared verbatim between both entrances. The desire preamble now says "a
  Zulip chat" instead of "a direct message".
- `agforge/src/agforge/zulip_listener.py`: an `accept` rule replaces the
  implicit DM filter:
  - DMs, as before;
  - stream messages in channels starting with `create-`
    (`REQUEST_CHANNEL_PREFIX`) **whose topic is not resolved** (`✔ ` prefix)
    — a resolved topic is a finished conversation, and late chatter there
    should not cost a run;
  - not `#FreeForge`: that channel is for the assistant's announcements, and
    agforge answering every announcement would be noise at ~$0.13 a message.
  The passive `AGFORGE_ZULIP_LOG_ONLY=1` handler logs both shapes.

New tests: the accept rule's five cases, `conversation` answering a channel
message in its own topic, and the DM fallback (including the no-partners
degenerate case).

## Live verification (real run, cost accepted)

Posted as Devworld Assistant into `#create-20260812-spike-0001`, topic
`live-test`: "small image of a green frog sitting on a lily pad". After
`launchctl kickstart -k gui/$(id -u)/com.agdev.agforge-zulip`:

- listener log: `1 messages of context, channel='create-20260812-spike-0001'
  topic='live-test'`, then `profile=sonnet … cost_usd=0.1244673
  duration_ms=28929 status=ended`;
- the topic, read as Developer: request → Forge's ack → Forge's reply with
  the presigned URL;
- the URL served `200`, 166 KB, `image/jpeg`.

## Notes

- The in-topic context works exactly like the DM narrow: a follow-up "same
  frog but blue" in the same topic would carry; a new topic starts clean.
- One deliberate cost quirk: every non-resolved message in a `create-*`
  channel starts a run, including the Developer saying "thanks". Same open
  question as the DM entrance (zulip_receive report); same answer — watch
  real usage before adding rules.
