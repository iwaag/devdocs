# agent_guide p3 — step 2: why Front skips the delegation

## Not new: the rewrite did not cause it

Step 1's arms, `delegate-answer`:

| arm | reached autolab |
|---|---|
| before p1 | 1/6 |
| p1 | 2/6 |
| now | 4/6 |

The pre-p1 guides skip *more* often, and give the same reasons. So the
behaviour is older than p1, and p2's composition (the shared `board.md` and
`callback.md`) is the best of the three. The plan's second branch applies:
"if both skip, it is older", and p1's "answer from the board" framing is a
suspect alongside as9 and fd-wr.

## What each composition told Front

Dry runs of `delegate-answer` in each arm (`pj-agdev/.local/agp3/dry/<arm>/prompt.md`,
ignored). Only the sentences that bear on "should I post to autolab?":

| text | before p1 | p1 | now |
|---|---|---|---|
| as9: "a 'how is it going?' restarts/starts their job … post only when you have something for them" | yes (in "Each run ends") | yes (`board.md`) | yes (pyagag `board.md`) |
| fd-wr: "a `✔` topic is finished — read the result there …; do not post a second start" | yes, twice, plus "`agentchat send` refuses a resolved topic" | yes | yes |
| "If the work is already done, say so." | yes | — | — |
| "If the last message is small talk or something plain text answers, just reply." | yes | — | — |
| "Most questions are about something that already exists … Answer them from the board, with where you found it." | — | yes (`requests.md`) | yes |
| "Ask autolab in its own channel about project reality" | — | yes | yes |
| "Ask with `agentchat send` where the agent's introduction says" | — (only "talk to the agent … with `agentchat`") | — | yes (pyagag `callback.md`) |

The table shows no sentence that forbids asking, in any arm. But none of
the three covers the case the probe makes. The Developer names an agent
and asks Front to put a question to it about finished work. The guides
sort a request into two kinds:

- **work**, which is delegated after a proposal;
- **a question about something that exists**, which is answered from the
  board (p1 onwards), or with a plain reply when the work "is already done,
  say so" (before p1).

"autolab に聞いて教えて" about a finished mission reads as the second kind.
The finished mission is on the board, so the board answers. That leaves
as9 and fd-wr, which make posting look costly or forbidden:

- **fd-wr**: one before-p1 run quotes it as its reason ("解決済みトピックへの
  再投稿は新しい作業を始めてしまうため"). The sentence is about a second
  *start* in *that* topic. It was read as "do not ask about finished work
  at all".
- **as9** is never quoted for finished work. ex1's first probe (running
  work, "今どう？"-shaped) was answered by citing it. In step 1 it shows as
  `delegate-decision`'s no-send runs: "m20402 is running; if autolab asks me
  to choose I will answer the cheaper one". Nothing was asked, although the
  Developer said "autolab に決めてもらって".

The "none" runs share one thing: the reply calls the record complete
("記録上は特に詰まった箇所はなく", "これが全経緯です"). The board holds two
posts for m20390, its start and its done line. What happened between them
(41 minutes of work, a paywalled source) was autolab's experience, and
nobody posted it. No guide says that a record's silence is not an answer.

## The wrong door is partly the fixture's

Five step-1 runs asked autolab in a topic of their own in `#pj-growbox`,
with `--to` and no mention. No listener is served by that. Four had read
`agentchat intro autolab-agstudio1`:

- **The fixture's introduction** (pyagag `agag/fixture/board.py`) names a
  single door: "**Ask** in the project's own channel, in a topic
  `workplan-<stem>`: say what you want done. I plan a mission there". For a
  question that is not work, a model reads "the project's own channel" and
  invents a topic name there.
- **The live introduction** (`#agents › intro-autolab-agstudio1`, read
  from the mirror) also says "**Questions about me go in
  `autolab-agstudio1`**, my own channel. Nothing starts there — but **ask
  about my work there** and I answer". The fixture dropped that sentence
  when it condensed the introduction.
- **p1's and now's `requests.md`** says "Ask autolab in its own channel
  about project reality". The wrong-door runs of those arms (p1 2, now 1)
  followed the introduction over it.

So the fixture is poorer than the realm in exactly the place this probe
tests. The measurement needs the fixture's introduction to carry the
question door. That is a fidelity fix, not a guide change, and step 3 makes
it first and re-measures `now` on it, so that the guide fix is not credited
with the board's correction.

## The suspects for step 3

1. **"Answer them from the board"** (agfront `shared/requests.md`) holds
   for what the board records. What an agent did but did not post is
   theirs to tell. A record that is silent on it is not "no stall".
2. **as9** (pyagag `guides/board.md`) is about asking after running work to
   see progress. A question the person you serve asks you to put to a
   named agent is theirs. Posting it is the request, not a poll.
3. **fd-wr** (pyagag `guides/board.md`) is about a second *start* in a
   resolved topic. A question about finished work is a new conversation at
   the agent's entrance, which its introduction names.

No Evidence-Driven paragraph is deleted: each is scoped where it stands.
