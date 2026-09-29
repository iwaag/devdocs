# agent_guide p3 ex2 — step 1: the responder answers only where live autolab would

pyagag `8d372f2`.

## The change

A scripted agent's lines are now reached only through its **doors**
(`agag.fixture.responder.Doors`). A door is where that agent's live
listener serves a post in the way the script means. The script's lines no
longer carry their own topic prefixes. The doors are a per-agent rule
(`responder.DOORS`), because they are the agent's contract, not the
probe's.

autolab's doors (`AUTOLAB_DOORS`) follow the introduction the fixture
posts for it (`board.INTROS`) and the live listener
(`agautolab.listener`, `handle_mention`):

| post | live autolab | fixture before | fixture now |
|---|---|---|---|
| any topic of `autolab-agstudio1` | the entrance answers about its work | next line | next line |
| a mission's own `workplan-`/`workrun-` topic (on the board before the trial) | that mission's planner or task is served | next line | next line |
| a **new** `workplan-` topic in a `pj-` channel | a new mission is planned | next line | a plan acknowledgement for a new mission, and no script line |
| a post in that new topic afterwards | the new mission's planning | next line | nothing scripted |
| a `workplan-` topic outside a `pj-` channel | not acted on | next line | nothing |
| a new `workrun-` topic | "not bound to any task" | next line | nothing |
| `@**autolab-agstudio1**` in any other topic | ignored, unless it is a callback to its own task | next line | nothing |
| `to=` without a mention | reaches nobody | nothing | nothing |

The plan acknowledgement is autolab's ACK, then a line to the asker: it read
the post as a new request, planned it as a new mission from what the
topic asks, and nothing runs until the asker says start there. It ends
with `ag-post intent=response_request … ask=decision`. Because it names
the asker, the trial kit serves the conversation again, as a live callback
would. The run can then see that it opened a new mission and ask again at
the right door. Live autolab gives it about the same chance.

An agent with no rule in `DOORS` is served by a mention or in its own
channel, as before.

The mention door closed too. The plan named the three doors to keep, and a
mention outside them is not one of them: live `handle_mention` drops any
mention that carries no root note of autolab's own task. No ex1 run went
through that door (below), so it changes no ex1 count.

## What a run now records

- Every send on an overlay is logged with the door it took: `answer`
  (which script line), `new`, or `none`. The trial kit writes that log to
  `outcome.json` as `doors`.
- `python -m agag.fixture doors <out-dir>…` puts saved runs' sends through
  today's doors. For each send it gives the place, whether a scripted
  agent answered it then, and the door it would take now. It lists every
  run where the two differ.

## Tests

In `tests/test_fixture_responder.py`:

- **ex1's four wrong-door runs** (`WRONG_DOOR_RUNS`, one case each: ctl
  `delegate-decision` 2, 5, 12 and fix 11). Each sends its run's topic and
  question through `agentchat send --to autolab-agstudio1`, on an overlay
  of `delegate-decision`'s script.
  - The question gets the plan acknowledgement, not the scripted "16 h or
    12 h?".
  - The choice then sent into the same topic ("12時間/日でお願いします")
    gets nothing.
  - The same question in `work-m20510 › workrun-task1-m20510` gets the
    decision question.
- A `workplan-` topic in `#front` is not acted on.
- A mention in a `pj-` topic gets nothing. autolab's channel and the
  mission's plan topic do get a line. The door log reads `none, none,
  answer, answer, none`.
- The replay marks a saved run answered in a new `workplan-` topic as
  changed, and one answered in the task's own topic as unchanged.

The whole pyagag suite passes (1208 tests before the replay test was added;
the responder file is 12/12 after).

## ex1's counts under the new door

ex1's counts stay as they are. The door changes what a run sees next, so
re-judging cannot say what a changed run would have done. Here is only
which runs the new door would have treated differently (`python -m
agag.fixture doors pj-agdev/.local/agp3ex1/s3`):

| probe | fix | ctl |
|---|---|---|
| `delegate-answer` | 0 of 12 | 0 of 12 |
| `delegate-decision` | 1 of 12 (11) | 3 of 12 (2, 5, 12) |

The four are exactly ex1's open finding 2. Each asked about m20510 in a new
`workplan-` topic (`workplan-growbox-lighting-hours`, or
`…-control-loop-lighting-hours` in ctl 2) after the plan topic refused a
second anchor. Each would now get a new-mission acknowledgement where it
got the decision question, and nothing where it got the "Set: 12 h" line.
The other 44 took the same doors:

- every `delegate-answer` asked in `autolab-agstudio1`;
- the other `delegate-decision` runs asked in the running task's topic,
  or sent nothing (fix 4).

The same replay over p3's board-1 runs (`agp3/s1`, `agp3/s3`) is
background for step 3:

| record | arm | `delegate-answer` | `delegate-decision` |
|---|---|---|---|
| p3 s1 | before p1 | 0/6 | 0/6 |
| | p1 | 0/6 | 3/6 |
| | now | 2/6 | 2/6 |
| p3 s3 | ctl | 0/6 | 3/6 |
| | fix | 0/6 | 6/6 |

Board 1 has a `work-m20402 › workrun-task1-m20402` topic, holding one
progress post (#20073). ex1's report3 said board 1 had no task topic for
m20402; that is not right. What board 1 lacked was the task's start
records. Every before-p1
delegate-decision run that sent anything (5 of 6) asked there, which is
the right door. Most later "reached" runs asked in a new `workplan-` topic
instead:

- p1: 3 of 5 (runs 1, 4 and 6);
- now: 1 of 5 (run 3);
- s3 ctl: 3 of 6;
- s3 fix: 6 of 6.

One more run (s1 now 6) asked in a `workrun-task1-m20402` topic it made up
in `#pj-growbox`. Live autolab would answer that with "not bound to any
task", and the new doors give it nothing.

So p3's three-arm delegate-decision row partly measured a door that live
autolab does not keep. That row was 5/6, 5/6 and 4/6, with now's run 5 a
pass in substance. Counting only runs that went through a live door, it
reads:

| arm | runs that reached autolab |
|---|---|
| before p1 | 5 of 6 |
| p1 | 2 of 6 |
| now | 3 of 6 |

A run might have recovered after the acknowledgement, so the p1 and now
figures are lower than they could have been. This is the reverse of what
p3's row suggested. Step 3 measures it again with the doors.

## Deus Ex Machina notes

None.
