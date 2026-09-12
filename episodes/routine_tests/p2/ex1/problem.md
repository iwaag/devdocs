# routine_tests p2 ex1 — three defects the trial demonstrated, and what to do about them

Source: [../report.md](../report.md), with the evidence in
[../report3.md](../report3.md), [../report4.md](../report4.md) and
[../report5.md](../report5.md).

p2 delivered its research and its publication, but seven interventions were
needed and three defects are open. Two of them are the same shape — a
routing anchor that does not survive a change of context — and the third is
why every request had to be typed twice.

None of these were repaired during the trial, deliberately: the plan said to
repair only demonstrated defects at their owner, and the developer assigned
the root causes here.

---

## Problem A — retiring a plan strands the supervising run's anchor

### What happened

Front opened a run, delegated it to
`#pj-studyuspolitics` › `workplan-collect-and-analyze-contributions`, and
anchored that topic with its root note, message **6367**. The first attempt
stalled, the plan was scrapped, and autolab retired mission m6371.

After the retirement, autolab named `@Front` correctly — twice, in two
different topics — and Front refused both:

```
mention in 'work-m6400'/'workrun-task1-m6400' carries no root note of ours; ignoring
mention in 'pj-studyuspolitics'/'workplan-collect-and-analyze-contributions' carries no root note of ours; ignoring
```

Neither agent was wrong. Front's rule — only a topic carrying its own note
reaches its run — is what protects it from p1's twin-forking defect.

### Mechanism

`retire_conversation` (`agautolab/src/agautolab/worklog.py:608`)
renames the **whole topic** to `✔ retired-<name>-<label>`, and the
replacement mission then takes the freed display name. Renaming a topic
moves every message in it — including **other agents'** root notes. Front's
6367 went into `✔ retired-workplan-collect-and-analyze-contributions-m6371`;
the live topic the replacement opened under the old name carried no Front
anchor at all.

The same file already reasons correctly about this for the *task* half.
`retire_task` says the task topic is deliberately **not** renamed because
"a late callback still has to find this conversation under the name it was
delegated from". That reasoning was never extended to the third parties
anchored in a workplan topic.

Autolab cannot fix it by copying the note: a root note is identified by its
sender (`own_rootchat` filters on `sender_id`), so a note autolab writes is
autolab's note, not Front's.

### Proposal A — follow the `replaces` pointer when no own anchor is found

The information needed is **already written**. `refactor` p2 added
`[selfnote][replaces] <message id>` into the replacement conversation
(`agautolab/src/agautolab/anchor.py`), naming
the retired work's anchor **by message id** — precisely because the
replacement takes over the display name. In this trial it was message
**6401**, naming 6371. Message ids survive renames and moves.

So the reader side can be made whole without anybody forging a note:

> When an agent looks for its own root note in a topic and finds none, and
> the topic carries a `[selfnote][replaces] <id>` note, resolve that id to
> the conversation it now lives in and look for its own root note **there**.
> One hop, then stop.

Where: `agag.zulip.rootchat_home` / `own_rootchat`
(`pyagag/src/agag/selfnote.py:191`,
`pyagag/src/agag/zulip.py:978`). It is a shared
convention, so it belongs in `pyagag` and every agent gains it at once.

Why this shape rather than the alternatives:

- **Not** "autolab re-posts other agents' notes" — it cannot, and a forged
  note would make the record lie about who is party to what.
- **Not** "retirement leaves the old name alone" — releasing the display
  name is the whole point of retiring, and `retire_conversation`'s docstring
  is right that a replacement created first would merge into the
  conversation it replaces.
- The `replaces` note is a *relation*, already generic, already by id, and
  already documented as the thing that survives a rename. Reading it is the
  cheapest correct fix.

**Test it bites:** retire a mission in a workplan topic a second agent has
anchored, then have autolab name that agent from the replacement. Without
the change the mention is ignored; with it the callback resolves. Both
halves must be asserted — p1's lesson was that a fix nobody exercised is
covered by tests only.

**Also worth stating in `README_DEV.md`**: retiring a plan is a rename, and
a rename is the thing routing lookups have to survive. That sentence has now
cost two episodes.

---

## Problem B — a run that delegates in the serving that opens it anchors to the Front Desk

### What happened

The `publish` run. Front's `front` role opened
`#routine-publish` › `routinerun-20260912T1636Z` **and delegated to autolab
in the same serving**. The root note it wrote into the delegation topic
(message **6482**) names the **Front Desk** conversation, because that is
the conversation the `front` role was serving.

The `routine_run` role that actually drives the run was never told anything:

- autolab's plan callback served the Desk; the run sat at message 6488
  saying it had seen "only autolab's initial acknowledgment … no plan";
- the task was therefore **never started** until a human answered;
- the completion callback served the Desk again;
- the run wrote no entries, no finish block, and never resolved.

The work itself was never at risk — the `front` role produced an accurate
report — but the run was lost. **This one arose with no intervention at
all**: an ordinary request at the ordinary entrance.

### Mechanism

`AGENTCHAT_HOME` is the conversation being served, and `agentchat send`
writes the root note from it (`pyagag/src/agag/selfnote.py`,
`HOME_VARIABLE`). A role that delegates on behalf of a *different*
conversation of its own has no way to say so.

**Adding a second anchor does not help.** `own_rootchat` is explicit:
*"The **earliest** of our own root notes wins. A topic is anchored once, by
the run that opened it; a later note would be a repeat."* The run's own
anchor was written into the same topic at message 6500 and the completion
callback still resolved to the Desk. That rule is right for its original
purpose and wrong for repair.

