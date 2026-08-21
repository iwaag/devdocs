# p8 step 3 — agfront: adopt and trim

AI-generated (Omni Agent, 2026-08-21). Commit `586cc6f` in `agfront`, pushed
to GitHub. pyagag re-locked `065de12` → `6d5853d`.

## What changed in code

Less than the plan expected in one place and more in another.

- **`role_run.py`** — `AGENTCHAT_LEDGER` and the `agag.participation` import
  are gone. The run's tool environment is `AGENTCHAT_ZULIP_ENV`,
  `AGENTCHAT_HOME` and `PATH`; the home is the whole handover, because
  `agentchat send` now writes it into whatever topic the run posts in.
- **`zulip_listener.py`** — two lines of substance:
  - `threads/` comes from `agag.zulip.remotes_for_home(client, …)` — the
    `sender:me search:rootchat` narrow — instead of the ledger file.
  - `handle_mention` resolves its home with
    `agag.zulip.rootchat_home(client, channel, topic, self_id)`, and serves
    that conversation **without `reply_to`**. That last part is the phase's
    substance and is the deviation from "nothing else in code": p7 answered
    into the topic that called, so every progress report was a post in
    another agent's conversation, and a post in somebody's topic serves them.
    Now the reply goes home. A mention in a topic Front never anchored is
    logged and dropped, exactly as p7 chose.

The stale `.local/agentchat/participations.jsonl` was deleted; nothing reads
it.

## The guide

Unchanged from what the developer had already written into it before this
phase (committed here with the code):

> Your reply to this conversation will be sent to the developer directly.
> …
> If the developer accepted your plan, or plan is already going on, keep
> talking with the other agents to fulfil the request, and report progress in
> your reply. Report must include channel name and topic name you've talked
> in, and what other agent told you. To talk other agent, command
> `agentchat --help` to learn how.
>
> If you think task is already done, just reply so.

**The first line is now literally true for every serving**, which is the
point of step 3. In p7 it was false exactly when Front was brought back —
the case where it mattered — and Front believed it, wrote a status report for
the Developer, and posted it into autolab's topic, which woke autolab. That
mismatch cannot occur any more: there is one reply target and it is home.

The last line is the terminator. Nothing else in the system decides that an
exchange is over.

## Tests

25 pass (`uv run pytest`). The mention test now asserts the opposite of what
it asserted in p7 — everything posted goes to `#front`, nothing to the
calling topic — and adds that the root note does not appear in the rendered
thread. The `test_role_run.py` seam test no longer looks for
`$AGENTCHAT_LEDGER` in the stub run's environment.

## Kickstart, and the three listeners

| listener | pid | state |
|---|---|---|
| `com.agdev.agfront-zulip` | 42420 | started 05:28:54Z, sweeping as user 15 |
| `com.agdev.agforge-zulip` | 41793 | started 05:26:01Z, sweeping as user 13 |
| `com.agdev.agautolab-zulip` | 42532 | started 05:29:30Z, sweeping as user 11 |

Front's startup recovery found the stale p7 mention and declined it with the
new rule, which is the first live evidence that the rule works:

```
2026-08-21T05:28:55Z full sweep: 0 awaiting, 1 mentioning, 26 calls spent, 978 left in the window
2026-08-21T05:28:55Z serving mention in 'pj-simpleshooter'/'workplan-shield-pickup-icon'
2026-08-21T05:28:55Z mention in 'pj-simpleshooter'/'workplan-shield-pickup-icon' carries no root note of ours; ignoring
```

**agautolab is deliberately unchanged and still on the pre-p8 pyagag**
(`065de12`) — it imports `agag.participation`, so it cannot take the bump
until the later phase adopts selfnotes. It is running as the third witness
and it is safe to run: its bot (user 11) is subscribed to `agents`,
`autolab-agstudio1`, `general`, `ops`, four `pj-*` and five `work-*`
channels, and **not** to `agforge-agstudio1` or `front`, so no topic of this
proof is in its sweep at all. On its own startup it declined a stale
`create-` mention through the old ledger rule.
