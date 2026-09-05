# Step A — what Zulip alone can say about a task's state

Investigated item **A** of `plan.md`: can an outside observer rebuild
"who owes a reply, and for how long" from Zulip reads only, and does the
match survive the `✔ ` resolve rename.

**Answer: yes, and it is the strongest signal this system has.** Both of the
listener's serving routes can be reproduced from public-channel GETs by a
credential that is not the agent, the reconstruction is exact enough that its
disagreements with reality are all explainable, and *stalled* falls out of it
for free as the age of the owed reply.

Probes are throwaway and live in the ignored `agdevworld/.local/opsprobe/`
(`opsprobe.stepa`, `opsprobe.stepa2`, `opsprobe.renamegap`, `opsprobe.peek`).
Nothing was posted; every call is a GET.

## The reconstruction

`agag.zulip` serves an agent two ways, and an observer can compute both.

**Owner route.** `sweep_topics` (zulip.py:1203): an unresolved topic in the
agent's own channel, or under one of its topic prefixes anywhere, whose last
*real* speaker is not the agent. "Real" is `last_real_sender` — selfnotes
dropped — and Zulip's own notices dropped too. Reproducing it needs
`channel_topics` per channel plus one `topic_history` per matching topic.

**Mention route.** `sweep_mentions` (zulip.py:861) plus `served_marks`
(:1021): a topic the agent does *not* own, where a post names it
(`@**<full name>**`) and no `[selfnote][served] <channel>/<topic> <id>` note
written by that agent covers that post. The observer cannot use Zulip's
`is:mentioned` narrow — that is scoped to the credential — but the fallback
`is_mention_for_us` already uses, a literal `@**name**` scan of the raw
(`apply_markdown=false`) body, works from outside and needs no extra calls
once the histories are read. `served_marks` transfers directly: it is
`narrow=[sender:<me>, search:served]`, and asking it about another sender is
the same one call.

**The routes are exclusive, and getting that wrong is the trap.** The first
pass ran the mention differencing over owned topics too and reported
`work-m-32/workrun-task2-m-32` as a 5.9-day-old unanswered call. Reading it
(`opsprobe.peek`) shows autolab answered it in the topic two minutes later.
An owner answers **in place** and writes no served note, so served-note
differencing over an owned topic reports every answered call as pending,
forever. Split by ownership first, then choose the route.

## What it found on the live realm

119 unresolved topics across 54 channels, all histories read:

| agent | served notes | awaiting | stalled (≥15 min) |
|---|---|---|---|
| Front | 326 (155 topics) | 7 | 7 |
| autolab-agstudio1 | 10 (3 topics) | 0 | 0 |
| agforge-agstudio1 | 0 | 0 | 0 |
| arxivsage-agstudio1 | 0 | 0 | 0 |
| agecho-agstudio1 / agecho-agautolab1 / agping-agstudio1 | 0 | 0 | 0 |

The board is genuinely quiet, and the probe says so. The seven Front rows are
all `front/routine-*` — see the false positives below.

## The ✔ rename, measured

The plan asks whether the match works across the resolve rename. It does not,
unless the key is the bare topic name, and the size of the failure is now a
number rather than an argument.

Matching a served note's topic **verbatim** against the topic as it exists
today, over Front's 326 notes:

- 121 name a topic still live under that exact name
- **116 name a topic that exists only under its `✔ ` name** — resolved after
  the note was written
- 89 name a topic in a channel that has since been archived
  (`pj-simpleshooter`, `work-s2-…`), reachable under neither name

For autolab's 10 notes: 0 / 9 / 1. Verbatim matching therefore loses **36% of
Front's index and 90% of autolab's**, and every loss reads as "this call was
never answered". End to end that is 135 false pending calls for Front versus 1
under bare-name matching — the p9 accident, reproduced as a metric.

`mark_served` writes the note under `live_topic_name`, so the note is correct
at the moment it is written; the drift is entirely later resolves. Strip
`RESOLVED_TOPIC_PREFIX` from both sides of every key. `agentroom.room.bare_topic`
already does this for the agent room and is the same one line.

## Stalled

Stalled is not a new signal: it is `age_min` on a row the reconstruction
already produces — now minus the timestamp of the post that owes the reply.
No extra read, no extra state. A 15-minute threshold is a starting point, not
a finding; the p9 incident was 26 minutes.

