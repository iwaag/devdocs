# zulip_receive — Step 4 report: history awareness and supervision

Date: 2026-08-12. Status: **complete**. Both finishing checks pass. The
supervision check found and fixed a real defect that only a restart could
have revealed.

## Check 1 — multi-turn history

Done as part of Step 3 and reported there: after "make me a small icon of a
red bird" was answered with a link, the bare follow-up "same bird but blue"
was answered as a continuation — "here's the blue version: <url>" — and
produced a bird. Nothing but the transcript could have supplied the subject.

A third exchange after the restart, "what kinds of assets can you make? just
answer in words, no generation needed", was answered in 8.9 s from the
capability card without generating anything. So a chat message is not
assumed to be a generation request; the agent reads the conversation and
decides.

## Check 2 — supervision

The listener now runs the same way the request service does: a launchd
template `devenv/launchd/com.agdev.agforge-zulip.plist.in`, sibling to the
existing `com.agdev.agforge.plist.in`, with `RunAtLoad`, `KeepAlive`, and its
log under the project's ignored `.local/out/`. It was installed and
bootstrapped on the workstation.

- **Process killed** → launchd restarted it within the 10 s throttle window
  and it re-registered by itself.
- **Zulip stack restarted** (`docker compose restart` of the whole stack) →
  see below.

### The defect the restart found

The first restart killed the listener outright, three times in a row, and
only launchd's `KeepAlive` brought it back. Two causes, both fixed:

1. **`whoami()` was outside the retry loop.** The bot's own user id was
   resolved once at startup; while Zulip was still coming up that call
   failed and took the process with it. The lookup now happens inside the
   loop and retries like anything else.
2. **`urlopen` does not wrap every connection failure.** A connection
   dropped *while the response is being read* raises
   `http.client.RemoteDisconnected` directly rather than `URLError`, so it
   escaped the handler — which is exactly what a long poll experiences when
   the server restarts underneath it. The client now also catches
   `http.client.HTTPException` and `OSError`.

After the fix, a third `docker compose restart` was ridden out **in place**:
the same process (PID unchanged) logged the dropped poll, retried the
re-registration every 5 s for about 20 s, registered a new queue, and
answered the next DM normally. No launchd intervention.

```
zulip call failed: GET events -> RemoteDisconnected('Remote end closed connection without response'); retrying in 5s
zulip call failed: POST register -> <urlopen error [SSL: UNEXPECTED_EOF_WHILE_READING] …>; retrying in 5s
… (two more) …
registered event queue 0ba1…  (last_event_id=-1)
chat run 4937e78e…: 12 messages of context, partners=[8]
chat run 4937e78e…: … model=anthropic/claude-sonnet-5 … duration_ms=8947 status=ended
```

This is the episode's clearest Failure Farming result: the mechanism looked
finished after Step 3 and was not.

## Docs updated

- `agforge/service/GUIDE.md` — the chat entrance, the transcript format the
  run receives, and the `reply` field it is asked to write.
- `agforge/README_DEV.md` — `service/listen.sh` in the map, and a "Chat
  contract" section stating that both entrances share one pipeline.
- `pj-agdev/.local/devenv.md` (ignored) — how the listener is installed,
  reloaded, and observed; that every DM costs an agent run; and the
  correction that the request service is under launchd after all, which the
  memo still described as hand-started.

## Agent run record

- Request/job id: `zulip_receive/step4`; the live exchange was agforge job
  `4937e78ec0e443a6b24416acba9a12d1` (forge agent, `sonnet` profile,
  Claude Code + `claude-sonnet-5`, 8.9 s, 0.13 USD).
- Omni Agent: Claude Code harness, `claude-opus-5[1m]`.
- Outcome: done.
