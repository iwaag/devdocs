# Step 4 report — Carry forward the context needed to continue

Plan: [plan.md](plan.md) step 4. Repositories: `pyagag` (`5ae95f7`), `agfront` (`def0af2`).

## The continuation view

`agag.continuation` (new) extends the conversation-in-prompt mechanism with
one compact block, between `===== BEGIN CONTINUATION =====` and
`===== END CONTINUATION =====`, carried after the conversation and before
the guide. It is **derived from recorded evidence**, and it says what it
could not derive:

- **New input.** Speech by others past the input boundary of the last
  *delivered* serving of this home (`serving.input_up_to`, read from the
  journal by `last_delivered_for(channel, topic)` whatever route brought
  that serving); without a record, past this bot's last real post. The
  newest posts are quoted (bounded to 240 characters each, the newest 8);
  more are counted, not dropped silently. A callback says which remote post
  brought it.
- **The agent's own summary.** Goal, agreed conditions and intended next
  actions as the agent last wrote them, with the message id it was written
  after. Any post newer than that id is listed above it and the view says
  it overrides the summary where they disagree — the newest instruction
  wins.
- **Outstanding requests.** Every conversation this one reached out to
  (its root notes, the same list `threads/` is built from), each with a
  state read from evidence: `awaiting` (our post is the last real one),
  `answered, not yet dealt with` (a reply above the served mark, quoted),
  `answered and already dealt with` (at or below the mark, from the
  listener's mirror), `finished (✔)`, or `could not be read … unknown`.
- **An interrupted previous serving** of the same input (step 1's
  `context.previous`): the stage it reached and its ack id, with the
  instruction to check the threads and workspace before repeating actions.
- **What is omitted.** How many earlier posts the carried conversation
  left out (parsed from `conversation_context`'s own notice) and how many
  threads could not be read — "unknown, not settled".

## The agent's part

The agent keeps the semantic part itself, in one machine block anywhere
in its output:

    ```ag-continue
    goal: <one line>
    conditions: <agreed so far, with message ids where it matters>
    next: <what to do when which answer arrives; what if it does not>
    ```

`serve_topic` reads it off the whole output before the reply contract, so
it is never posted; after the reply is delivered it is written into the
served conversation as `[selfnote][continuation] {"after": <input id>,
…}` — a selfnote, hidden from chatlogs, buying nobody a run — and the next
serving's view reads the newest one this bot wrote. An unreadable block is
a visible notice after the reply and no note. The block is described once,
`CONTINUATION_GUIDE`, appended by `prompt_with_guide(…, continuation=True)`
after the reply guide; the guide says not to restate what has not changed.

No separate memory service: the note is the representation, the journal
and the mirror are the evidence.

## Front

`continuation_for(context, chatlog_text, remotes)` builds the view from
what the serving already has: the history as read (acks dropped as for the
chatlog), the journal's last delivered serving of this home, the served
marks off the listener's mirror (`current_mirror()`), the interrupted
record, and the thread snapshots `write_evidence_threads` now hands back
(`collected=`) so nothing is read twice. It is refreshed for every serving
of `front`, `desk`, `routine_run` and `argue`, callbacks included; the
callback's remote is named as what brought the serving.

## Evidence

`pyagag`: **681 passed** (12 new in `tests/test_continuation.py`).
`agfront` against this checkout: **181 passed** (the prompt-shape test now
checks the view sits between the conversation and the guide).

The plan's scenario is `tests/test_continuation.py::scenario`: the
developer asks for a trailer; Front delegates to autolab and forge and
records its plan after message 22; the developer posts a correction (#30)
while both delegates run; autolab answers in its topic (#31); the
listener restarts (an interrupted record at `acked`, ack #32); the history
carried is truncated before #18; a third thread cannot be read. The next
run's view is pinned to say:

| Question | Answer in the view |
|---|---|
| what arrived | "Your last reply here is message 22; it answered everything up to message 19." then `[Dev #30] correction: make it 20 seconds, not 30`, and "brought by a post in #pj-x › workrun-1 (message 31)"; the older history is not quoted |
| what was decided | goal / conditions / next as recorded, "written after message 22", and "Posts newer than message 22 (listed above) override this where they disagree" |
| what remains pending | `#pj-x › workrun-1: answered, not yet dealt with: [autolab #31] …`; `#agforge › assetplan-t: awaiting a reply to your post (message 21).`; `#pj-x › workrun-0: could not be read … unknown` |
| what not to redo | "A previous run on this same input was interrupted at stage 'acked' (acknowledged as message 32) … Check the threads and the workspace before repeating any of them." |
| what is missing | "3 earlier messages … not carried in the prompt (before message 18)"; "1 thread could not be read; the state of that request is unknown, not settled." |

Also pinned: the served and finished states; a brand-new conversation;
more new posts than the quote limit; the block kept out of the post and
the note written after the reply, then read back by the next view; an
unreadable block as a notice.

## Remaining gaps

- Consumers other than Front do not build the view yet (step 5 decides
  which of their roles are conversational enough to want it; the shared
  entrance answers one question and delegates nothing).
- The view names remotes by the root notes; a delegation made without a
  root note (a shell `agentchat send` with no `AGENTCHAT_HOME`) is
  invisible to it, as it is to `threads/`.