The value is that it is computed **outside** every agent. An agent that is
not running cannot report that it is not running, and this is the one signal
that does not ask it to.

## False positives, and what they are made of

Both classes found are structural, not noise.

**1. Registry topics.** All seven Front rows are `front/routine-*`: a topic
whose newest post *is* the standing definition of a routine
(`front/routine-imgprompt`, one Developer post, 12.9 days old, no reply
expected). Front's filter is `channel == "front"`, so every topic in its
channel is swept, and by the last-speaker rule these await Front forever.
This is not the probe being wrong — the real listener computes the same thing
— but the operation room must not paint 7 red rows for it. **Design note:** the
observer needs an exclusion, and `routine-` is a prefix it can be told about;
the honest version is that `#front` is doing two jobs in one channel.

**2. Prefix ownership does not identify an instance.** The bot
`Autolab Agautolab1` came out with 59 awaiting rows, every one of them a
`workplan-`/`workrun-` topic that `autolab-agstudio1` is actually serving.
Sweep prefixes are compiled into each agent's `AgentSpec`
(`agautolab/src/agautolab/instance.py:45`) and two instances of the same agent
carry the same ones. **A topic prefix says which *kind* of agent owns a topic,
never which instance.** The instance is decidable only from the channel
(`channel == instance_name`), and outside the instance's own channel it is not
decidable at all from the topic name.

The fix for the roster is the `#agents` intro topics rather than the bot list:
six instances have an `intro-` topic and `Autolab Agautolab1` is not among
them. But `Front` has no intro topic either, so the roster is *intro topics ∪
known non-standard agents* — not one clean source.

## What an observer cannot get from Zulip

- **Sweep prefixes.** Compiled into each agent's code. The observer must be
  told them, or parse them out of the introduction post, which states the
  vocabulary as prose. Today the probe hard-codes a table.
- **Whether the listener is alive.** "Awaiting for 18,626 minutes" is
  identical in shape whether the agent is dead, throttled, or looking at a
  topic nobody meant it to answer. Zulip cannot separate those; step B is
  where that separation has to come from, if anywhere.
- **Whether a run is in progress right now.** Serving posts an ack
  (`SWEEP_ACK`, "Message received. Please wait for the reply."), which makes
  the agent the last real speaker and drops the topic out of "awaiting" — so
  the observable is *ack posted, reply not yet posted*, which is a real
  execution signal but arrives only after the ack. Anything the run does
  before the ack, and any run not started by a topic, is invisible here. Step
  B.
- **Non-agag participants.** `Comfy Notifier` was named 42 times and has
  written 0 served notes — it acknowledges with a reaction on purpose
  (README_DEV). `Omni Agent`, 17 mentions, 0 notes. Served-note differencing
  applies only to agents that write the notes; for everyone else a mention is
  not evidence of anything.

## Cost, and the one hard limit

A full reconstruction is `1 + channels + unresolved topics + agents` calls —
**183 on this realm**, about 8 seconds. Repeating it immediately returned
**HTTP 429** from Zulip. That is a design constraint on the operation room and
not a small one: this view cannot poll the realm on a short timer the way a
process monitor can. `agag.zulip` already reserves headroom for exactly this
(`SWEEP_BUDGET_RESERVE = 40`, zulip.py:67) and the agents' own listeners are
spending from the same realm quota — a greedy operation room would throttle
the agents it is watching.

Two ways out, both for p2: read the newest N messages in one narrow
(`opsprobe.stepa` does the whole realm in 16 calls for 3,000 messages) and
keep an incremental cursor instead of re-sweeping; or hold the reconstruction
in a process and refresh it on a Zulip event queue, which is what the agents
themselves do. The agent room's 30-second in-memory cache is the shape, but
30 seconds is far too aggressive at this call count.

## Verdict for the state model

Derivable from Zulip, per (agent, channel, topic), cheaply and reliably:

- **awaiting** — the agent owes a reply here, by whichever route
- **stalled** — awaiting, and older than a threshold
- **acked, not answered** — the coarse "running" proxy, available only after
  the ack post
- **done** — the topic carries the `✔ ` prefix

Not derivable: planned-but-not-started as distinct from awaiting (a post is
what starts a task; before the post there is nothing to observe), and anything
about the process behind the reply.
