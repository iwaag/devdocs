# failsafe p2 — step 2: task-specific execution health

## The interface

A monitor asks an owner one question: **is the work of the serving acked by
`#<ack>` still being done?** The answer is an `agag.health.v1` document.
The facts in it carry observation times and sources, and the document
lists what could not be established. The command is:

```sh
python -m agag.health --dir <live execution records> --queue <listener.sqlite> \
    --ack <id> --channel <c> --topic <t> [--window 120]
```

It is a command with its own process, so the monitor calls it with a
timeout of its own (step 3). An owner without it still takes part in the
system; its units of work just stay on the conversation-only rules.

| section | content | source |
|---|---|---|
| `process` | `alive` / `exited` / `unknown`, pid, observed time, how it ended (the runner's recorded end, not in the process table, pid reused) | the record's pid checked in one `ps` read; the runner's recorded end |
| `progress` | last harness event: time, age, kind (`tool Bash`, `tool result`, `text`, `generating`), counts | the live execution record |
| `wait` | `tool` (name, the telling argument, since, open count, processes under it), `children` (processes under the harness), `none`, `unknown` | the record's open tool calls; the process tree under the pid |
| `serving` | the listener journal's stage for that ack (`acked` … `delivered`, and whether anything was posted), whether the conversation is queued or served again | the queue file, read-only |
| `verdict`, `why`, `unknowns` | `running`, `waiting`, `stopped`, `ended`, `unknown` | derived from the above only |

The verdicts (`agag.health`):

- **`running`**: the process is alive and its last event arrived within
  the window.
- **`waiting`**: the process is alive and quiet, and a tool call not yet
  returned, or a process under it, explains the quiet, inside the run's
  own deadline. A live process is not enough by itself.
- **`stopped`**, confirmed:
  - the harness process is gone and nobody recorded an end; or
  - the run ended and its serving posted nothing (`delivered` with no
    message, `acked`, `executed`);
  - and in either case nothing is queued to serve the conversation again.
- **`ended`**: the run is over and a reply was or is being posted, or the
  conversation is already served again (a listener restart requeues an
  interrupted serving). That is the conversation's business, not a stop.
- **`unknown`**, in these cases:
  - the process is alive with no event for longer than the window and
    nothing names a wait;
  - the process is alive past its own deadline;
  - a source could not be read;
  - there is no record for this serving.

What it deliberately does not claim:

- A listener heartbeat is not task health. The probe looks at the harness
  process of that one serving.
- Lack of output is not a failure. A quiet process with a named wait is
  `waiting`.
- **Stale evidence is never applied.** The subject is the ack. A record
  for the same conversation from another serving is listed in
  `other_servings` and concludes nothing. A pid whose process started
  after the record did belongs to another process.

## Where the facts come from

- **`agag.execution`** (pyagag `bea4583`): `run_harness(live=…)` keeps
  one JSON record per run:
  - the serving (journal id, ack, conversation, route, role);
  - harness, pid, start, deadline;
  - last event and counts;
  - the tool calls not yet returned (`claude_code` and `agcode` streams
    carry tool results; `agy`, `codex` and `gemini` say they cannot tell);
  - the end (exit code, outcome), written by the runner.

  Writes are atomic. Tool starts and ends are written at once, other
  events at most every 5 s, and a failed write never touches the run.
  `run_role(live=…)` takes the serving from the thread's journal.
- **Long generations count as progress.** With a live record, `claude_code`
  runs with `--include-partial-messages`, so a model writing a long answer
  keeps producing events. Those chunks go to the record only. They are
  dropped from the captured output and the transcript, and extraction is
  unchanged (tested).
- **agautolab** (`3cbd378`): every role run inside a listener serving keeps
  a record in `.local/executions/`, named `s<serving>-<role>-<ns>.json`.
  The newest 200 are kept. A run outside a serving (CLI, gateway, tests)
  keeps none.
- **The listener's queue file** is read with `mode=ro` and a 2 s timeout.
- **One `ps -A` read**, with a 5 s timeout. An idle Claude Code process
  has no children, and a running Bash tool shows as `zsh -c …` under it,
  so descendant processes are a usable wait signal on this host.

**Cost**: the probe against the real autolab queue took 0.07 s.

## The trial fault

agautolab `faults/silent-exit` (one-shot, created only by a person):

- the next task serving's harness is killed with SIGKILL at its first tool
  call (a real process termination);
- the serving then ends with an empty reply, so nothing is posted after
  the ack.

What stays is what any unreported exit leaves:

- an ack with no reply, so the trace reads `execution=open`;
- a live record with an end the runner wrote (`exit -9`);
- a journal row `delivered` with no message.

The probe reads that as `stopped` (tested).

## Tests

- pyagag: 952 passed, 17 of them in `test_health.py`. They cover:
  - a real stub harness writing the record, including the open subagent,
    `--include-partial-messages`, and no partial chunks in the transcript;
  - each verdict;
  - pid reuse;
  - another serving's record;
  - an unreadable process table;
  - the command-line document.
- agautolab: 312 passed. New tests:
  - a live record inside a serving and none outside it;
  - pruning;
  - the fault kills only its own serving's live run and posts nothing but
    the ack.

## Limitations

- Only `claude_code` and `agcode` say when a tool call returns. For the
  other harnesses a quiet run is `waiting` only through child processes,
  and otherwise `unknown`.
- The probe is host-local: it reads files and the process table.
  - Observer and autolab both run on agstudio, which is what p2 covers.
  - An owner on another host would expose the same document another way.
  - Cross-host execution is out of scope.
