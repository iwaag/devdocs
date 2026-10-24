# Workflow editor fix6 — pre-research: why fix5's reminders did not produce presentations

Written by the Omni Agent on 2026-10-24 from an investigation the Developer
asked for, and the discussion that followed. It is input for fix6's
braindump and plan, not a plan.

## The question

fix5 ([report](../fix5/report.md)) made WFE warn before an operation when
completed work has no presentation (`presentation-missing`, among others).
In the next real conversation the agents still made no presentation until
the Developer asked for one. The Developer found it hard to believe that
current agents would ignore such a warning, and asked first to rule out a
simple cause, such as the information not reaching the agent.

## The case

Conversation `20261024-160217-forusr` (aghangout, facilitated by
ponderer): the Developer asked whether the method of Narita & Kanai
(PG 2026) can be tried locally and, if so, for a new project `pj-misc`
(with `study-misc` and `wedo-misc`) to try it. ponderer/conduct delegated
each step to misc/freework:

| Child run | Requested by | Produced | Presentation |
| --- | --- | --- | --- |
| misc/freework/run-001 | ponderer/conduct/run-031 | 2D C++ prototype, numbers, README | none |
| run-002 | run-032 | paper's rules, 2D/3D, formula notes | none |
| run-003 | run-034 | Godot partition viewer, 6 screenshots, spin video | none |
| run-004 | run-035 | 2D FLIP dam-break comparison video (mp4) | none |
| run-005 | run-036 | review page of the above | p1, reviewed, stopped by cleanup |

run-005 exists only because the Developer wrote (#361) "Miscの方で
Presentationノードでわかりやすく提示させてください". The child runs are on
WFE `506a116` and the wfrun guide `8f341ed`, both fix5's final releases.

Evidence: the executor's attempt logs of these runs (stream-json), read in
full, and the run and delegation records (`wfe run show`, `wfe delegate
show`). Attempt ids: run-001 `amvm26eo0-69b0f1`, run-002
`amvm2fkxx-cce507`, run-003 `amvm2zwub-46850e`, run-004
`amvm3fa4y-e54a1a`; ponderer run-031 `amvm2azzy-7540cf`, run-033
`amvm2uk0v-184d42`, run-035 `amvm3vgq3-589964`. Thinking text is redacted
in these logs (signatures and durations only), so reasons below come from
commands, tool output and the agents' final messages.

## Findings

### The information reached the agent

- Every attempt's prompt is the wfrun guide (fix5 revision, with the
  *Reminders* paragraph) followed by the brief (agautolab `wfexec.py`
  `prompt()`). The brief lists the run's node catalog with each
  template's expectation (`shared/catalog.ts` `catalogBrief`). The present
  template says: "When a run delivers something I can look at or try, I
  expect a node that presents it to me for review, before the step where I
  confirm the result, and a cleanup node later in the same run that stops
  it."
- `presentation-missing` was actually returned in run-001, run-003 and
  run-004. Each time the agent repeated the command with `--yes`.

### Simple causes found

1. **run-002 got no warning at all.** The agent added its work nodes by
   hand (`type: do`, `read-paper`, `work-2d`, `work-3d`) instead of
   instantiating the `work` template. fix5 asks only nodes of the `work`
   template to be presented (a limitation its report names), so nothing
   fired.
2. **run-003 and run-004 saw only the tail of the warning.** The agent
   called `wfe run complete … 2>&1 | tail -3`. The reason line was cut
   off: `presentation-missing: … the person's catalog expects a node that
   presents what they can look at or try, before they confirm it`. What
   remained was:
   - the instantiate/adjust repair command;
   - `if the work only produced records that need no viewing, continue with --yes`;
   - `Nothing was recorded … give the same command with --yes …`.

   Read alone, this is directions to `--yes`. The turn before `--yes`
   thought for about 0.5 s. In run-003 the command also chained the
   completion of `_final-report` after the warned completion.
3. **run-001 read the whole warning** (`tail -5`) and chose `--yes`. Its
   final message says it added no presentation node because the result
   was a text report and a table, which matches the warning's own escape:
   "records that need no viewing". This one is a defensible judgement.
4. **The child's requester is an agent, not the person.**
   - Each brief says the run was requested by ponderer-agent, and that
     questions go `--to ponderer-agent`.
   - The requests end with "Return what was built, how to run it, and where
     the files are" (run-003) or "Return: code …, a short result summary"
     (run-001). The completion condition is a report to ponderer.
   - The catalog's expectation speaks of presenting "to me", and from the
     child that "me" is not in the room.
