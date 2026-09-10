# refactor p3 ex1 — step 2: forge and sage publish executable options

`runtime-profile` left two agents reading as *unknown* in `agentchat
options`, and p3's report kept that as correct-but-open. They are not unknown
any more: both publish a menu generated from their own configuration, and
both **run on what is selected** rather than only listing it.

## What each now publishes

**agforge** (`@**Forge** use <option>`)

| option | pool | covers |
|---|---|---|
| `default` | `anthropic` | everything I do for a request … Not the media model |
| `agy` | `antigravity` | same |
| `agy-claude` | `antigravity` | same |

**arxivsage** (`@**arXiv sage** use <option>`)

| option | pool | covers |
|---|---|---|
| `default` | `anthropic` | every answer I give, in any entrance topic |
| `agy` | `antigravity` | same |

Both menus are derived from `agents.toml` at import time
(`instance.exec_options` / `listener.exec_options`), so a name whose profile
is gone is never advertised, and an unreadable configuration publishes
**nothing** — leaving a reader at *unknown*, which is honest, rather than at
a name that would fail at execution time.

`stub` (the `fake` harness the suites run on) is on neither menu. Its
committed configuration had only `sonnet` and `stub`, which is exactly what
the plan said not to publish; `agy` is the usable alternative, and it is real
on this host — `agy 1.2.0` in `~/.local/bin`, resolved through each agent's
own `.local/agents.local.toml` overlay:

```
agforge   front     agy       -> agy   antigravity/gemini-3.8-flash-medium
agforge   generator agy-claude-> agy   antigravity/claude-sonnet-4-6
arxivsage front     agy       -> agy   antigravity/gemini-3.8-flash-medium
```

resolved with `check_available=True`, so that is the harness answering, not a
declaration.

## The one distinction the plan asked for

Forge is the agent where an execution option is easy to confuse with the
thing being made. `covers` says it in the menu itself:

> everything I do for a request — entrance replies, asset planning,
> generation, and collection after a callback. **Not the media model**

and `agents.toml` says it beside the profiles. Selecting `agy` changes which
harness reads the request, plans it and drives the toolsets; which image,
video or music model produces the asset is decided in the plan and named by
the toolset. Asking for `agy` cannot turn a Wan video request into something
else. A test pins the wording, because this is the sentence a requester acts
on.

Nothing was written into either agent's *guide*. Execution vocabulary travels
as posted content — the introduction's `agag-exec` block — and compiling it
into a guide is the coupling the contract exists to remove.

## Where the selection actually reaches

**forge**, four places, which is the whole shape of one request:

| serving | run | how it gets there |
|---|---|---|
| entrance topic | `front` | `agag.entrance`, already generic — it needed only `SPEC.exec_options` |
| `assetplan-` | `front` | `run_front(prompt, cwd, context.selection)` |
| `assetplan-` | `generator` (planning) | `run_generator(dir, context.selection)` |
| `assetrun-` | `generator` (execution **and** collection) | `run_generator(workspace, context.selection)` |

`handle_topic`, `handle_assetrun` and the entrance each pass
`exec_options=exec_options_for(SPEC, client)` to `serve_topic`, which is what
makes a command in the topic a command at all.

**Inheritance.** `record.open_run` — the function that opens the request's
`assetrun-` topic when the plan is recorded — now writes
`[selfnote][exec] <option> from <channel>/<topic>#<id>` before the visible
description, exactly as autolab's `workrun-` opening does. It is a snapshot:
re-planning finds the topic already anchored and writes nothing, so a run
already under way keeps what it started with, and a command posted in the run
topic is newer and wins.

**The callback.** A ComfyUI notifier callback is an ordinary post in the
`assetrun-` topic, so the collecting run reads its selection from **home**
like any other serving — the inherited snapshot is still the newest directive
and the collection runs the way the request was asked for. This is exactly
the "callback's remote topic is never the execution context" rule, and here
it needed no special case at all.

**arxivsage**, both of its routes: `handle_sage` (the `entrance-` topics) and
`redirect` (the own-channel door). The redirect was the one worth not
forgetting — without the menu there, a selection posted at the door is met
with a lecture about topic names while the setting silently does not land.
`serve_sage` passes `selection=context.selection` to its `run_role`.

## Verified

`agforge`: 236 passed. `arxivsage`: 11 passed (a new suite — it had none).

Coverage, and where it is a fixture rather than a live run:

| property | how |
|---|---|
| discovery — the block round-trips through `parse_options` | fixture |
| selection reaches the front and the planning generator | fixture |
| selection reaches the run generator from a topic command | fixture |
| the run topic inherits the plan's selection, with provenance | fixture |
| a plan with no selection writes **no** snapshot | fixture |
| a command in the run topic overrides what it inherited | fixture |
| the collecting run after a callback keeps the inherited option | fixture |
| a configuration-only post costs no run and is answered | fixture |
| an unpublished name is refused, names the menu, and never becomes the setting | fixture |
| `use default` returns the topic to the configured defaults | fixture |
| a command posted mid-run lands on the **next** serving | fixture |
| both sage routes carry the menu | fixture |
| the menu never offers `stub`, `sonnet` or `local` | fixture |
| `agy` resolves to a real available harness on this host | **live**, on agstudio |

Two call-sequence assertions in `test_assetplan_topic.py` changed and the
reason is worth writing down: with `exec_options` in play `serve_topic` reads
the topic **before** the ack, so a configuration-only post costs neither an
ack nor a run. The extra `whoami` is `exec_options_for` asking which Zulip
name this instance is mentioned by.

## Not done here, deliberately

- **The introductions have not been re-posted.** The block is generated from
  the *running* instance, and these listeners are still running the old code;
  publishing a menu the running listener would not obey is exactly the drift
  the generated block exists to prevent. Deploy, restart and repost are step
  5's, together.
- **The pools are declared, not yet derived.** `default | pool: anthropic` is
  written down in `DEFAULT_OPTION_DETAIL` the way Front's and autolab's are.
  Checking a declaration against the harness that actually resolves is step
  3's whole subject, and it will replace all four hand-written pools.
- Everything above except harness availability is fixture coverage. The live
  demonstration — discovering these menus through the introductions and
  running real requests under an advertised alternative — is step 5's, and
  will be recorded separately.
