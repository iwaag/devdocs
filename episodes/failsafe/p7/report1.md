# failsafe p7 — step 1: the claims and where their evidence is

This is the checker's contract: which acts a reply can say it did, which
record proves each one, and how a checker finds the records one serving
wrote.

## The serving's window

A listener serves one conversation at a time (`agag.listen`, one executor
thread). While a serving runs, its run's tools post with the agent's own
credential (`AGENTCHAT_ZULIP_ENV` is the listener's). So **every post the
agent's bot made between the serving's start and its delivered reply is
that serving's**, whatever conversation the post went to and whatever
harness ran. Zulip ids grow across the whole realm, so the window is an id
range:

- **from**: the newest message the listener's mirror holds when the serving
  opens. On the owner route this is at or just before the ack
  (`Serving.ack_id`). On the mention route there is no ack, so the window
  cannot start from one.
- **to**: `Serving.delivered_id`, the reply. The listener's own writes after
  delivery (the served marks, `[selfnote][continuation]`) have higher ids
  and fall outside it.

The records are read from the listener's own mirror
(`Mirror.notes(tag=…, sender_id=…)`, `MirrorStore.messages_by_sender`),
with no Zulip call. The mirror lags the realm by the event queue's
latency, so the check waits until the mirror holds `delivered_id` before
it concludes anything. A mirror that does not reach it concludes nothing.

Two exclusions:

- **Memo channels.** Front's render worker posts `ag-memo` records with
  Front's own bot, on its own thread, while a desk serving runs. In the
  incident, #15840/#15841 (`memo`) fell between #15838 and #15842. A memo
  is never a record or a send (`agag.memo`).
- **The listener's own lines.** The ack and the reply bound the window and
  are not acts.

The incident, read off Front's mirror:

| serving | window | Front's posts inside it | the reply claims |
|---|---|---|---|
| run-0182 | ack #15836 → reply #15838 | #15837 `[hold] decision a15835 …` | a hold #15837 — **present** |
| run-0183 | ack #15843 → reply #15844 | none | a release of #15837 and a `withdrawn` disposition — **both missing** |
| run-0184 | ack #15848 → reply #15851 | #15849 `[hold-release] #15837 …`, #15850 `[disposition] withdrawn a15835 …` | both — **present** (and it cites #15844's false claim, which is not a claim of its own) |

The run record (`agfront/.local/agent/desk/run-0183.json`) agrees:
`num_turns: 1`, `reply.delivered_id: 15844`,
`reply.intent.end: 15843`, `reply.serving_id: 728`. It holds no tool
calls. Neither the record nor the window needs them.

## The acts and their records

Each act is a record kind. A claim may name a **target**: the id the reply
gives, such as the hold's id, the unit's anchor or the answer's id.

| act | what the reply says | record (always a post by the agent's own bot) | where it lands | target it can name |
|---|---|---|---|---|
| `hold` | "held", "put on hold" | `[selfnote][hold] <purpose> a<unit> by <user> … #<evidence>` (`agag.holds.hold_note`) | the request's origin conversation (`holds._origin_of`) | the unit (`a<unit>`); the hold's own id |
| `release` | "released the hold" | `[selfnote][hold-release] #<hold> by <user> … #<evidence>` | the origin | the hold id |
| `disposition` | "recorded as withdrawn/completed/cancelled/suppressed", "reversed the disposition" | `[selfnote][disposition] <kind> a<unit> upto=#… by …`, or `[selfnote][disposition-reversed] #<id> …` (`agag.dispositions`) | the origin | the unit; the kind; the reversed id |
| `relation` | "recorded the relation as work/reference" | `[selfnote][relation] #<note> work\|reference` (`agag.relations.correction_note`), or a first `[selfnote][rootchat] … rel=` written by `agentchat send --relation` | the conversation holding the root note | the root note's id |
| `accept` | "accepted the mission", "recorded the acceptance" | `[selfnote][acceptance] #<evidence> by <user> … after=#<shown>`, with `[selfnote][state] accepted` in each task and `[selfnote][state] done` (`agag.acceptance.accept_mission`) | the mission's plan conversation and its task conversations | the mission id |
| `reserve` | "reserved the approval for …" | `[selfnote][approval] reserved <user> … #<post>` (`acceptance.reservation_note`, `agentchat reserve`) | the mission's conversation | the mission id |
| `receipt` | "repaired the receipt", "marked the answer received" | `[selfnote][receipt] <remote> #<answer> …` (`selfnote.receipt_note`), or a `[selfnote][served] <remote> <answer>` written by the **run** through `agentchat receipt --repair` | the caller's home | the answer id |
| `send` | "asked autolab in …", "posted to …", "sent the request to …" | a non-selfnote post by the agent's bot in a conversation other than the reply's own, usually after a `[selfnote][rootchat] … rel=…` there (`agentchat send`) | the other agent's conversation | the channel/topic, or the agent asked |

Notes on the table:

- **`send` is the same failure class and costs more.** A reply that says it
  asked autolab, when it asked nobody, leaves the person waiting for an
  answer that nobody owes. Front's guide asks it to report where it posted
  (`agfront/agent/guides/shared/work.md`, `callback.md`). A send's evidence
  is the post itself, found by sender and window. No selfnote is needed:
  the root note may be absent (a second send into a topic already anchored
  writes none).
- **The listener's served marks are not the `receipt` act.** They are
  written after delivery, outside the window. A `[served]` note inside the
  window was written by the run (`agentchat receipt --repair`).
- **The proxy case (p6 ex2).** A release on the Omni Agent's words is
  written by Front's bot as `[hold-release] #<hold> by 9 (Omni Agent) …`
  (#15849). The checker looks at **who posted the note** (the agent that
  claims to have recorded it) and its kind and target. The `by` field
  says whose decision it was, and the checker does not compare it: the
  tools already refuse a record on the wrong person's words
  (`holds.release`: `acts_for`).
- **A claim about the state of the world is not an act.** "Hold #15837 is
  still in force" or "the command is `agentchat release …`" claims nothing
  done in this serving. Whoever reads the reply must tell the two apart.
  The comparison below also tolerates a restatement of an act done earlier.

## How a checker decides a claim is true

For each claimed act, in order:

1. **In the window.** A record of that kind (and target, when the claim
   names one) posted by the agent's bot inside the serving's window. This
   is the normal case: the serving did what it says.
2. **Already on record.** When the claim names a target: a record of that
   kind naming that target anywhere in the realm, by anyone, before the
   reply. When it names none: a record of that kind in the conversation the
   reply went to (for a hold or a disposition that is the origin). This is
   a reply that restates something done earlier ("the disposition is
   recorded as withdrawn", said again a serving later). The world is as
   the reply says, so nothing needs repair.
3. Otherwise the claim is **missing**: the reply says an act happened and
   no record shows it.

For `send` the record is a post, not a note. A target is a channel (and
topic, when named) or the agent asked, matched against the conversation
the post went to and that conversation's owner. The same two places are
looked at: the window, then (for a restatement) the agent's own earlier
posts in that conversation.

This comparison needs only posts: the mirror's note index and sender
index. It reads no transcript, so it is the same for claude_code, codex,
agy, gemini and agcode runs. How a reply's claims become a list of
`(act, target)` is step 2.

## What this checks and what it does not

- It checks that the **record exists**, not that the act was allowed. The
  tools refuse a disallowed act before writing: a hold on the agent's own
  words, a release by somebody without authority, an acceptance without
  the holder's evidence. A record that exists was allowed when it was
  written.
- It does not check acts that leave no record in the realm: a file written,
  a commit, a command run. Those have their own records (autolab's
  `[change]` checkpoints) and are outside this phase.
- Other recorded acts can be added as rows later: `agrun finish` (the
  `ag-routinerun` block), `agproject open`, `agroutine update`, `archsage
  sage sync` (`[sagesync]`), autolab's `correct_state`. Each writes a note
  or a post by the agent's bot, so each fits the same window and index.
