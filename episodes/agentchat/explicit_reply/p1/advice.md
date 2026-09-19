# Explicit reply — advice for p1

An agent's reply into the conversation it serves is, today, *whatever its
run printed last*. That contract has no boundary between the agent thinking
and the agent speaking, and the only thing that draws one is a sentence in
each guide ("no notes to yourself"). This advice records the problem as it
was observed, why a guide sentence is the wrong tool for it, and the
mechanism to give agents instead: a way to **mark their own reply**, with
everything outside the mark left to them.

## The observation

`#argue › argue-20260918-124701` (2026-09-18, the first human-led argue on
`argue` p2's code). Every one of Front's replies is two texts in one post —
a paragraph of Front telling itself what it is about to do, then the thing
itself. Message 7222 opens:

> The desire isn't on record yet. This is a sprawling, philosophically
> loaded question — before I draft anything or bring in archsage, I should
> reflect it back to the developer in a form they can confirm or correct,
> and probe the scope they actually want.
>
> Let me post a reply asking for that.
>
> Is the following a fair statement of what you want us to pursue? …

The second paragraph is a thought; the third is the reply. Both went to
Zulip as one message, so:

- The human and archsage read the thought as if it were addressed to them.
- The `present` role, whose whole job is faithfulness, re-voiced both. In
  the Arguing Room Front says the same thing twice per reply — first "I
  should ask them to confirm", then the asking. In the smoke argue that
  evening (#7262) the rendering is exactly two turns: the thought and the
  reply. This is what was noticed as "two interpretation utterances per
  reply"; the renderer and the memo hold exactly one rendering per post —
  the doubling is in the source.
- Rendering cost follows the doubled text (presentation was 19 % of that
  morning's session).

It is not a Front habit. archsage's #7230 opens "The sage is defined. Here
is my analysis for the argue." — the same shape from another agent, because
the cause is the same contract.

## Where the contract lives

- `pyagag` `agag.topics.serve_topic`: the handler returns `TopicResult`
  sections, the listener joins them and posts. The sections are the run's
  output: `agag.harness` returns the harness's final assistant message
  (`result` of the stream), and nothing between that and the post says which
  part of it was meant to be said.
- `agfront` `argue.py` removes the `ag-argue` machine block, appends the
  system's notes (desire recorded, outcome, block errors), posts the rest.
  `desk`, `front`, `routine_run`, autolab's roles and archsage all post the
  same way.
- The guides carry the rule in prose. `argue/guide.md`: "Write only the
  reply: no `[Name #id]` headers … no notes to yourself." A model that
  narrates before it answers — every current harness does, and the habit is
  useful to it — breaks the rule without noticing, and the record shows no
  one noticing either until a human read the room.

The second route an agent has to Zulip, `agentchat` on PATH with
`AGENTCHAT_HOME` naming the served conversation, is **not** the problem and
is not redundant with the first: it is how a run speaks in *another*
conversation and waits there (Front's workspace request into
`#pj-worldtrend › workplan-setup-worldtrend` in that same argue). The
output route is how a serving answers the conversation it was started for,
with the listener's guarantees — always one answer, the bot last so the
sweep stops, the system's notes appended, `resolve_after`, the hand-off
mention, the post-run re-check. Neither route can absorb the other; the
defect is that the output route has no mark.

## Why not a stricter guide

Adding "really, no preamble" to five guides would be the third time this
sentence is written and would fix nothing structural: the model still has
one channel for two kinds of text, and each future model, harness and mood
renegotiates where the line is. `devpolicy/styles.md` already says which
way to lean — tool giving over tool implantation. Thinking aloud before
answering is not a fault to suppress; it is the part of the output that was
never meant to be posted. The fix is to let the agent say which part was.

## The proposal: a reply mark

Give every run one mark, and make the listener post only what is marked.

```
The desire isn't on record. Before anything else I should reflect it
back and ask about scope.                       ← the agent's own, unposted

```ag-reply
Is the following a fair statement of what you want us to pursue?
…
```
```

- **Everything outside the mark is the agent's.** Notes, plans, the reason
  it chose a wording, a draft it discarded — none of it is posted, none of
  it is forbidden. It stays in the run record and the transcript, where it
  already is, which is also where a reviewer of a bad reply wants it.
- **Inside the mark is what is said**, verbatim, in the served
  conversation. Several marks in one output concatenate in order, so an
  agent can write its reply in pieces as it works rather than hold it for
  the end.
- **Machine blocks stay as they are** (`ag-argue`, and any block a handler
  parses). They are read from the whole output as today and never posted;
  their place relative to the mark does not matter.
- **No mark, no change**: an output with no mark is posted whole, exactly
  as now. The run record and the listener log say `reply_marked: false`,
  so the rooms and the Observer can show which agents have not taken the
  mark up, without a human having to read Zulip to find out. Silence is not
  an option — `serve_topic` must answer, or the ack leaves the bot as last
  poster and the topic disappears from the sweep.
- **The system's notes** (`— the desire is on record as message 7225.`)
  keep being appended after the reply by the handler, as they are.

### Where it lands

One place, so every agent gets it with a `pyagag` push:

- `agag.reply` (new, small): `split_reply(output) -> (reply, rest, marked)`,
  fence-aware in the CommonMark sense — an outer fence of four backticks
  closes at four, so a reply may itself contain three-backtick code. The
  `ag-argue` splitter in `agag.argue` keeps its own job; both run over the
  same output.
- `agag.topics.serve_topic`: the handler's sections pass through
  `split_reply` before the join; `marked` goes into the run record and the
  log line.
- Guides: the mark is *described* as a tool, once, in the shared skeleton
  text every role guide already inherits — what it is, that anything
  outside it is not posted — not added as a rule to each guide. The
  "no notes to yourself" sentences come out, since the mark makes them
  unnecessary.
- `present` receives the posted text and nothing else, so the room's Front
  speaks each reply once, and the double turn disappears without touching
  the renderer or its faithfulness rule.

### What it does not change

- `agentchat send` and the home/root-note handover: unchanged. The mark is
  for the served conversation; speaking elsewhere stays a deliberate act.
- The ack, the hand-off mention, `resolve_after`, the re-check for posts
  that arrived during the run: unchanged, because the reply still comes back
  through `TopicResult`.
- The `ag-argue` block and every other machine block: unchanged.

## Verify

- A run whose output is thought + marked reply posts only the reply; the
  thought is in the transcript and the run record.
- Two marks post as one message in order.
- No mark posts the whole output and records `reply_marked: false`.
- A reply containing a code fence survives intact.
- `present` renders one rendering with no "I will now reply" turn for a
  marked reply — re-render #7222's shape after the change and compare with
  memo record `j46335f85a54417f5f8ec`.
- The argue's system notes still follow the reply in the same post.

## Open question for p1

Whether the mark should also be what `agentchat` offers — an
`agentchat reply <text>` that posts into `AGENTCHAT_HOME` mid-run — so a
long serving can answer early and keep working. It is a natural extension
and the same principle (the agent marks what is said), but it moves the
reply out of `TopicResult` and so out of the listener's guarantees: the
notes, the resolve and the last-poster rule would each need a home of their
own. Not for p1; the output mark covers the observed problem with one
function and keeps every guarantee.
