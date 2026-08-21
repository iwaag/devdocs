# p7 step 1 — pyagag: the callback

AI-generated (Omni Agent, 2026-08-21).

`agentchat wait` is gone, and what replaces it is being called back. pyagag
`2890a29`, pushed; all three consumers re-locked onto it.

## What was built

**`agag.participation` — the ledger.** One JSON object per line:
`{"remote": "<channel>/<topic>", "home": "<channel>/<topic>", "message_id",
"at"}`. `remote` is where a run posted; `home` is the conversation it was
serving, read out of `AGENTCHAT_HOME` in its environment. `home_for(remote)`
is the lookup a listener does when a mention arrives; `remotes_for_home(home)`
is the list of threads a run is party to. String operations and a file — no
model, and no decisions: the ledger records, the listener rules.

A line that is not JSON is skipped rather than fatal, so a crash mid-append
costs one participation and not every conversation the agent is part of.

**`agentchat send` participates.** Before it posts it calls
`ZulipClient.ensure_subscribed(channel)` — before, not after, because the
answer can arrive in seconds and only a subscribed channel's messages reach
the event stream. After it posts it appends the participation. A run with no
`AGENTCHAT_HOME` records nothing, which is right for a run nobody will call
again. A subscription that fails prints and lets the message through: the
post matters more than the callback.

`read` and `topics` still touch no subscriptions at all. The old blanket rule
("this never touches subscriptions") became a split one: **looking is free
and invisible, posting joins.** That is p2's "subscription is the routing
decision" kept, with the decision moved to where it is actually made.

**`sweep_serve(on_mention=…)` — the second trigger.** The owner of a topic is
served by anybody else's post in it (unchanged). A participant is served only
when a post *names* it. Mentions have their own pending set, drained after
the owner set, and a topic that `topic_filter` already matches is served as an
owner topic and never as a mention — owning beats being named, or one post
would buy two runs.

Detection is Zulip's own `mentioned` flag off the event, with a scan for
`@**<bot name>**` as the fallback for payloads that arrive without flags.
Recovery is `is:mentioned` (`ZulipClient.mentions`) on every queue
registration, beside the full topic sweep — so the mention route is exactly as
lossless as the owner route, and a mention that landed while the listener was
down is found at startup. The route is opt-in: a listener that passes no
`on_mention` spends no calls on any of it.

**`serve_topic(reply_to=…)` — work here, answer there.** A run brought back by
a mention serves its own conversation (its workspace, its chatlog, its task)
and posts the ack, the reply and the resolve into the topic that called it.
`TopicContext` gained `reply_channel` / `reply_topic`, `replies_here`, and
`post_home()` for a handler that wants to say something in the served topic
anyway — which is what keeps autolab's `RunProgress` posting into the workrun
topic while the answer goes to forge.

The post-run re-check loop is skipped for a mention serving, on purpose: this
bot never became the served topic's last poster, so whatever arrived there is
found by the ordinary owner sweep instead of being served twice.

**`handoff_mention` — the turn, handed over by code.** Every reply is prefixed
with `@**<name>**` of the last speaker in the reply topic who is not this bot.
Whose turn it is next is not etiquette to be remembered; it is the mechanism
by which the next run happens at all. No guide asks for it, and none has to.
A topic nobody else has spoken in gets no prefix — there is no turn to hand
back yet.

**`write_threads`** renders `threads/<channel>/<topic>.md` beside
`chatlog.md`, same renderer, one file per conversation the ledger says this
run is party to. A thread that cannot be read is logged and skipped: a missing
file is a run with less context, an exception is a run that does not happen.

## What was deleted

`wait`, `_wait`, `DEFAULT_WAIT_TIMEOUT`, `DEFAULT_WAIT_INTERVAL`,
`TIMEOUT_EXIT_CODE`, the seven `wait` tests, the `import time`, and every
sentence about waiting in `USAGE_DOC` and the README. `agentchat --help` now
offers `{send,read,topics}` and says instead:

> Say what you want and finish. You will be brought back when somebody
> answers you, with their conversation in front of you — so there is nothing
> here to sit and watch, and nothing is lost while you are not running.

`read --since` stays, as the plan asks: it is how a returning run catches up
when the thread file is not enough. It still follows a topic across the `✔`
rename.

## Evidence

- pyagag `2890a29` — `A run is one reply: the callback replaces the wait`,
  pushed to GitHub.
- **312 tests green** (269 before): +10 `test_participation.py`,
  +8 `test_topics.py` (handoff, reply target, threads), +9 `test_zulip.py`
  (`ensure_subscribed`, `mentions`, `mention_from_event`, `sweep_mentions`,
  the two-trigger cases), +5 `test_chat.py` (join before post, ledger, no
  home, reading joins nothing, a failed subscription), −7 (`wait`).
- `grep -rn 'agentchat wait' src tests README.md` → 0 hits.
- Re-locked on `2890a2966fb7d1bf9e50a59727d026c112596eef`: agautolab
  `uv.lock:80`, agforge `uv.lock:285`, agfront `uv.lock:63`. Those lock bumps
  land with step 2, because the new reply prefix breaks 18 consumer tests
  whose expectations step 2 is what updates.

## Decisions taken inside the step

- **Where a mention resolves to its home is the consumer's business.**
  `on_mention(channel, topic)` hands the listener the *remote* topic; the
  listener looks it up in its own ledger and decides what an unknown topic
  means (entrance question, or ignored-and-logged). pyagag stays ignorant of
  the policy, as the plan leaves to the implementer.
- **Resolve follows the reply, not the serving.** Consistent one rule: ack,
  reply and `✔` all act on where the run is speaking. In practice only
  owner-path handlers set `resolve_after`, so it is the served topic anyway.
- **`handoff_mention` swallows every exception**, not just `ZulipError`. A
  reply lost because the mention lookup failed would be worse than a reply
  that hands the turn to nobody.

## Left for later

- Silent mentions (`@_**…**`) for a reply that is an ack with no question.
  Without them, every reply is a turn handed over, so an exchange that has
  nothing left to say still costs one run per side until something resolves
  the topic. Named in the plan as a refinement; carried to `report.md`.
- Ledger garbage collection: nothing prunes `participations.jsonl`, and
  `remotes_for_home` renders every thread a home ever touched.
