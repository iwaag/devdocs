# zulip_receive — episode report

Date: 2026-08-12. Status: **reconciled**. Sending a private message to the
Forge bot in Zulip makes agforge react, and the run is given the chat log the
sender can see on screen. The braindump's open question — "first I want to
know whether this is technically possible; if not, think of alternatives" —
is answered: it is possible, with the stdlib and no inbound port, and no
alternative was needed.

## What the desire asked, and what it got

| Braindump | Outcome |
|---|---|
| Implement the receive side of chat | `agforge/service/listen.sh`, supervised by launchd |
| A DM to agforge makes it react | DM → immediate ack → answer, 9–35 s per exchange |
| The visible chat log is part of the context | Up to 50 messages of the DM narrow, speaker-labelled, fed as the run's desire |
| Is it technically possible? | Yes — Zulip's events API long-poll, four calls, stdlib only |

## How it works

The bot registers a Zulip event queue and long-polls it. A `message` event of
type `private` whose `sender_id` is not the bot's own becomes one run:
the DM narrow's last 50 messages are fetched as raw text, formatted one line
per message with the speaker's display name, and handed to
`agent_run.run_request()` — **the same pipeline the `:8092` request service
uses**, called in-process. An ack DM goes out first, because generation takes
tens of seconds and a silent chat reads as broken. The run writes its usual
`result.json`; the text of its `reply` field is posted back as a DM.

Nothing is validated on the agent's behalf: `reply` is a documented request,
not a schema, and a run that answers in some other shape still gets posted.

## What made it work, in the order it mattered

1. **Step 1's manual spike paid for itself.** Two findings from curl alone
   shaped every later line: this realm hides real email addresses from events
   (so the self-loop guard and every narrow key on numeric user ids), and a
   dead queue arrives as HTTP 400 with a body that must be read to be
   recognised.
2. **The self-loop guard is not optional.** The bot's own DMs come back as
   events; without the guard it answers itself forever.
3. **The restart check found the real bug.** Everything looked finished after
   Step 3. Restarting Zulip killed the listener three times: the identity
   lookup sat outside the retry loop, and `urlopen` does not wrap a
   connection dropped mid-response (`http.client.RemoteDisconnected` escapes
   as itself). Both are fixed, and the listener now rides a full stack restart
   in place. See `report4.md`.

The fallback the plan allowed — periodic `GET /messages` polling if the
events API misbehaved through self-signed TLS — was not needed. Events
arrived in under 100 ms. No debt taken there.

## Two entrances, on purpose and temporarily

agforge now has two conversational entrances: the `:8092` `desire` endpoint
(string telephone, used by agdevworld) and Zulip DMs. This is the migration
direction the plan named — chat becomes THE entrance and the string telephone
retires in a later episode — not a design that wants two.

The cost of the temporary state is deliberately kept at zero duplication:
both entrances run the same charter, the same agent, the same `result.json`,
and the same run record. Only the shape of the desire differs. The moment
chat becomes the only entrance, nothing has to be merged back together.

The desire field being handed a conversation is the one honest wart. It is
absorbed in the desire text rather than by renaming a contract that
agdevworld still depends on.

## Seeds for the next episode

- **Other agents' receive sides.** cagent, the devworld assistant, and the
  autolab agents all have Zulip credentials deployed but no listener.
  `agforge/src/agforge/zulip.py` is deliberately agforge-free except for the
  path to its env file — it is the obvious thing to lift into `pyagag` when
  the second agent needs it, rather than to copy.
- **Streams and mentions.** Only DMs are handled. A stream message with an
  `@Forge` mention arrives on the same queue; the filter is what is missing,
  plus a decision about whether a stream reply should quote or thread.
- **Group DMs are already handled** structurally (all non-bot participants
  become the narrow and the reply recipients) but have never been exercised.
- **Cost is now attached to a chat window.** Every DM, including "thanks",
  starts a `sonnet` run at roughly 0.13–0.33 USD. Nobody has decided whether
  the agent should be able to answer trivially without a full run, and the
  answer should come from watching what people actually send.
- **Missed messages.** The listener is dumb by design: a DM sent while it is
  down is lost, and `last_event_id` resets on every re-registration. If
  losing one turns out to matter in practice, that is the evidence that
  justifies persistence — not before.

## Records

- Steps: `report1.md` (API spike), `report2.md` (listener), `report3.md`
  (context and reaction), `report4.md` (history and supervision).
- In-system agent runs: forge agent, `sonnet` profile (Claude Code +
  `anthropic/claude-sonnet-5`), three live exchanges, 8.9–31.3 s,
  0.13–0.33 USD each; every run also left its own `.agent-run.json`.
- Omni Agent: Claude Code harness, `claude-opus-5[1m]`.
- Deus Ex Machina note: the Omni Agent wrote agforge's chat entrance and drove
  its verification — did X for agent forge; handoff candidate.
