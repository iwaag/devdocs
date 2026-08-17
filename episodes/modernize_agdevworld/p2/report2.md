# p2 steps 2–3 — the listener asks forge, and the suite says so

Commit `bc1dbe7`. `18 passed` on the stub profile.

## What agfront decides now

| Before (p1) | After |
|---|---|
| `dispatch.md`, first line the topic | `create.md`, description only |
| `parse_dispatch` split topic from body | deleted — Front never names a topic |
| topic came from the agent | `create_topic_name(topic, generation)` |
| `run-<something unique>` in `#general` | `create-<stem>-<N>` in `#general` |
| reply: "dispatched to …" | reply: "asked forge in #general > …" |

`create-20260817-advance` generation 1 → `create-20260817-advance-1`. Two
properties earn the derivation: the stem reads back to the `front-` topic that
caused the request, and the generation number keeps a second request in the
same conversation out of the first one's topic — the same thing that already
stops an old command file from being posted twice.

`DISPATCH_CHANNEL` became `OUTBOUND_CHANNEL`. The word "dispatch" belonged to
the retired route; the constant's value (`general`) did not change, and the
comment now records *why* that channel works — forge is subscribed to it too.

## Errors, kept to the one that matters

p1 rejected three shapes of command file (no topic, no body, empty). Two of
them existed only because the file carried a topic. What is left is: **an
empty `create.md` is an error**, because a blank post into `#general` would
start a forge run over nothing. A *missing* `create.md` is not an error — it
is two of the guide's three branches (answered, refused) doing exactly what
they should.

## Unchanged on purpose

- `ROLE_ALLOWED_TOOLS["front"] = "Read,Write,Glob,Grep"`. `Write` still exists
  for exactly one file; only the file's name in the comment changed.
- The `front-`-only sweep filter. The bot-loop argument is now symmetric and
  written down in the module: Front sweeps `front-` and never `create-`,
  forge sweeps `create-`/`runcreate-` and never `front-`.
- No mention of autolab is ever synthesised into a create request. autolab
  also watches `create-` topics in `#general` but reacts only when the last
  message mentions it by name, so silence is what keeps it out.

## Test suite

`tests/test_zulip_listener.py` rewritten around the three guide branches;
`tests/test_role_run.py`'s stub-harness wiring proof now writes `create.md`
and asserts the post lands in `create-stub-1`. That test still runs the real
config pair, real `run_harness`, real workspace — only the harness process and
Zulip are stubs. Nothing asserts what an agent said.
