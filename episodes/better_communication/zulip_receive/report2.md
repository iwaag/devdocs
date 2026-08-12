# zulip_receive — Step 2 report: the listener loop

Date: 2026-08-12. Status: **complete**. agforge now has a long-poll listener
that sees DMs sent to the forge bot within about a second, ignores its own
messages, and re-registers by itself when the event queue dies.

## What was built

Two new modules in the agforge package, stdlib only, matching the house
style (no new dependency, no inbound port):

- `src/agforge/zulip.py` — the client. Reads the ignored `.local/zulip.env`
  without sourcing shell code (same trick `agent_run` already uses for the ACE
  Studio path), HTTP Basic on every call, and one `call()` seam that turns
  Zulip's error bodies into typed exceptions: `QueueExpired` for
  `BAD_EVENT_QUEUE_ID`, `ZulipTimeout` for a poll that simply saw nothing,
  `ZulipError` for the rest. The four mechanics from Step 1 are one method
  each: `register`, `poll`, `dm_history`, `send_dm`.
- `src/agforge/zulip_listener.py` — the loop: register → poll → filter →
  dispatch, with a pluggable handler. Launchers `service/zulip_listener.py`
  and `service/listen.sh` follow the existing `serve.sh` pattern.

Per Step 1's finding, everything keys on numeric user IDs: the self-loop
guard compares `sender_id` to the bot's own `user_id` from `/users/me`, and
DM history and replies address participants by ID rather than by an address
this realm hides.

## Design notes

- **Timeouts are normal, not errors.** Zulip holds the poll open; the client
  sets its own 90 s ceiling and the loop simply polls again on
  `ZulipTimeout`. Only a real failure sleeps and re-registers.
- **A queue death is not a restart.** `QueueExpired` drops the queue id and
  re-registers on the next iteration, resuming from `last_event_id = -1`.
  Messages sent while no queue existed are lost by design — the plan asked
  for a dumb listener, and the sender can resend.
- **One bad message must not end the loop.** The handler call is wrapped;
  a failure is logged against the message id and polling continues.

Eight deterministic tests live in `tests/test_zulip.py`, following the
house rule that the agent's behaviour is never unit-tested — they pin only
self-echo filtering, partner extraction, and the shell around the run.

## Evidence (from the listener's own log)

```
listening as user_id=13 (forge-bot@agstudio.local)
registered event queue 9ed3… (last_event_id=-1)
DM #23 from 'Developer' (id=8, partners=[8]): 'step2 test: does the listener see this?'
event queue expired (Bad event queue ID: 9ed3…); re-registering
registered event queue 12a1… (last_event_id=-1)
DM #24 from 'Developer' (id=8, partners=[8]): 'step2 test: after the queue was killed'
```

- A DM appeared in the log **about one second** after it was sent.
- The queue was then deleted from outside (`DELETE /api/v1/events`). The
  blocked poll returned the error immediately, the listener re-registered by
  itself, and the next DM arrived normally — **no restart**.
- A DM sent *by the bot itself* produced no handler line at all: the
  self-loop guard works.

The DMs above were injected with the developer account's own API credentials
rather than typed in the web UI. The browser path exercises the same
`POST /api/v1/messages` endpoint and Step 3's live exchange was driven the
same way; a UI-typed DM is a nice-to-have confirmation, not a different code
path.

## Agent run record

- Request/job id: `zulip_receive/step2`
- Agent: Omni Agent (Claude Code harness, `claude-opus-5[1m]`)
- Outcome: done. No in-system agent run was involved in this step — the
  listener only logged at this point.
- Deus Ex Machina note: the Omni Agent wrote agforge's listener — did X for
  agent forge; handoff candidate.
