# Plan: Zulip receive side — agforge reacts to private chat

## Goal

Minimal receive-side milestone: sending a Zulip private message (DM) to the
`forge` bot makes agforge react. The visible DM history is included in the
agent's context, so the sender can talk with the normal expectation that the
bot "remembers" the on-screen conversation.

## Non-goals

- No stream/mention handling, no other agents — DMs to forge only.
- The existing request service (`:8092`) keeps working unchanged; string-telephone
  routes are untouched.
- No polish on conversation quality; this proves the mechanism.

## Constraints (everything else is implementer's discretion)

1. No secrets or local host/IP details in git-tracked files. Zulip credentials
   stay in `agforge/.local/zulip.env` (already deployed, mode 0600).
2. Record which backend/model served each agentic run (Agent ≠ Model policy).

## Useful context and hints

- **Zulip**: `https://agstudio.local:8543`, self-signed TLS (`curl -k` or pass the
  CA). Forge bot credentials: `agforge/.local/zulip.env`. Auth is HTTP Basic
  `bot_email:api_key` on every call.
- **Receive = events API long-poll** (chosen in discussion; stdlib-only, no
  inbound port):
  1. `POST /api/v1/register` with `event_types=["message"]` → `queue_id`,
     `last_event_id`.
  2. Loop: `GET /api/v1/events?queue_id=...&last_event_id=...` — blocks until
     events arrive or heartbeat. Advance `last_event_id` after processing.
  3. On `BAD_EVENT_QUEUE_ID` (queues expire after ~minutes of no polling, and on
     server restart): re-register and continue. This is normal, not an error.
- **Self-loop guard**: the bot's own outgoing DMs come back as message events.
  Filter `sender_email == bot email` (or `sender_id`) before reacting, or the
  bot answers itself forever.
- **DM detection**: message events with `message.type == "private"`. Recipients
  are in `display_recipient`; 1:1 vs group DM both work the same way.
- **History fetch** (the "visible chat log"):
  `GET /api/v1/messages?anchor=newest&num_before=50&num_after=0&narrow=[{"operator":"dm","operand":"<sender email>"}]`
  Returns the exact conversation the sender sees, bot replies included, edited
  messages in their current form. Add `apply_markdown=false` for raw text.
  For group DMs, `operand` takes the list of participant emails.
- **Reply**: `POST /api/v1/messages` with `type=direct&to=[<sender email>]&content=...`.
- **agforge shape today**: request service is Python stdlib HTTP
  (`src/agforge/request_service.py`, run by `service/serve.sh` on `:8092`,
  launchd template in `devenv/launchd/`). Agent runs are OpenCode subprocesses;
  generation goes through `scripts/generate.sh` → SwarmUI, results uploaded to
  MinIO with presigned URLs signed against `agstudio.local:9100` — those URLs
  paste straight into a Zulip reply and render as links (browsers on agstudio
  resolve them; noted working for the devworld UI).
- **Two ways to wire listener → agent**, both acceptable:
  (a) listener POSTs the assembled desire to its own `:8092` request pipeline —
  one processing path, but the pipeline is desire-shaped, not chat-shaped;
  (b) listener invokes a chat-role agent run directly with the transcript —
  more natural conversations, slightly more new code. Pick one and say why in
  the report.
- **Latency**: generation jobs are long. Post an immediate short ack DM, then
  the real result when done. A chat where nothing comes back for minutes reads
  as broken.
- **Tool Giving includes usage info**: whatever transcript format you feed the
  agent, document it (and the reply expectations) in agforge's agent guide so
  the in-system agent knows what it is looking at.
- **Single Entrance note**: this adds a conversational entrance alongside the
  `:8092` desire endpoint. That is the intended migration direction (chat
  becomes THE entrance; string telephone retires in a later episode) — record
  the temporary two-entrance state in the report rather than designing around it.

## Steps

### Step 1 — Manual API spike (no code)
From agforge's runtime environment, with curl and the forge bot credentials:
register an event queue, send the bot a DM from the developer's Zulip account,
observe the event arrive on `GET /events`, fetch the DM narrow history, and
post a DM reply as the bot. This validates every API mechanic before any code.
**Done when**: all four operations shown working (transcript in `report1.md`).

### Step 2 — Listener loop
Implement the long-poll listener in agforge (stdlib only, matching the house
style): register → poll → dispatch, with self-message filtering, re-register on
`BAD_EVENT_QUEUE_ID`, and a plain-text log. On DM receipt it can just log the
message at this step. Run it manually in the foreground.
**Done when**: a DM from the UI appears in the listener log within seconds;
killing the queue (or waiting out expiry) recovers without restart.
Produce `report2.md`.

### Step 3 — Context assembly and agent reaction
On DM: fetch the last ~50 messages of that DM conversation, format them as a
speaker-labeled transcript, invoke the agent (route (a) or (b) above), send an
immediate ack, and DM the final result (with presigned media URL when the
request produced an asset). Update the agent guide with the transcript format.
**Done when**: a DM like "make me a small icon of a red bird" yields an ack and
then a result DM with a working link. Produce `report3.md`.

### Step 4 — History awareness and supervision
Two finishing checks:
1. Multi-turn: after step 3's exchange, send a follow-up that only makes sense
   with history ("same bird but blue") and confirm the agent uses the context.
2. Supervision: run the listener the same way the request service is run
   (`serve.sh` sibling or launchd template) so it starts/restarts with the
   service; confirm it survives a Zulip stack restart.
Update `agforge` docs / `.local` notes, write the episode `report.md` including
the two-entrance note and anything that should seed the next episode (other
agents' receive sides, stream/mention handling).
**Done when**: both checks pass and report.md exists.

## Notes for the implementer

- Step boundaries exist to produce verifiable reports; order and tooling within
  a step are yours.
- If the events API misbehaves through the self-signed TLS + container path,
  falling back to periodic `GET /api/v1/messages` polling is an acceptable
  stopgap — note it as debt in the report.
- Keep the listener dumb: no queue persistence, no delivery guarantees. A missed
  DM during a restart is fine for this experiment; the sender can resend.
