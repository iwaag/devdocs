# agent_standardize p5 — Step 1 report: `agentchat wait`

AI-generated (Omni Agent, 2026-08-21).

## What changed

`agentchat` gained the verb a supervisor needs. Until now an agent could
`send` and it could `read`, but a run that wants to follow a task through had
only snapshots: read, guess how long to be busy, read again. `wait` is the
missing half — block until the other agent says something.

```
agentchat wait <channel> <topic> [--since <message_id>] [--timeout <s>] [--interval <s>]
```

- **Baseline.** Without `--since`, the topic's current last message is the
  starting point, so only what arrives *from now on* counts. With `--since`,
  anything newer than that id counts — including what arrived before the
  command started, which is what makes a resumed supervision correct rather
  than merely convenient.
- **Timeout is a status, not an error.** Exit `3` on timeout, `1` on failure,
  `0` when something arrived. A loop can tell "still quiet" from "it broke"
  without parsing text. `--timeout 0` waits forever.
- **Poll, not long-poll.** A modest interval (default 15 s) over
  `GET /messages` with `include_anchor=false`, which the plan left to the
  implementer. An event queue would have been a second connection lifecycle
  to get wrong for no gain at this cadence; the anchor makes each poll cheap
  and repeat-safe.

Two smaller pieces came with it because `wait` is useless without them:

- `read --since <message_id>` — the same window without the blocking. Step 2's
  fallback shape (a run ends, the next post picks the supervision back up)
  needs exactly this.
- Every printed message now carries its id in its header
  (`[time] Sender (message 724):`), because an id you cannot see is an id you
  cannot pass to `--since`.

`--help` is the documentation, per the tool's own rule, and it was extended
with both — the waiting example says what exit 3 means, and the notes say
where the ids come from. Its examples still name no real channel or topic:
the p2 leak test (`test_help_names_no_real_agent_channel_or_topic`) still
passes unchanged.

## Files

- `pyagag/src/agag/zulip.py` — `topic_since()` (messages strictly newer than
  an anchor) and `topic_last_id()` (the baseline lookup), beside
  `topic_history()`.
- `pyagag/src/agag/chat.py` — the `wait` subcommand and `_wait()`, `read
  --since`, ids in `format_messages()`, usage doc.
- `pyagag/tests/test_chat.py` — 8 new cases: arrival, `--since` skipping the
  baseline lookup, the distinct timeout code, never sleeping past the
  deadline (a fake clock, so no test waits), a rejected interval, and the
  three `read --since` / id cases.
- `pyagag/README.md` — `wait` in the `agag.chat` paragraph, with why the
  timeout has its own exit code.

Tests: pyagag **272 passed** (264 before), agfront **31 passed**.

## Live proof

Both paths were exercised against the real realm, as the Omni Agent, before
anything depended on them.

```text
$ agentchat wait agents intro-agforge-agstudio1 --timeout 20 --interval 5
nothing new in #agents > intro-agforge-agstudio1 after 20s (last seen message 708)
exit=3                                              # 20.17s wall clock
```

Message 708 is p4's re-posted forge introduction — the baseline lookup found
the right last message on its own.

```text
$ agentchat wait front agentchat-wait-check --timeout 60 --interval 3 &   # blocks
$ agentchat send front agentchat-wait-check "p5 step 1 live check: does wait return?"
sent message 724 to #front > agentchat-wait-check

[2026-08-20T17:01:04+00:00] Omni Agent (message 724):
p5 step 1 live check: does wait return?
exit=0
```

The check topic is `agentchat-wait-check`, deliberately not `front-*`: Front
sweeps only `front-*`, so proving the tool cost no agent run.

## Commits

- **pyagag** `4e6e903` — pushed to GitHub.
- **agfront** `8ad5820` — `uv lock --upgrade-package pyagag` + `uv sync`;
  `agfront/.venv/bin/agentchat wait --help` answers, which is the copy Front's
  runs get on PATH. Pushed.

agautolab and agforge were **not** re-locked: neither needs `wait` this
phase, and the plan asked for agfront. Their next ordinary bump picks it up.

## Deus Ex Machina note

Wrote a tool for the agents rather than as one — this is harness work, which
is the Omni Agent's proper half; no note owed.
