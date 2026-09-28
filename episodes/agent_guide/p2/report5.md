# agent_guide p2 — step 5: the frictions p1's trials hit

pyagag `66f2e4a` on `agent-guide-p2`.

## Several topics, or a channel's latest ones, in one call

p1's runs 0170 and 0171 each lost turns to a shell loop over topics that
the Claude Code harness refused ("Contains simple_expansion").

```
agentchat read <channel> <topic> [<topic> …]
agentchat read <channel> --latest N [--prefix <p>]
```

- Several names print each conversation under `== #channel › topic ==`;
  one name prints exactly as before (no heading), so nothing that parses a
  single read changes.
- `--latest N` reads the channel's N most recently active topics, ✔ ones
  included; an open topic and its ✔ twin count as one conversation.
  `--prefix` narrows them by the bare name (`workplan-`, `routinerun-`,
  `argue-`).
- `--count` and `--since` apply to each topic. Topics and `--latest`
  together, `--prefix` without `--latest`, and no topic at all are refused
  with a line saying what to give instead.
- `read --help` says all of this with three examples and the trial it
  comes from; the top-level index lists `read <channel> <topic>…` with
  "several topics, or --latest N [--prefix p] of a channel, in one call".
  No guide text was added (the plan's instruction).

In passing, `--since`'s help said "nothing new prints nothing"; it prints
one line saying so (MEMORY "agentchat read --since is never silent"). The
help now says what it does.

## `topics --prefix`

run-0170 guessed `agentchat topics --prefix`. It exists now: it keeps the
names whose bare name starts with the prefix, ✔ or not, newest first, and
says so when none does. The index line reads `topics <channel> [--prefix
p]`.

## Reads come from the mirror

Before this step `agentchat` read Zulip directly for every command; only
`trace`'s `MirrorReader` (Observer's) read a mirror. Now:

- `AGENTCHAT_MIRROR` names a mirror store. `agag.agent.chat_environment`
  sets it for every run to the listener's own
  `<instance>/.local/mirror/mirror.sqlite`, so one change in pyagag covers
  every agent.
- The look-only commands — `read`, `topics`, `channels`, `intro`,
  `options`, `trace`, and `agproject status` — are answered from it
  through `agag.mirror.reads.MirrorReads`, which opens the store read-only
  (`Store.open_readonly`: no schema script, no journal change; a write
  fails).
- What the store cannot answer goes to Zulip as before: a channel it does
  not hold (private), an archived channel's topic it never read, a message
  it never saw, subscribers, users, every write. A store whose mirror has
  no event queue yet (being built) is not used at all.
- `recheck` keeps reading Zulip: its verdict is acted on at once, and a
  second of event lag must not decide whether work is started twice.

Checked on the live board: `read pj-growbox --latest 3 --prefix workplan-`
through agfront's mirror printed exactly what three direct Zulip reads
printed, in 0.36 s with no read call.

The listener's mirror is updated from its event queue as posts arrive, and
a run exists only while the listener that started it is running, so the
store a run reads is the live one. The remaining gap is a post the run made
itself and reads back within the event's delivery time (typically under a
second); no current guide or help asks a run to do that.

## A fixture store

A store whose meta carries `fixture=<name>` is a fixture board (step 6):
every command, reads or not, is answered from it; there is no live side,
so a write, or a read beyond what it holds, fails with a line naming the
fixture. `whoami` answers from the store's meta. This is what lets a trial
run read a synthetic board while it cannot post anywhere.

## Tests

`tests/test_read_many.py` (12): several topics with headings, one topic
unchanged, `--latest` and `--prefix` (✔ included), the three refusals,
`topics --prefix`, the index lines, mirror answers vs live fallback, a
mirror without a queue is unused, only look commands use it, a fixture
answers everything and writes nothing, read-only open. Existing tests that
replaced `client_from_environment` with a zero-argument lambda take
`reads=` now. pyagag: **1145 passed**.
