# p7 problem — two agents in one topic never stop, and neither is told where it is speaking

AI-generated (Omni Agent, 2026-08-21). Written from the Step 4 proof run;
the trail with message ids is in [report4.md](report4.md).

p7's machinery works. A run posts and ends; the participation is recorded;
the answer names the agent and brings it back with the other conversation in
its workspace. That was true on the first live attempt and every attempt
since, and no run in either attempt came within a factor of four of its
ceiling.

Two things are wrong, and they are different in kind. The first is a design
gap the plan named and set aside. The second is a plain mistake.

## Problem 1 — the turn is never declined

Three rules, each of them right on its own:

1. **The owner of a topic is served by any post from somebody else.**
   Pre-p7, and correct: a topic is a request, and a request is answered.
2. **A participant is served only when a post names it.** p7's second
   trigger. Without it, nothing ever comes back.
3. **Every reply names the last other speaker.** p7's mechanical handoff.
   Without it, rule 2 never fires.

A's reply names B, which serves B by rule 2. B's reply names A *and* is a
post in A's topic, which serves A by rule 1. A replies, naming B. Nobody
declines.

With a human on one end this was invisible for four phases, because a human
simply stops posting. Two agents have no stopping rule at all.

Observed in `pj-simpleshooter/workplan-shield-pickup-icon`: autolab said the
same thing four times (856, 860, 862, 865), Front three times (857/859, 863),
and the exchange ended only because the listeners were shut down by hand.
`work-s2-17/workrun-task1-s2-17` still holds exactly one message — the task
description autolab posted when it opened the topic. **The task never
started.**

`serve_topic` already has the exit: a handler returning no sections posts
nothing, and the exchange dies. What is missing is anything that makes an
agent use it. The plan foresaw this and deferred it:

> Silence (`@_**`) where a reply is an ack with no question would be a
> refinement — not now.

It is not a refinement. It is the terminator, and there is no other.

A related consequence, worth stating separately because it may need its own
answer: **a mention-triggered run answers in the topic that named it, which
for a supervisor is the one place the work cannot advance from.** Starting
task 1 means posting into a *different* topic — a deliberate `agentchat` act,
not a reply. Front never took it.

## Problem 2 — the run is not told where it is speaking

A run brought back by a mention is started with **the same guide, and the
same shape of prompt, as an ordinary serving.** Nothing says it was called
back, and nothing says where its reply will go. The only hint is one
placement line: `One of them is why you are running.`

| | ordinary serving | mention-triggered serving |
|---|---|---|
| guide | the role's guide | **the same file** |
| chatlog | that topic | **the home topic**, not the one that called |
| `threads/` | from the ledger | the same |
| ack | posted | not posted (`065de12`) |
| reply goes to | that topic | **the topic that named it** |
| said in the prompt | — | **nothing** |

This is not a subtlety. Front's guide opens with:

> Your reply to this conversation will be sent to the developer directly.

For a mention-triggered run that sentence is **false**. The reply goes to
`pj-simpleshooter/workplan-shield-pickup-icon`, prefixed with
`@**autolab-agstudio1**`. Front believed the guide, wrote a progress report
addressed to the Developer —

> `@**autolab-agstudio1**` Status update: the mission is moving. I confirmed
> autolab's plan … mission S2-17 is now "In Progress" …

— and that report landed in autolab's topic and woke autolab. **A status
report for the requester is the fuel of Problem 1.**

The same mismatch produces the double-speaking: 857 (`agentchat send`) and
859 (the run's reply) are the same message twice. Front reported in its
reply as the guide asks, *and* sent to autolab with `agentchat` because it
did not know its reply was already going there.

The supercoder guide has the mirror-image version of the same false promise:

> Post the request and finish. You will be called again when they answer, and
> the result goes into this task's own topic.

On the call-back, the reply goes into forge's `assetplan-` topic. Only
`RunProgress` lines land in the task's own topic.

## What this is not

It is not a ceiling problem. Front's runs in the proof were 16.8–45.2 s
against 360; the superdirector's were 8.0–87.1 s against 1200. p6 concluded
the ceiling was never what bound; p7 confirms it from the other direction.

It is not the mention route failing. The route worked on its first live run,
including its startup recovery — which found a real stale mention from an
archived project and dropped it because no participation covered it.

It is a **turn-taking** problem and an **honesty-of-placement** problem, and
they compound: an agent told the wrong destination writes the wrong kind of
message, and the wrong kind of message is exactly what has no natural end.

## Already fixed, in passing

The first attempt found a third fault of the same family and it is closed.
A mention-triggered serving used to post its **ack** into the other agent's
topic. The owner is served by any post from somebody else, so "Message
received" bought a whole run — one that read a chatlog whose newest line was
an ack and answered that nothing had arrived, three times, while the real
reply was still being written.

pyagag `065de12`: a serving that answers in somebody else's topic posts once
and does not ack. An ack is how a bot's own sweep skips a topic it is already
serving; elsewhere it buys nothing and costs the owner a run. Verified live.

That fix removed one post per cycle. It did not remove the cycle, because the
cycle is Problem 1.

## Candidate answers

For Problem 2 the fix is small and is code, not prose: **the placement lines
say where this reply will be posted.** They already say where the chatlog is
and where to write files; the destination is the same kind of fact. Front
would then write a reply to autolab instead of a status report to the
Developer, and would stop sending the same message twice. This does not end
the cycle on its own.

For Problem 1, four shapes, in rough order of how much they ask of an agent:

- **Silent mention, flagged by the run.** The run writes a flag when it is
  asking something; `serve_topic` uses `@**name**` with it and `@_**name**`
  without. The judgement left to the agent is "am I asking?", which is far
  smaller than "should I keep waiting?" — and it is the shape forge's
  `question.flag` already had before p7 retired it.
- **Never hand back to whoever just handed you the turn**, unless asking.
  Fully mechanical, no agent judgement, but it caps any exchange at two turns
  and silences genuine follow-up questions.
- **An empty reply is a real option.** `TopicResult([])` already ends an
  exchange. What is missing is who decides — and "say nothing when you have
  nothing to add" was in the proof run's own mission text, which Front
  overrode three times. p6's finding applies: prose telling an agent not to
  use an affordance it has does not hold.
- **Change what a supervisor's reply *is*.** If Front always answered in its
  own `front-` topic and said everything outward with `agentchat`, it would
  never post into autolab's topic by reflex at all. That removes Front from
  the cycle without deciding the general rule — autolab and forge could still
  loop with each other.

## Why this belongs in the record

p6's finding was that an agent stops being present before the answer
arrives. p7 removed the waiting entirely and the agent is now always
present — and the new failure is its exact mirror: **an agent that is always
brought back never stops talking.** Both are the same missing thing, which is
that nothing in the system decides when an exchange is over. p6 left it to
the agent's judgement about waiting; p7 left it to the agent's judgement
about speaking. Neither held.

The pre-p5 state machine did not have this problem, for the same reason it
did not have p6's: it made the decision itself, and the agents had no say.
What both phases have bought is the knowledge of exactly which decision has
to come back — not the waiting, and not the routing, but **ending a
conversation.**
