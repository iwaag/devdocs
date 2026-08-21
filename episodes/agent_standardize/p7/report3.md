# p7 step 3 — the guides and intros, cut back

AI-generated (Omni Agent, 2026-08-21).

Every target met, and the waiting vocabulary is gone from all of them.

| file | before | target | after |
|---|---|---|---|
| `agfront/agent/guides/front/guide.md` | 265 | ≤ 110 | **110** |
| `agautolab/.../workrun_supercoder/guide.md` | 441 | ≤ 200 | **199** |
| `agautolab/.../workplan_superdirector/guide.md` | 369 | ≤ 280 | **270** |
| `agautolab/params/intro.md` | 744 | ≤ 300 | **299** |
| `agforge/params/intro.md` | 476 | ≤ 280 | **280** |

```
$ grep -rn -i "wait|background|budget|blocking" <all guides and both intros>
none
```

Commits: agfront `35bb43a`, agautolab `ffa8ef2`, agforge `2ac0b5c`. Pushed.
Both introductions re-posted to `#agents` — `intro-autolab-agstudio1`
message **802**, `intro-agforge-agstudio1` message **803**.

## front guide, 265 → 110

The last three paragraphs deleted entirely, as the plan asks: "stay with it",
"wait inside this run", "if you run out of time". Nothing replaces them.
`undestand` and `fullfilled` fixed.

**One deviation from "keep the first nine lines as they are".** The file
begins with a blank line, so line nine is
`If the developer permit you to proceed your plan…` and line ten is
`Use command "agentchat --help" to learn how to communicate with other
agents.` Cutting at nine would have taken away Front's only pointer to the
tool it is being told to use — an Unexplained Chainsaw in one edit. The two
lines are merged into one instead, which keeps the pointer and lands exactly
on 110.

## supercoder guide, 441 → 199

The delegation section is now:

> The introductions file this prompt names above says how to reach each agent
> and what it calls finished. Talk with them using `agentchat` (`--help`
> explains it).
>
> Post the request and finish. You will be called again when they answer, and
> the result goes into this task's own topic.

Deleted: "delegating is a supervision, not a message you send and forget",
all five bullets (including the blocking `agentchat wait`), "you are the one
who decides", "bring it back into this task's own topic", and the
`read --since` recovery paragraph.

One extra deletion beyond the plan: the paragraph *following* the section —
"Then report as any other task does. The developer is the one who says the
task is complete; delivering the result does not close it." The final line of
the guide already carries that rule ("If the developer agreed that the task
was done, create `report.md`"), and the duplicate was what stood between 224
and the 200 target.

## superdirector guide, 369 → 270

"Work other agents can do" is three lines and one more: which agent, what to
ask in words they can act on without this project, one request per task. The
"twice as long a window" paragraph went with the waiting it was reasoning
about — a task that delegates is no longer a task that sits idle.

## autolab intro, 744 → 299

Kept, in the plan's own list: work goes in `pj-<slug>`, `workplan-` plans
only, an explicit go starts it, I open `work-<label>`/`workrun-task<N>-`
myself, one topic is one task, posting into one starts it, tasks run in
order, I mention you when it is your turn, I close a task only when you say
it is complete, and "yes, commit" is not that.

Cut: every rationale sentence ("there is no other place that says it", "a
message into an empty room", "that is a queue, not a failure"), the whole
"While a task runs, somebody has to be there" framing including "expect to
wait rather than to be answered at once", the "Choosing the project" section
(folded into two clauses), and the entire delegation paragraph — internal
detail a requester never sees.

The heading changed from "While a task runs, somebody has to be there" to
"While a task runs". Nobody has to be there any more; they have to answer
when named, which the section says.

## forge intro, 476 → 280

Kept: where to write, plan-only, "I mention you when it is your turn", Work
registered ≠ generated, the `assetrun-` trigger, one-trigger-one-Work,
delivery into the `assetplan-` topic with the URL and `[S3KEY]`, the resign
endpoint, retry by re-triggering.

Cut: "a button, not a conversation", the two-triggers illustration, "Expect a
few minutes per exchange", "Generating takes minutes", and "do say in your own
topic that you got it" — what a requester tells their own side is not forge's
business to ask for.

**"wait for the delivery before you fire the next one" is now "let the
delivery land before the next one."** The plan asked to keep that rule and
also asked that nothing mention waiting; this wording keeps the sequencing
constraint without the word, which is the point — the requester is not being
asked to sit and watch, only not to fire twice.

## What the cut is really about

Every deleted line was true when it was written. What changed is that the
thing it described stopped being the agent's job. p6's finding was that
telling an agent not to background a wait does not work; p7's answer is that
there is no wait, so there is nothing to be told. The guides got shorter
because the system got the responsibility, not because the advice was bad.
