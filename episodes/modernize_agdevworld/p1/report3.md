# p1 step 3 — the listener, proven with the stub

Done. `agfront` now has code, and the whole route — config pair, harness
seam, generation workspace, command file, dispatch post — is exercised by the
test suite with no real harness and no real Zulip. 19 tests, all passing.

## What was written

| Path | What it is |
|---|---|
| `src/agfront/role_run.py` | role → resolved agent → `run_harness`, plus the per-role tool grant |
| `src/agfront/zulip_listener.py` | `sweep_serve` over `front-*`, `serve_topic` per topic, the dispatch |
| `tests/test_zulip_listener.py` | what agfront decides, against a fake client |
| `tests/test_role_run.py` | resolution, the run record, and one end-to-end stub run |
| `uv.lock` | pyagag pinned at `4f8d611e` |

### `role_run.py`

agforge's shape with its tool handover removed: agfront ships no CLI of its
own and gives its role no PATH, because Front runs nothing.

```
ROLE_ALLOWED_TOOLS = {"front": "Read,Write,Glob,Grep"}
```

No `Bash`. That is the role's definition, not a fence — Front routes, and the
work happens in the agent that receives the dispatch. `Write` is in the grant
for exactly one file, `dispatch.md`. A test pins both halves (no `Bash`,
`Write` present) and another pins that every role in `agents.toml` appears in
the table, because a role missing from it gets no `--allowedTools` at all and
claude_code then blocks on an interactive permission prompt until the timeout.

### `zulip_listener.py`

`sweep_serve(client, handler, topic_filter=("front-",))` for the pull loop,
`serve_topic(...)` for each match. agfront's own part is one function:

```
serve(context):
    <N>/front/  chatlog.md   → front run → its answer, posted
                dispatch.md  → posted to #general, and reported
```

Mechanism **(a)** from the plan, as report2 recorded: the run writes
`dispatch.md`, the handler posts it. First line is the topic, the rest is the
message. Both are required — a dispatch with no topic has no destination, and
one with no body would post an empty message into a channel two other bots
are watching; either is reported as `failed during dispatch: …` and nothing
is posted.

The **channel is not in the file**. `DISPATCH_CHANNEL = "general"` is a
constant in the handler, so `dispatch.md` cannot name a destination, and Front
reaches `#general` because that is the one channel it is subscribed to
besides its own entrance. Subscription stays the routing decision.

No `dispatch.md` is not a failure: a Front that refused, or that only answered
a question, is the normal case. The reply into the `front-*` topic is the
run's own answer, plus one line naming the topic the message went to and
saying the reply will appear *there* — the fire-and-forget honesty the plan
asked for.

Cutting a new generation `<N>/` per serving is what stops a previous
generation's `dispatch.md` from being posted twice; a test runs two servings
and asserts exactly one dispatch.

**No bot loop**, by filter rather than by luck: the sweep prefix is `front-`,
so autolab's replies inside `run-*` topics never match it, and autolab's own
prefixes (`mission-`, `run-`, `create-`) never match `front-*`.

Front's ack is dropped from `chatlog.md` (agforge's rule): leaving it in would
teach Front that "please wait for the reply" is something it once said in
answer to a request. A human quoting the ack still stays.

## The stub proof

`test_a_stub_run_dispatches_through_the_real_harness_seam` monkeypatches no
agfront function. It points `role_run` at a generated `stub`-profile config
pair whose `fake` harness is a three-line shell script, and calls
`handle_topic`. Real: `load_config`/`resolve_role`, `run_harness`, the
subprocess with the prompt on stdin, the generation workspace, `dispatch.md`,
the run record. Stubbed: the harness process itself and `topic_write`.

It asserts the post `("general", "run-stub-1", "Please advance the work.")`
was made, that the reply names that topic, that the prompt the process
received on stdin is the placement line plus **the guide this repository
actually ships** (it looks for `run-<something unique>` in it, not a fixture
string), and that `records/front/run-0001.json` recorded `harness: fake`.

The committed `agents.toml` is untouched by all of this — it stays on
`sonnet`, and nothing in the suite can fall back to it and launch a paid run.

## Not done here

`.local/zulip.env` and the launchd job are step 4; without them nothing is
listening yet.
