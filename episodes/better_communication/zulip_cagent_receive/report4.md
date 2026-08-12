# zulip_cagent_receive — Step 4 report: the listener

Date: 2026-08-12. Status: **complete**. A direct message to the Cagent bot
becomes one `POST /window`. A message reporting a defect produces an ack, an
incident file, and a reply naming the record — with no repair attempted — and
the listener survives a restart of the Zulip stack in place.

## What was built

- `cagent/src/cagent_api/zulip_window.py` — the adapter: DM → last 50 messages
  as a speaker-labelled transcript → `POST /window` → ack DM → poll →
  answer DM. It holds no cluster capability of its own; everything it can
  cause is bounded by the window's permission set.
- `cagent/service/zulip_listener.py` — launcher, matching agforge's layout.
- `devenv/launchd/com.clusterintent.cagent-zulip.plist.in` — supervision, same
  shape as the other four jobs. Log: `.local/cagent-window/zulip-listener.log`.

Choices worth naming:

- **The listener names the reporter, the window records it.** The composed
  text tells the window to use exactly `--reporter "zulip:<id> <name>"
  --source zulip-dm --ref "zulip message <id>"`, so the record traces back to
  a message without the window having to invent identifiers. Numeric ids
  because this realm hides email addresses.
- **The ack goes out after the request exists**, so it can name the
  `request_id`, and before the wait, because a silent chat reads as broken.
  Acks are filtered out of the transcript on the next turn.
- **A turn that produces no answer still gets a reply** ("That turn did not
  produce an answer (…)"), and the poll gives up after
  `CAGENT_WINDOW_TIMEOUT_SECONDS` (default 420) rather than hanging forever.
- **`CAGENT_ZULIP_LOG_ONLY=1`** watches DMs without spending a turn, same
  passive mode agforge has.
- Dumb by design: no queue persistence, no delivery guarantees. A DM missed
  during a restart is the sender's to resend.

## The acceptance run

DM to the Cagent bot (from the Omni Agent account, id 9):

> You told me node agpc was up half an hour ago. It is not — it does not
> answer at all. Please deal with this.

Chat, within 6 seconds:

```text
[Cagent] Got it — thinking. (request `req_de421f2e…`)
[Cagent] Recorded the incident at
         `.local/cagent/incidents/20260812T070752Z-you-told-me-node-agpc-was-up-half-an-hour-ago.md`.
```

The file:

```text
---
id: 20260812T070752Z-you-told-me-node-agpc-was-up-half-an-hour-ago
time: 2026-08-12T07:07:52Z
reporter: zulip:9 Omni Agent
source: zulip-dm
ref: zulip message 46
---

You told me node agpc was up half an hour ago. It is not — it does not answer
at all. Please deal with this.
```

Listener log for the same turn:

```text
window request req_de421f2e…: 3 messages of context, partners=[9]
window request req_de421f2e…: state=completed cost_usd=0.00207714
  backend={'harness': 'opencode', 'provider': 'openai', 'model': 'gpt-5.6-luna'}
```

**No repair was attempted** — the only tool call the permission layer saw for
this turn was the incident script, allowed by
`uv run cagent/window/incident.py*`. "Please deal with this" did not become a
reconcile, a re-check, or an argument about whether the node is really down.

## Zulip restart

`docker compose restart` of the Zulip stack at 07:08:48Z. The listener logged:

```text
07:08:48Z zulip call failed: GET events -> RemoteDisconnected(...); retrying in 5s
07:08:53Z zulip call failed: POST register -> <urlopen error [SSL: UNEXPECTED_EOF_WHILE_READING] ...>; retrying in 5s
07:08:58Z zulip call failed: POST register -> <urlopen error [SSL: UNEXPECTED_EOF_WHILE_READING] ...>; retrying in 5s
07:09:04Z registered event queue 51c5…
```

Back in **16 s**, same PID (1309, started 07:07:30Z) — recovered in place, not
restarted by launchd. This is the shared `agag.zulip` loop doing exactly what
Step 1 moved it there for.

A DM sent afterwards ("what has been reported to you lately?") was answered in
10 s for $0.0027 with the three incidents recorded during this episode, listed
from `incident.py --list`. So the post-restart path is proven end to end, and
the "what has been reported" question works as a chat question.

## Notes

- Tests: `cagent/tests/test_zulip_window.py` (9) pins the transcript, the
  reporter/source/ref the window is handed, ack-then-answer ordering, the
  no-answer reply, and the poll timeout. Full cagent suite: 119 passed.
- The recorded incidents name real hosts and addresses and stay in the ignored
  `.local/cagent/incidents/`; only the wording above is quoted here.
