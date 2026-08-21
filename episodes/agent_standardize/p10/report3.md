# p10 Step 3 — the entrance runs on `roles.front`

Both agents, the same change: the entrance stops being a canned line and
becomes an ordinary serving of an ordinary role.

## The model unification, first

`agautolab/agents.toml` `[roles.front]`: `local` → `sonnet`, and
`requires = ["nested_harness"]` → `[]`. That requirement belonged to the
in-process backend the `local` profile used, not to the role.

`.local/agents.local.toml` sweep, all three agents:

| agent | `local`-steering override | action |
|---|---|---|
| agautolab | `[roles.front] profile = "local"` | removed |
| agforge | none (`[roles.generator] profile = "sonnet"`, redundant) | left |
| agfront | none | left |

Also checked: `.local/projects/*/agents.toml` — the per-project overlay that
may steer `coding` and `director` — has no file at all, so nothing is steered
there either. Resolved live, every autolab role now answers
`sonnet / claude_code / anthropic/claude-sonnet-5`.

The `local` profile definition stays in all three files, defined and unused.
Agent ≠ Model; every run records its backend, so an unused profile costs
nothing and deleting it would be pretending the choice was never available.

**One thing to watch:** the gateway's `POST /window` runs the same
`roles.front`, so the agautolab1 node's window is on `claude_code` now too.
That node has its own `.local` overlay, which this Mac cannot see. Noted in
`.local/devenv.md`; not blocking, because the node runs no listener and this
phase is proved on agstudio.

## The entrance, both agents

`entrance_reply()` is gone from both. In its place, `serve_topic` over a
generation workspace, exactly as every other topic is served — ack, chatlog
as a file, one role run, always a reply, the post-run re-check.

- **agforge**: a new `src/agforge/entrance_topic.py`, reached from `dispatch`
  when a topic in the instance's own channel is neither `assetplan-` nor
  `assetrun-`. 600 s.
- **agautolab**: `serve_entrance` / `handle_entrance` in the listener beside
  `serve_bmining`; `dispatch` sends every topic in the instance's own channel
  there, whatever it is named — a `workplan-` name in the entrance is still
  not a request to run anything. 900 s, because a survey across projects is
  many small reads.

Both runs get `AGENTCHAT_HOME` naming the conversation being served, so
anything the entrance says elsewhere resolves back to it.

agforge needed the chat handover autolab has had since p6 — it had none.
`role_run.chat_environment()` adds `agentchat` to PATH **prepended to**
what `tool_environment()` built (a run needs both `agentchat` and agforge's
own tools; overwriting `PATH` instead of prepending would have silently
broken every generator run), `AGENTCHAT_ZULIP_ENV` pointing at the bot's
credentials file, and `AGENTCHAT_HOME`. The `front` grant gains
`Bash(agentchat:*)`.

## The guides

`agent/guides/entrance_front/guide.md` in both, 8 lines of body each.

forge's: its own channel's topics are its plans (`assetplan-`) and runs
(`assetrun-`); `✔` means finished; `read` for detail; look up only what was
asked; a new request is a new topic, not something started here; resolve
finished ones **when asked**, after reading them to check.

autolab's: `channels --prefix pj-` are the projects and their `workplan-`
topics the missions; `channels --prefix work-` are the execution channels and
each description names its project and mission; `workrun-task<N>-…` topics
with `✔` are finished tasks; development work is not started here; asked to
close out, verify, `agentchat resolve`, then `uv run python -m
agautolab.mission_done`.

The `uv run` form is checked: a run's cwd is under the agent's own
`.local/topics/…`, so `uv` discovers the project by walking up and the guide
carries no absolute path.

A test asserts forge's guide names none of autolab's vocabulary
(`workplan-`, `workrun-`, `pj-`, `autolab`, `front-`). Its own channel's name
is not in the guide either — it arrives as a placement line from
`instance_name()`, which is this agent's name for its own entrance, not
somebody else's routing.

## Introductions

One paragraph each, re-posted:

- forge (`#agents` > `intro-agforge-agstudio1`, message 1217): "Ask about my
  work in my own channel… where I answer what I have planned and how far each
  plan has got, and where I will close out the finished ones if you ask me
  to."
- autolab (message 1218): "…ask about my work there and I answer: which
  missions I have planned, across which projects, and how far each of their
  tasks has got. Ask me there to close out the finished ones and I will,
  marking their Work Done."

That is the whole of what Step 4 gives Front. Nothing else was told to it.

**The forge channel description was stale and is fixed.** It said "Open a
`create-…` topic", a prefix retired in p3 — a reader following it would have
hit a name matching no sweep. Now: "Open an `assetplan-… topic to request an
asset; any other topic is a question about this instance and its work."
Neither the forge bot nor the Omni Agent may administer that channel
(HTTP 400, "You do not have permission to administer this channel"); only the
developer account could. *Did X for agent Y — handoff candidate: an agent
cannot correct its own channel's description, so a stale one can only be
fixed by a human.*

## Tests and deployment

agforge 202 passed (10 new in `tests/test_entrance_topic.py`: the workspace,
the conversation as a file, our own ack not counting as conversation,
`AGENTCHAT_HOME`, a new generation per serving, a silent run still saying
something, a failed run raising, the prompt, guide terseness, no foreign
routing). agautolab 179 passed (the entrance-run tests replace the
canned-reply one).

Pushed: agforge `567f829`, agautolab `49f337c`, superproject `ebf3a5a`. Both
listeners kickstarted and registered their queues clean.
