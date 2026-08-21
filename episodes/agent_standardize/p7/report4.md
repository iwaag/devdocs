# p7 step 4 — the proof run, and where it stopped

AI-generated (Omni Agent, 2026-08-21). **Stopped for a decision the plan
deliberately deferred.**

Two attempts. The first found a real defect in four minutes and was aborted.
The defect was fixed and verified live. The second attempt got as far as a
plan, a go, and `start.flag` — and then **never left the `workplan-` topic**,
because Front and autolab hand each other the turn forever.

| criterion | verdict |
|---|---|
| 1. the delegation completes with zero human/Omni posts after the go | **not met** — never reached forge |
| 2. no run exceeds the reverted ceilings (Front 360 s, work 1200 s) | **met** — Front max 45.2 s, superdirector max 87.1 s |
| 3. `agentchat wait` no longer exists; no waiting vocabulary anywhere | **met** ([report1](report1.md), [report3](report3.md)) |
| 4. guides and intros at or under target | **met** ([report3](report3.md)) |

## Attempt 1 — the ack was a trigger

`front-20260821-p7-image` (804) → `pj-simpleshooter/workplan-add-powerup-icon`
(809). autolab planned in 38 s and replied naming Front (814). The mention
route brought Front back — **this part worked exactly as designed, on its
first live run.**

Then:

```
814 autolab  @**Front** I've written the plan and a two-task breakdown…
815 Front    Message received. Please wait for the reply.      ← the ack
816 autolab  Message received. Please wait for the reply.      ← served on the ack
817 Front    Go — the plan and two-task breakdown look right…
818 Front    @**autolab-agstudio1** I reviewed autolab's plan…
819 autolab  @**Front** Nothing new has come in from the requester…
```

Front, serving a mention in autolab's topic, **acked there**. autolab owns
that topic, so "Message received" was a post from somebody else, so autolab
served its own topic — reading a chatlog whose newest line was an ack, and
answering that nothing had arrived. Its reply named Front, which brought
Front back, which acked again. Two runs per cycle, neither about anything,
while the real reply (817) was still being written. autolab said
"`start.flag` created" three separate times (823, 827, 830) because three
separate runs read three near-identical chatlogs.

**Fixed: a serving that answers in somebody else's topic posts once and does
not ack.** An ack is how a bot's own sweep skips a topic it is already
serving; in a topic it does not own it buys nothing and costs the owner a
run. pyagag `065de12`, re-locked into all three agents (`8c1a587`,
`709dbc4`, `52d8c94`). 317 pyagag tests green, with the live sequence written
into the regression test's docstring.

Verified in attempt 2: every Front serving posted exactly one message.

## Attempt 2 — the turn is never given up

Clean slate: the aborted mission's Works cancelled in Plane, its topics
resolved, `work-s2-14` archived, Front's ledger set aside.

`front-20260821-p7-shield` (849). The good half:

| # | at | who | what |
|---|---|---|---|
| 849 | 03:04:03 | Developer | the mission and the go |
| 851 | 03:04:26 | Front | opens `pj-simpleshooter/workplan-shield-pickup-icon` — **and ends** |
| 853 | 03:04:29 | Front | tells the Developer where it went |
| 856 | 03:05:10 | autolab | plan, `task1.md`, `task2.md`, Work S2-17, channel `work-s2-17`, topics `workrun-task1-s2-17` / `-task2-`, naming `@**Front**` |
| — | 03:05:10 | | the mention route serves `front-20260821-p7-shield` |
| 857 | 03:05:29 | Front | "The plan looks right — go ahead and start task 1." |
| 862 | 03:05:55 | autolab | `start.flag` created; S2-17 In Progress |

Front's ledger, live:

```
{"remote": "pj-simpleshooter/workplan-shield-pickup-icon", "home": "front/front-20260821-p7-shield", "message_id": 851, "at": "2026-08-21T03:04:27+00:00"}
{"remote": "pj-simpleshooter/workplan-shield-pickup-icon", "home": "front/front-20260821-p7-shield", "message_id": 857, "at": "2026-08-21T03:05:29+00:00"}
```

**Every mechanism p7 built worked.** Front posted and finished. The
participation was recorded. The subscription was made. The mention brought
Front back with autolab's thread in its workspace. The turn was handed over
without any guide asking for it. Not one run waited for anything, and not one
came near its ceiling.

