# failsafe p7 — step 3: what happens on a mismatch

## The owner repairs, once, in the same conversation

The act belongs to the agent that claimed it, and the conversation it
claimed it in is where the person reads the claim. The repair is what the
Omni Agent's #15847 did by hand: serve that agent again there with the
mismatch stated, so it records the act or corrects its reply.

One mismatch writes two notes into the conversation the reply was posted
in (its home), both by the agent's own bot:

1. **The record**, `[selfnote][claim] {json}`:

   ```json
   {"reply": 15844, "serving": 728, "attempt": 1, "window": [15843, 15844],
    "requester": 9, "requester_name": "Omni Agent",
    "missing": [{"act": "release", "target": 15837, "quote": "Released hold #15837 …"},
                {"act": "disposition", "target": 15835, "quote": "Recorded the disposition … withdrawn"}],
    "found": []}
   ```

   It is a selfnote: hidden from chatlogs, and it buys nobody a run.
   `found` lists the claims of the same reply that were checked and
   matched, so the record shows the whole check.

2. **The trigger**, `[selfnote][start] #<claim note> for <requester id>
   <name>` (`agag.selfnote.start_note`). This is the one note of an owner's
   that makes its own listener serve a conversation. autolab starts tasks
   with it and `agrun continue` resumes runs with it. Its `because` is the
   claim note.

What follows:

- **The notice is the trigger.** The listener's intake enqueues the start
  like any other, so the repair serving's `trigger_id` is the start note.
  `because` points at the claim, never at the stale decision (#15842).
- **The run is told.** `prompt_with_guide(reply=True)` already appends a
  notice read from the bound serving: the retry notice for an owed reply.
  The claim notice is added the same way, for every conversational role of
  every agent, with no guide text. It holds every open claim in the
  serving's home: what the reply said (the quotes), which record each act
  needs, that no such record exists, and the two ways out: run the tool
  now, or say plainly that it was not done. It also says that the tools
  refuse a record without the person's words, and that correcting the
  reply is a sufficient answer.
- **The answer goes to the person who was misled.** The start names the
  original serving's requester, so the repair reply is handed to them: the
  Omni Agent in the incident, as #15851 was.
- **A person's post meanwhile is not lost.** The owner entry coalesces. The
  next serving of the conversation gets the person's post *and* the notice,
  because the notice is read from the open claims in home, not only from
  the trigger.

## One mismatch buys one serving

After every delivered serving in a home that holds an open claim, the
listener settles the claims that serving was the answer to (claims written
before the serving started):

| what the repair serving left | written | the claim is |
|---|---|---|
| every missing act now has its record (step 1's comparison, rerun) | `[selfnote][claim-settled] #<claim> recorded #<reply>` | closed: the act came about |
| not all of them, and its own reply does not claim them again | `[selfnote][claim-settled] #<claim> corrected #<reply>` | closed: the reply that answered the notice is the correction, on the record for the person to read |
| not all of them, and its own reply claims one of them again | `[selfnote][claim] {… "attempt": 2 …}`, **no start note** | escalated: the listener stops asking |

The repair serving's own reply goes through the ordinary check too. A new
false claim of a different act starts its own attempt 1. A repeat of the
same act is attempt 2. A second mismatch never buys a third serving.

## Escalation goes to Observer's reporting path

The listener has no way to reach a person without buying somebody a run:
naming the requester in a post serves them when they are an agent. So an
escalated claim is a record, and the realm's existing reporter reads it:

- `agag.trace` reads the claim notes into the node of the conversation
  holding them (`Node.claims`: open claims, their attempt, what is missing).
- `stall_candidates` produces a **`claim`** candidate for a node with an
  escalated claim (at once, 60 s like `unanswered`), and also for an
  attempt-1 claim that nothing settled within 600 s (the repair serving
  never ran: a listener down, a queue stuck).
- Observer's monitor reports a `claim` incident to the owners and does not
  ask the agent again. Asking again would buy another run of the same
  failure: the `unanswered` rule. Its review topic groups it by owner and
  kind like every other incident.

Observer traces requests that came through `#front`. A claim in a
conversation outside every such request is still recorded, shown by
`agentchat trace` and counted in the listener's status file, but nobody is
told. That is a limit of this phase.

## Shown as open until settled

- `agentchat trace` prints an open claim under its conversation, as
  "owed now" prints an owed receipt:
  `claim #<note>: the reply #<reply> says release #15837, disposition —
  no record (attempt 1, repairing)`.
- The progress panel (`agag.progress`) carries the node's open claims on
  the unit (`claims`), and the card's owed stages list them. A card is not
  `completed` while a claim on it is open.
- A settled claim is history: the trace lists it under the node's
  records, not as owed.

## What this deliberately does not do

- **It does not write the missing record for the agent.** A record says who
  decided and on which post. The listener cannot know that a release was
  wanted; it only knows a reply said so. Writing it would be the same
  mistake one layer down.
- **It does not unsay the reply.** The false reply stays as posted; the
  correction or the record follows it, as in the incident.
- **It does not check every serving the same way.** A serving whose reply
  is the listener's own failure line, or that delivered nothing, has no
  claim of the agent's to check (step 2).
