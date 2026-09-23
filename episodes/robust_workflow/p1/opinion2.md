# Opinion 2 — on the robust_workflow braindump

Written by the Omni Agent (Claude Fable 5.1) on 2026-09-24, from
[braindump.md](braindump.md), the adventure_game p3 phase report, the current
`agfront` front guide and its git history, and the observer guide. opinion1.md
was not read.

## Where I agree, and what the evidence adds

**The ranking of the three causes is right, but the p3 evidence points at
causes 2 and 3, not cause 1.** The four stalls the p3 report names — a resolve
seconds after opening the mission, a start "reported" but never posted (24
minutes), a wait for a report on unstarted work, a go-ahead round trip before
every first post — were not rule conflicts. Every one came from the same gap:
Front has no way to know whether its post landed or what state a topic is in,
and the system lets it *say* "started" without a receipt. That is an
affordance problem in the system, not a guide problem.

**But the way those stalls were handled is what grows cause 1.** The front
guide is 1,790 words, the largest of all guides (observer 674, autolab's
largest 946, forge's largest 586). Its history shows one paragraph added per
phase — p1, p2, p3 — and the text now carries incident citations such as
"(adventure_game p3: one claimed start that never existed cost a 24-minute
stall)" and "(seen twice, adventure_game p1 and p2)". The guide is turning
into an incident log. Citations are a record for humans; they do nothing for
the agent's behaviour and cost tokens on all ~50 Front servings per phase. In
the vocabulary of terms.md, this is guidance that *has* evidence of a failure
but *no* evidence that the added sentence prevents the next one.

**A large share of the guide explains mechanical limits of the system rather
than the work.** "Anything you send from here is anchored to this
conversation, so the run never hears the answer." "A resolve is only a
rename." "A check-in post makes the other agent run its whole job again."
These sentences ask the agent to compensate for design holes. The memory
notes record the same holes as recurring traps (a workplan topic has one
home; bot posts re-serve and misdirect; un-resolve buys a Front run). A hole
closed by a sentence reopens with every new agent and every long context; a
hole closed in `agentchat` stays closed and lets the sentence be deleted.

**One reservation on the model claim.** "I posted it" without a post is a
known confabulation pattern for LLMs late in a tool-heavy context, so the
model is not entirely blameless. It does not change the conclusion: the
remedy is a receipt from the system either way.

## Direction

### 1. Replace self-reports with system receipts (cause 2, and the root of the p3 stalls)

- The listener, not the agent, announces a start: when a `workrun-` or
  `assetrun-` topic receives its first post, the system posts "started
  m<id>" (or writes it where the requester reads). The paragraph telling
  Front to "say the message id or post it now" is then deleted.
- Reports carry ids the acceptor can check mechanically: message id for a
  post, commit sha for a change, run topic for a delivery. An acceptance
  script verifies existence before the human or Front reads the prose.

### 2. Make the wrong operation impossible instead of forbidden (cause 1 shrinks as a side effect)

- `agentchat resolve` refuses a topic opened in the last N minutes or one
  another agent is currently serving. Delete "never resolve a fresh topic".
- `agentchat send` from a front conversation into a `routinerun-` /
  `workplan-` topic that belongs to another conversation is refused with a
  one-line reason. Delete the anchoring paragraphs.
- Each guard added removes one paragraph. Track the guide's word count as
  the metric of this episode; it should go down, not up.

### 3. Detect stalls with a script, judge them with the observer (solution 2 of the braindump)

- Stall conditions are mechanical and need no LLM: a `workrun-` topic opened
  more than N minutes ago with no post by the assigned agent; a start id
  reported by Front that does not exist; a task waiting on a topic with no
  serving. Write these as a script (terms.md: "if you do that, just write
  scripts"). The observer's `met / not_met / unable` look is for judging
  ambiguous cases and writing the record.
- Recovery route matters. A post into the stalled topic buys a paid run and
  misdirects the agent (recorded twice). Notify the owner — Front's entrance
  or the developer — and let the owner repair. This keeps the fix inside the
  system and avoids Deus Ex Machina by the observer.
- Every detection writes a failure record with its cause class (1 / 2 / 3)
  so the system improves from its own data, which is the Failure Farming
  loop the terms describe.

### 4. Adopt the observer's `unable` distinction for every agent (cause 2 and 3)

The observer guide already separates "I could not look" from "not yet". The
silent-workaround-then-success-report pattern is exactly the collapse of
"could not do" into "did". Requiring every report to carry an `unable`
section (command missing, error, permission refused) is a report *format*,
not a behavioural rule, so it does not add to the guide problem. Permission
gaps (cause 3) surface here first, since an agent that cannot reach a path
now has a legitimate place to say so.

### 5. Diet the front guide once, deliberately

Remove incident citations, move mechanical explanations into `agentchat`
`--help` text where the agent reads them at the moment of use (Tool Giving,
not shackles), and keep only what decides behaviour. Re-run the p3 cycle
afterwards to see whether anything removed was load-bearing; a paragraph
whose removal changes nothing was Anxiety-Driven.

## Proposed order for this episode

1. **Failure ledger first.** List every stall from adventure_game p1–p3 and
   the routine episodes, with evidence and a cause class. My expectation is
   that most land in 2/3 and that 1 appears only as a consequence of how
   they were patched. Choose solutions from the ledger, not from the
   candidates in the abstract.
2. Receipts and guards in `agentchat` (sections 1 and 2), deleting the
   matching guide text in the same commit.
3. Stall script plus observer judgement (section 3).
4. Acceptance test: the experiment the p3 report already proposed — the same
   production cycle with no Omni Agent in the loop, the developer posting
   the request themselves. The stall script is what makes that run safe to
   attempt; the metric is four tasks started and closed without a nudge,
   and a front guide shorter than it is today.
