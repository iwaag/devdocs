# p2 step 1 — `agentchat` in pyagag

AI-generated (Omni Agent, 2026-08-20).

## What was built

`agag.chat`, exposed as the `agentchat` console script
(`[project.scripts]` in pyagag's `pyproject.toml`). It is the agent-facing
half of the chat entrance: the listener already speaks Zulip on the harness's
behalf, and this is what an agentic *run* calls when it decides to talk to
another agent.

Three subcommands, the minimum the plan asked for:

- `agentchat send <channel> <topic> <text…>` — posts, printing the message
  id. A topic that does not exist yet is created by posting into it, which is
  how a new request is opened. An empty message is refused rather than
  starting somebody's run over nothing.
- `agentchat read <channel> <topic> [--count N]` — the conversation, oldest
  first, each message with its sender and UTC timestamp.
- `agentchat topics <channel>` — the channel's topic names, most recently
  active first.

No `wait`/polling: deferred, as planned. p2 stops at create.

## Identity

`AGENTCHAT_ZULIP_ENV` names a bot credentials file and `ZulipClient.from_env`
does the rest. Whoever's env is set is who speaks — there is no `--as` flag
and no shared service account, so the caller's identity is decided by
whoever launched the run. Both failure modes (variable unset, variable
pointing at a non-file) fail with a message naming the variable, because an
agent that hits this can only be helped by learning what was supposed to be
set for it.

## No subscriptions

Zulip lets a bot post into and read any public channel without subscribing,
so `agentchat` never calls a subscription endpoint — the test client raises
if it does. Subscription stays the listener's sweep-routing decision,
untouched this phase.

Proven live rather than assumed: the **front** bot, which is subscribed only
to `#front` and `#general`, listed and read `#agents`:

```
$ AGENTCHAT_ZULIP_ENV=.local/zulip.env agentchat topics agents
intro-agforge-agstudio1
$ AGENTCHAT_ZULIP_ENV=.local/zulip.env agentchat read agents intro-agforge-agstudio1 --count 1
[2026-08-20T07:32:28+00:00] agforge-agstudio1:
# agforge
… To request an asset, open a `create-…` topic in this instance's
`agforge-agstudio1` channel and describe what you want.
```

That is Step 2's whole premise, checked before Step 2 depends on it.

## Documentation

`--help` is the tool's only documentation and is written as a usage document
— what a channel and a topic are, that another agent's entrance is a channel
of its own, that a topic prefix is how you say what kind of request this is,
and four worked examples. Tool Giving: a command this powerful handed over
with a bare argparse synopsis would be an Unexplained Chainsaw. A pinned test
keeps `Examples` and all three subcommands in that output. pyagag's README
gained a matching paragraph beside the `agag.zulip` one.

## Verification

- `uv run pytest` in pyagag: **253 passed**, of which 13 are the new
  `tests/test_chat.py`. Nothing asserts what an agent said; what is pinned is
  which credentials file speaks, which Zulip calls each subcommand makes,
  that no subscription call is ever made, and that a Zulip failure exits 1
  with a message instead of a traceback.
- Live smoke test as two different identities, shown above.
- `agentchat --help` renders from the installed agfront environment.

## Delivery

- pyagag `8a45527` — committed and **pushed to GitHub** (`main`).
- agfront `190fafe` — `uv lock --upgrade-package pyagag`
  (`3b289fc9` → `8a45527e`), then `uv sync`, so `.venv/bin/agentchat` exists
  in the environment the Front run will inherit. Per localrule.md: GitHub
  source only, no local path and no gitea.
- Nothing else was re-locked this phase, as planned. Rollout to autolab and
  cagent stays deferred.
