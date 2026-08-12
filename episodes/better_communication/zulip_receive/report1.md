# zulip_receive — Step 1 report: manual API spike

Date: 2026-08-12. Status: **complete**. All four receive-side API mechanics
were exercised by hand with `curl` before any code was written, plus the
queue-expiry recovery path. Two findings change the design slightly.

## What was exercised

Credentials came from the ignored `agforge/.local/zulip.env` (forge bot) and
`pj-agdev/.local/zulip/developer.env` (the human account, used to play the
sender). HTTP Basic on every call; the self-signed TLS needs `curl -k`.

1. **Identity** — `GET /api/v1/users/me` confirms the bot: full name `Forge`,
   `is_bot: true`, `user_id` 13, owned by the developer account (id 8).
2. **Register** — `POST /api/v1/register` with `event_types=["message"]`
   returned `result: success`, a `queue_id`, and `last_event_id: -1`.
3. **Send + receive** — a DM sent from the developer account to the bot
   (`POST /api/v1/messages`, `type=direct`, `to=[<bot email>]`) returned
   message id 21, and the very next `GET /api/v1/events?queue_id=…&
   last_event_id=-1` returned it as event id 0 in **0.06 s**, with
   `message.type == "private"` and both participants in `display_recipient`.
4. **History** — `GET /api/v1/messages` with
   `narrow=[{"operator":"dm","operand":[8]}]`, `anchor=newest`,
   `num_before=50`, `apply_markdown=false` returned the conversation as plain
   text (`apply_markdown=true`, the default, wraps content in `<p>` HTML).
5. **Reply** — `POST /api/v1/messages` with `type=direct&to=[8]` posted as the
   bot (message id 22).
6. **Self-echo** — the next poll returned the bot's own reply as an event with
   `sender_id == 13`. The self-loop guard is mandatory, exactly as the plan
   warned.
7. **Blocking behaviour** — a poll with nothing pending held the connection
   open for the full 12 s test cap without returning (no heartbeat inside that
   window). The listener therefore needs its own socket timeout, not a
   dependency on the server returning promptly.
8. **Queue death** — `DELETE /api/v1/events` then polling that queue returned
   HTTP **400** with `code: BAD_EVENT_QUEUE_ID`. This is the re-register
   trigger, and it arrives as an HTTP error status, so `urllib` will raise
   `HTTPError` rather than return a body — the listener must read the error
   body to distinguish it from a real failure.

## Findings that change the design

- **Email addresses are hidden in this realm.** The event carried
  `sender_email: user8@agstudio.local`, not the account's real address. So:
  - the self-loop guard must compare **`sender_id`** against the bot's own
    `user_id` (from `/users/me`), not `sender_email`;
  - the DM narrow and the reply recipient should both use **numeric user
    IDs**. `operand` accepts either the ID list or the pseudo-address and both
    were verified working, but the ID list does not depend on a realm privacy
    setting.
- **Events arrive essentially instantly** (sub-100 ms here), so the
  `GET /api/v1/messages` polling fallback mentioned in the plan is not needed.

## Notes

- No secrets, tokens, or host/IP values are recorded in this file; the base
  URL and credentials stay in the ignored env files.
- Numeric user IDs (8 = developer, 13 = forge bot) are realm-local
  identifiers, not host or network information.

## Agent run record

- Request/job id: `zulip_receive/step1`
- Agent: Omni Agent (Claude Code harness, `claude-opus-5[1m]`)
- Outcome: done
- Cost/time: not separately reported by the harness
