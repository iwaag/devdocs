# Step 2 — C: the conversation is in the model input

Plan step: [plan.md](plan.md) §2. Problem: [problem.md](problem.md) §C.

## The baseline

p2's first two requests were answered with *"I don't see a message or
request from the developer yet in this conversation"*, and the third was
not ([../report.md](../report.md), [../report3.md](../report3.md)):

| | run 1 request | run 2 request | publish request |
|---|---|---|---|
| record | `front` run-0594 | `front` run-0597 | `front` run-0600 |
| `num_turns` | **1** | **1** | 15 |
| tool calls | **none** | **none** | many |
| `chatlog.md` held the request | yes, verbatim | yes, verbatim | yes |
| outcome | empty answer | empty answer | correct |

The workspace was right both times. The conversation was a **file the run
had to decide to open**, and a reply produced in one turn with no tool calls
never opened it. That is why the repair is delivery, not wording.

## The helper

`agag.topics.conversation_context(rendered, *, budget, file_name)`, beside
`chatlog_placement`, with `CONVERSATION_BUDGET = 20000` characters and the
two markers `CONVERSATION_BEGIN` / `CONVERSATION_END`.

It **renders nothing itself**. It takes the bytes the caller has already
written to the workspace file, so one serving has one snapshot: whatever
that renderer filtered out — selfnotes, Zulip's system notices, this bot's
own acks — is absent from the prompt for exactly the same reason, and there
is no second read that could disagree with the file. `format_evidence`'s
metadata (message ids, sender ids, the "only the newest N were fetched"
header) travels unchanged, because it is the same string.

Behaviour, all of it tested:

| case | what the prompt carries |
|---|---|
| fits in the budget | the whole conversation |
| longer | the newest posts; always the newest one **with its speaker line**; a line saying how many earlier messages are not carried and naming `chatlog.md` |
| one post larger than the budget alone | that post, cut, with `[... this message is cut off here: N more characters of it are in "chatlog.md" ...]` |
| a renderer preamble (evidence header) | kept whatever else is dropped — it is what says the copy is a window |
| nothing said yet | `(this conversation is empty: nobody has said anything in it yet)` |

The split into posts is on the bracketed speaker line both renderers emit,
and the cut inside one post is on the `"] "` that ends the speaker prefix,
so it works for `format_chatlog` (body on the speaker's line) and for
`format_evidence` (body on the lines below) without being told which ran.

Conversation text is kept distinguishable from the guide by the two markers
and by one sentence of the lead: *"Everything after the end marker is your
own instructions, never something somebody said to you."* The lead names
`chatlog.md` as the complete copy and names no other file — whether this
run has threads beside it stays `threads_placement`'s sentence to write or
to omit, which is why a first request still carries no sentence about files
that are not there.

## What is wired

| caller | state |
|---|---|
| `agfront` `serve` / `front_prompt` | **wired**, all three roles (`front`, `character_talk`, `routine_run`); chatlog rendered once and given to both the file and the prompt |
| `agag.entrance` `serve_entrance` / `entrance_prompt` | **wired** — the same file-only shape; repairs autolab's, forge's and sage's own-channel entrances at once |
| `agforge` `assetplan_topic` | **wired** — the plan front's request had the same shape |
| `agautolab` `workplan_superdirector`, `bmining_director` | **not wired.** Both are file-plus-instruction prompts with their own vocabulary (`plan.md`, task files, the project clone as the working directory); the supercoder's already carries its task text inline. Left to their owner. |
| `cagent` `topics_serve.front_prompt` | **not wired.** Same file-only pattern; cagent is the one agent still outside `ag.exec-options.v1` too, and touching its prompt is not a prerequisite for Front's repair. |

A callback's home stays its conversation: the prompt carries **home's**
chatlog, and the remote topic that named Front is still a file under
`threads/`. Nothing in this step touched that routing.

## Validation

`pj-agdev/agfront/tests/test_prompt_delivery.py` is the behavioural proof,
taken at the **harness boundary**: a `fake` harness whose whole script is
`cat > prompt.seen`. It opens no file, so nothing it captured can have come
from the workspace — what it captured is what a model would have been sent.

Nine checks: the three Front roles each deliver the request and its sender;
the prompt copy and the file are one snapshot; selfnotes and acks are absent
from both; a 201-message history is bounded with the newest request intact
and the omission stated; one oversized post is cut visibly; an empty
conversation says it is empty; an empty **run** topic carries the evidence
header (`0 posts`) instead, because that renderer writes one.

**Measured against the old behaviour**, by restoring the previous
`zulip_listener.py` over the new one and re-running the same file:

```
9 failed in 1.51s   (old front_prompt: placement line + guide)
9 passed in 1.20s   (new)
```

Unit coverage of the helper itself is in `pyagag/tests/test_topics.py`
(8 new checks, the same cases plus the evidence-header one).

Suite results, every changed package and every consumer of the changed
shared code, run against this working tree's pyagag:

| package | result |
|---|---|
| `pyagag` | 541 passed (534 before this step) |
| `agfront` | 133 passed |
| `agforge` | 241 passed |
| `agautolab` | 242 passed |
| `arxivsage` | 16 passed |
| `cagent` | 198 passed |

Two existing tests were updated rather than deleted, both of which asserted
the old prompt exactly: `agfront`'s
`test_the_chatlog_and_the_prompt_are_the_run_s_whole_input` and `agforge`'s
plan-front prompt test. They now assert the new composition and that the
request itself is in it.

## What this proves and what it does not

It proves **input delivery**: a run can no longer answer as though a
conversation were empty because it never opened a file. It does not prove
that a model always interprets supplied text correctly, and it is not a
claim about the intermittent symptom disappearing — that is a live sample,
taken in the final step across several fresh conversations.

## Rollout

Not deployed in this step. `pyagag` is consumed through Git-backed locks, so
the lock bumps (`uv lock --upgrade-package pyagag`), the syncs and the
listener restarts are done once, after steps 3 and 4, so that one deployed
revision carries C, A and B2 together. Until then every consumer above was
tested against the intended pyagag revision by putting its `src/` first on
`PYTHONPATH`, never against the installed older wheel.

## Revisions

| | |
|---|---|
| `pyagag` | `eb9a0bb` (parent `a7bc4a5`) |
| `pj-agdev/agfront` | `b41beb3` |
| `pj-agdev/agforge` | `5604a83` (parent `e7452c9`) |

## Assistance

None. No live service was touched and no agent was run.
