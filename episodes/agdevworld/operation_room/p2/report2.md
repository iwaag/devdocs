# Step 2 — an observation identity of its own

p1 measured the cost of watching the task layer with a borrowed credential: a
full sweep is 183 Zulip calls, an immediate repeat returns HTTP 429, and the
quota it spends is the quota the agents' own listeners need. The agent room
relay borrows the Developer's account and is defensible at ~50 calls per
30 seconds; this one is not.

## The bot

`Opsroom Observer` (`opsroom-bot@…`, user id 22), created through the
provisioner credential named by `AGAG_ZULIP_ADMIN_ENV` — `agag.zulip.create_bot`,
the same call `agag init --provision` makes for a real agent. Its credential
went to `pj-agdev/.local/zulip/opsroom.env`, mode `0600`, alongside the other
ignored bot envs. Nothing about the path, the realm or the key is in a tracked
file.

Read-only is an operating rule, not a permission: **Zulip has no read-only API
key.** The rule is enforced by the code that holds the credential having no
route that posts, and by the one exception below being a channel nothing
watches.

## The one channel it may post in

`#ops-testbed`, created for Step 5's stalled proof. Its subscribers are
`Provisioner` and `Opsroom Observer` and nobody else — verified, not assumed —
so a post there cannot serve an agent by the owner route, which iterates the
*subscribed* channels of the agent doing the sweeping.

The mention route is the one that would still reach out of the room: `#front`
taught this system that a public channel's mentions are visible to a bot that
is not in it (`zulip_command` step 4, one paid Front run per lap). So the rule
for the testbed is narrower than "it is a test channel": **no post there ever
names a real agent bot.**

## The finding that shaped Step 3

The plan says the observer must run on an event queue rather than a sweep, so
the question was whether a queue delivers what a bot can *read*. It does not,
and the difference is exactly the subscription:

| | read (`GET messages`) | event queue |
|---|---|---|
| public channel, subscribed | ✅ | ✅ |
| public channel, **not** subscribed | ✅ | ❌ |

Measured: the bot read `#agents` and `#front` while subscribed to nothing, and
a queue registered for `message` + `update_message` received the post made to
its own `#ops-testbed` and **not** the one made to `#sandbox`.

So the observer must subscribe to every channel it watches, and must keep
doing so — `work-<label>` channels appear whenever autolab opens a task. That
is a *write* to the realm, but not a post, and it is the only one the observer
makes. Step 3 does it at startup and at every resync.

The same probe confirmed the other half of the event-driven design: resolving a
topic emits

```
type: update_message  stream_id: 117  propagate_mode: change_all
orig_subject: "queue-probe"  subject: "✔ queue-probe"
```

— both names in one event, which is what lets the state engine follow a `done`
without re-sweeping. It also emits a `message` event immediately after, from
Zulip's own Notification Bot ("marked this topic as resolved"); the engine
drops the `zulipinternal` realm, as `agentroom.room` already does.

## Cost, before and after

Nothing changed about the size of a sweep — 183 calls is a property of the
realm. What changed is whose budget it comes out of, and that the sweep now
happens twice in a relay's life (start, and resync after a queue expires)
rather than on a timer.

## Changed

- Zulip realm: bot `Opsroom Observer` (22), channel `#ops-testbed`.
- `pj-agdev/.local/zulip/opsroom.env` — ignored, `0600`, not committed.
- Nothing in any repository yet; Step 3 is the first code that uses it.