And then it would not stop:

```
859 Front    @**autolab-agstudio1** autolab's plan matched the spec…
860 autolab  @**Front** The plan and both tasks were already registered correctly…
862 autolab  @**Front** The plan and both tasks already match what the requester confirmed…
863 Front    @**autolab-agstudio1** Status update: the mission is moving…
865 autolab  @**Front** The plan and both tasks already match what was confirmed…
```

Four autolab replies saying the same thing, three Front replies saying the
same thing, and the listeners stopped only because they were shut down by
hand. `work-s2-17/workrun-task1-s2-17` still holds exactly one message: the
task description autolab posted when it opened it.

## Why it does not stop

Three rules that are each correct compose into a cycle:

1. **The owner of a topic is served by any post from somebody else.** (The
   pre-p7 rule; a human stops posting, so it terminated.)
2. **A participant is served only when a post names it.** (p7's second
   trigger — necessary, or nothing comes back.)
3. **Every reply names the last other speaker.** (p7's mechanical handoff —
   necessary, or rule 2 never fires.)

So A's reply names B, which serves B; B's reply names A *and* is a post in
A's topic, which serves A by rule 1; A replies, naming B. Neither side ever
declines the turn. With a human on one end this is invisible, because the
human simply stops. With two agents there is no stopping rule at all.

`serve_topic` already supports the exit — a handler returning no sections
posts nothing, and the exchange dies. What is missing is anything that makes
an agent *use* it. The plan foresaw this and set it aside:

> Silence (`@_**`) where a reply is an ack with no question would be a
> refinement — not now.

Live, it is not a refinement. It is the terminator.

## The second half of the same problem

Front never posted into `workrun-task1-s2-17`, so task 1 never started. That
is not only the loop: **a mention-triggered run answers in the topic that
named it, and that is the one place from which the mission cannot advance.**
Starting task 1 means posting somewhere else, which Front can only do with
`agentchat` — a deliberate act it did not take, because its guide says its
reply is its report and its reply was already landing where autolab was
listening.

The same seam shows up as double-speaking: 857 (`agentchat send`) and 859
(the run's reply) are the same message twice, because "report in your reply"
and "post to the agent" now name the same topic.

## What was measured

Every run, both attempts — cap 360 s for Front, 1200 s for autolab's roles:

| role | runs | shortest | longest |
|---|---|---|---|
| front | 8 | 16.8 s | 45.2 s |
| superdirector | 8 | 8.0 s | 87.1 s |

Nothing came within a factor of four of its ceiling. **The reverted ceilings
are not a constraint on this system**, which is the same conclusion p6
reached about raising them — the ceiling was never what bound.

The delegation legs (supercoder → forge) never ran, so there is no
per-delegation wall clock, exactly as in p6. That is the second phase running
aground before the same hop.

## What is left to decide

The plan's Decisions section fixed rules 1–3 and deferred the fourth. The
evidence says the fourth is not optional, and choosing it is the developer's
call, not something to slip into an implementation:

- **Silent mention** (`@_**name**`) when a reply asks nothing — a Zulip
  mention that notifies nobody, so it does not serve the other agent. Needs
  something to decide "asks nothing": a flag file from the run (forge's
  retired `question.flag` was exactly this shape), or a heuristic in the
  handler, or the agent choosing between two verbs.
- **No handoff to whoever just handed you the turn** unless you have a
  question — deterministic, in the code, no agent judgement, but it also
  silences a genuine second question.
- **A reply with nothing to add is not posted** — `TopicResult([])` already
  does this; what is missing is who decides, and p6's lesson is that a guide
  sentence is not enough.
- **Front's reply target** — whether a supervisor's answer should go to the
  topic that named it, or whether supervising means posting into the *next*
  topic, which is a different thing from replying.

## Deus Ex Machina

The Omni Agent posted as the Developer (mission, go) — that is driving a
test. It also **stopped both loops by hand**: resolving topics, cancelling
Plane Works, archiving a channel, shutting listeners down. Nothing inside the
system noticed that two of its agents were saying the same thing to each
other four times, and nothing inside it could have. **Handoff candidate: a
loop is a thing the system should be able to see.**