5. **The agent that talks to the person was never warned about
   presentations.**
   - ponderer's conduct runs have no work node of their own; they delegate.
     Their result posts (`wfe conversation reply --kind result`) therefore
     got no `presentation-missing`.
   - ponderer judged each delegation `satisfied` within seconds of take-up.
     run-035 d1 is an example:
     - the delegation's result acceptance is "the person's (relay their
       answer)";
     - the result manifest says "execution completed, no decision";
     - an 8.4 MB `dambreak_comparison.mp4` is among the artifacts;
     - ponderer judged it satisfied 26 s after the take-up.
   - The child's result had also already been integrated into misc's shared
     branches before anyone saw it.

### Agents do act on reminders whose repair is clear

ponderer got fix5's `discussion-activity` reminder in run-031, run-033 and
run-035 ("completed this way, the discussion gets no record of it"). Each
time it ran the named repair (`wfe conversation ponder --node … --decision
… --source …`) and posted again, without `--yes`. The contrast with
`presentation-missing` suggests that the outcome depends on the warning's
wording and on whether the agent sees the case as its own. It is not a
general habit of skipping warnings.

## Discussion and proposals

### Directions noted after the findings

- **A. Warning text and order.**
  - Put the decisive line where a truncated read still sees it, for example
    a one-line summary at the end as well.
  - Drop the "records that need no viewing" escape.
  - State the facts WFE knows: "the artifacts include a video
    (`….mp4`) and screenshots".
- **B. Coverage.** Treat untemplated `do` nodes as work too, so a run that
  writes nodes by hand is not silently exempt (run-002).
- **C. Delegation.** Make the parent that relays a child's result to the
  person check what the child left out, or say in the request that the
  presentation is the child's job.

The Omni Agent recommended C and A: most of the failure is explained by
the child not seeing whom it presents to, and by the truncated warning.

### The Developer's question

The Developer asked whether ponderer or Front, when they take a child run's
report and hand it to the person, could be given a checklist of the child
process before accepting it. At delegation and at report time, the checks
would be shown, and a clear violation would prompt a correction.

### Answer: feasible, and the receiving point already exists

- **The receiving point already exists.** A delegate node cannot complete
  until the parent's agent records `delegation.dispose satisfied`
  (`delegation-unsatisfied`; [delegation.md](../../../../pj-agdev/agdevworld/workflow_editor/docs/delegation.md)).
  It is a step every delegation passes.
- **The facts are already in the parent's hands.** The `completed` event
  carries a result manifest: decision state, artifacts by path, size and
  hash, and the report. The parent can read the child's record and files
  (`wfe delegate show|file`). run-035 d1 shows that "a video the person
  should see, no presentation, no confirmation, while the person accepts"
  is visible mechanically.

Proposed shape:

1. **Warnings at disposition (the core).**
   - Evaluate fix5's reminders (`shared/reminders.ts`) over the child's
     record, and return them on the parent's `wfe delegate dispose …
     --disposition satisfied`.
   - Cover:
     - viewable artifacts with no presentation;
     - a page still serving;
     - `accepts: person` with no confirmation or decision in the child.
   - Build the checklist from the child's frozen catalog and its record,
     not from a hand-written list.
   - Apply the same check to the parent's result post (`wfe conversation
     reply --kind result`, `--completing`), which in this case was never
     warned.
   - Same convention as fix5: advisory, refused before anything is
     recorded, continued with `--yes`.
2. **Information at dispatch (support).** `wfe delegate request` prints the
   child catalog's expectations (present, cleanup, …) and says that they
   will be checked when the result comes back. The parent can then write
   them into the request. This is information, not a warning.
3. **A way to send a completed child back (missing today).**
   - An instruction to a completed child is only kept: "an instruction
     recorded on a completed run directly is kept for an attempt that
     reopens work, and nothing continues the run for it" (`docs/runs.md`).
   - Today the only repair is a new delegation, as run-005 was. It loses the
     child's continuity, and the person sees a second run.
   - "Prompting a correction" needs an operation by which the parent reopens
     the completed child with added work, for example a rejection that
     carries a reopening instruction, so that the child adds its present and
     cleanup nodes.
   - The ordering of integration needs a look too: run-004's result was
     integrated before any review.

### Expectations and limits

- The parent did not do the work, and ponderer followed clear reminders
  every time in this conversation. A reviewer may follow "there is
  something to show" more readily than the worker did. This is a
  hypothesis until a trial shows it.
- The child's own reminders stay. The parent's check is a second net for a
  child that continued with `--yes`.
- The parent may also learn to answer with `--yes`, the same limitation
  fix5 accepted. As in fix5, the plan should record it rather than hide the
  flag or add approval.

## Open for fix6's braindump and plan

- The scope: warnings at disposition and at the parent's result post, the
  send-back path, information at dispatch, and possibly A and B.
- Whether a parent's reopening of a completed child, and integration
  before the person's review, belong in this episode or need their own.
- A trial: a real facilitated conversation in which work is delegated,
  with no mention of presentation in the request. Observe whether the
  parent sends the child back and whether a presentation reaches the
  person.