The `front` guide
(`agfront/agent/guides/front/guide.md`) says
of the opening post: *"That one post is the start: the run is served after
this reply, as its own conversation, and you must not post into it again."*
It does not say **do not delegate for it in this serving** — and that is
exactly the gap the run fell through.

### Proposal B1 — guidance first: opening a run ends the turn

Add to the `front` guide, beside the sentence above:

> Opening a run is the whole of that reply. Do not delegate for it, ask
> anybody for anything on its behalf, or post its request anywhere: the run
> is served straight after this reply and **the run does its own
> delegating**, so that everything it delegates is anchored to the run and
> its answers come back to it. Tell the developer where you opened it and
> stop.

This is the p1 policy applied honestly: the evidence says the wording was
incomplete, not that a lock is needed. p1 answered its premature-finish-block
defect with guidance and **re-measured** — four self-contradicting blocks
became none. Do the same here: change the sentence, run a routine, and count.

### Proposal B2 — make anchor repair possible at all

Independently of B1, there is currently **no way to correct a
mis-anchored topic**, which is why two of this trial's seven interventions
failed or had to be worked around. p1 needed the same capability four times,
for a different cause.

> Allow a later root note to supersede an earlier one **when it says so**:
> a distinct note, e.g. `[selfnote][rootchat-moved] <channel>/<topic>`,
> written by the same agent, wins over any earlier `rootchat` note in that
> topic. A bare repeat still loses, exactly as now.

This keeps "anchored once" as the default (its reason is sound: a repeat
must not redirect a live conversation) while making a deliberate correction
expressible. It is the generic fix the last two episodes both wanted, and
with it my manual repair at message 6500 would simply have worked.

**Do not** implement B2 as "latest wins" — that reintroduces exactly the
misdirection "earliest wins" was written to prevent.

### Proposal B3 — not recommended yet

A code guard (the listener detecting that a serving both opened a run and
wrote a root note, and rewriting it) is implementable, but it is a shackle
placed before the guidance has been re-measured. Hold it until B1 has run
and failed. Record the measurement either way.

---

## Problem C — Front's first serving of a new conversation can answer as if it were empty

### What happened

Twice out of the first two requests, Front answered a brand-new `front-*`
conversation with *"I don't see a message or request from the developer yet
in this conversation"* — and then **not** on the third. It is intermittent.

| | run 1 request | run 2 request | publish request |
|---|---|---|---|
| record | `front` run-0594 | `front` run-0597 | `front` run-0600 |
| `num_turns` | **1** | **1** | 15 |
| tool calls | **none** | **none** | many |
| `chatlog.md` held the request | yes, verbatim | yes, verbatim | yes |
| outcome | empty answer | empty answer | correct |

Re-stating the request in the same conversation fixed it every time, at one
wasted paid run each ($0.0511, $0.0307).

### Mechanism

The conversation is **a file the model has to decide to read**. The prompt
line is `chatlog_placement`
(`pyagag/src/agag/topics.py:277`):

> "The chatlog is placed in the working directory. You are '<bot>' in the
> chatlog."

A reply produced in one turn with **no tool calls** never opened that file,
so it genuinely saw nothing and said so. The workspace was correct both
times; this is not a delivery fault, and no amount of "read the chatlog
first" wording removes the possibility of a one-turn answer.

### Proposal C — put the conversation in the prompt

> Carry the chatlog's text **in the prompt** rather than only on disk — at
> minimum the newest message and who sent it, ideally the rendered chatlog
> when it is small, with the file kept for the long tail and for threads.

Then "saw nothing" stops being reachable in one turn, and the fix is
deterministic rather than a request for more diligence. It is also the
cheaper prompt in the common case: most `front-*` servings are a handful of
short messages.

Owner: `agfront`'s workspace build
(`agfront/src/agfront/zulip_listener.py`)
and `agag.topics`. Consider whether autolab and forge want the same — the
failure is a property of file-placement, not of Front.

**Alternative, weaker:** have the listener refuse to post a reply that
claims there is no request while a non-empty chatlog exists, and re-serve
once. It is a guard against one phrasing of one symptom; prefer C.

---

## What to do, in order

1. **B1** — the guide sentence. One line, no code, and it is the defect that
   arose unprovoked. Re-measure with a routine run and count.
2. **C** — the chatlog into the prompt. Deterministic, small, and it removes
   a tax on every new conversation.
3. **A** — the `replaces` hop in `pyagag`. The most code, and it only fires
   on the retirement path, which is rare but total when it happens.
4. **B2** — the supersede note, once A is in: both are anchor-lifecycle work
   and share their tests.

Each of these should carry a behavioural check that is verified to **fail
without the change**, and the owning project's required validation, commit
and push, with the deployed revision verified before anything is retried —
the same bar the plan set for this trial's own repairs.

## What is deliberately not proposed

- **No new agent, no dispatcher, no scheduler.** Nothing here needs one.
- **No research-side constraint.** The study routine's guide is the
  researcher's, and its one new rule (ask when a human is needed) has been
  measured once and should be left to accumulate evidence.
- **No guard on the `ag-routinerun` block or on subagent isolation.** Both
  behaved correctly in this trial — the finish block was written exactly
  once per run and at the end, and run 2's subagents used per-subagent
  output directories after run 1's collision. Guidance held; adding locks
  now would be anxiety-driven.
