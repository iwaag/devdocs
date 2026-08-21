# agent_standardize p6 — Step 3 report: what a delegating run needs

AI-generated (Omni Agent, 2026-08-21).

agautolab `65850ef` and `51301e7`; agforge `f154f49` (text only).

Step 2 removed the route. This step gives the ordinary path the three things
it was missing — knowledge, an identity, and time — plus the two guides and
the one introduction that make those things usable.

## The board reaches both runs

`write_agents_md` (pyagag, Step 1) is now called in `serve` and in
`serve_run`, immediately after the chatlog is written and before anything
expensive happens. Both prompts gained one line naming
`<workspace>/tools/agents.md`.

Both roles need it and for different reasons. The superdirector reads it at
planning time to know that another agent exists at all and to write the
delegation into a task. The supercoder reads it at execution time to learn
that agent's entrance, its topic vocabulary and what it calls finished. The
plan's own decision — who and what are decided at planning time, the
conversation happens at execution time — is why the file has to be in both
places and not just one.

Nothing is cached, per serving. Four tests pin it: the file lands in the
planning workspace and is named in the planning prompt; the same for the task
workspace; an empty board still lets the run happen (the honest sentence, not
a failure); and two servings of one topic see two different boards.

The test client serves `#agents` without recording the read, so the harvest
exercises the real code path in every existing flow test without appearing in
the call sequences those tests assert on.

## An identity to speak with

`role_run.tool_environment()` — agfront's shape, ported. `agentchat` goes on
PATH from the directory holding the interpreter the listener runs under
(`.venv/bin` under `uv`, so no deployment path is written down), and
`AGENTCHAT_ZULIP_ENV` names `agautolab/.local/zulip.env`. The supercoder
therefore speaks as `autolab-agstudio1` (user 11), which is what makes a
delegation attributable to the instance that made it rather than to a human.

The identity travels as a path, never as a value; the secret stays in
`.local/`.

`Bash(agentchat:*)` joins `WORKING_ALLOWED_TOOLS`. autolab's claude_code runs
pass `skip_permissions=True`, so the grant does not gate anything today — it
is there as documentation of what the role reaches for, which is what that
table is for.

## Time to wait

`WORK_TIMEOUT_SECONDS`: **1200 → 3600.** The p5 precedent, where
`FRONT_TIMEOUT_SECONDS` went 360 → 3600 for exactly this reason.

Both options in the plan were taken, not one. The ceiling is raised *and* the
standalone-task shape plus `agentchat read --since` remain the fallback: the
supercoder guide says to wait inside the run, and also says that a run which
ends mid-wait is recovered by reading the topic since the last message it saw.
The topic is the memory; the ceiling only buys the common case.

Forge's worst case is `360 + 900` for planning plus `1200` for execution —
2460 s — and a delegation is at least two of those exchanges plus the
requester's own thinking. 1200 could not have covered it; 3600 can, with the
`read --since` path for when it does not.

`SUPERDIRECTOR_TIMEOUT_SECONDS` and `DIRECTOR_TIMEOUT_SECONDS` were defined as
`WORK_TIMEOUT_SECONDS` and would have been dragged to 3600 by that edit. Both
are now pinned to 1200 explicitly: planning and brain-mining wait on nobody,
and the listener is serial, so an inflated ceiling on those is pure latency
for the next topic.

## The guides

**Superdirector** — a new "Work other agents can do" section: the
introductions file says who is there; anything a task needs that one of them
provides can be asked of them; **make the request its own task**, naming which
agent, what to ask for in plain words "enough that the agent can act on it
without reading this project", and what the task delivers back into its own
topic. One request per task, with the reason given (a task that waits is
doing nothing else while it waits).

**Supercoder** — a new "When the task is to ask another agent" section:
`agentchat` is the tool and `--help` is its documentation; asking is a
supervision, not a message; post where that agent's introduction says to; wait
**in this run**, blocking, because it can take many minutes; answer what it
asks — *you* are the one who decides on the project's behalf; **read its
introduction for how a request of that agent is actually finished, because
some need a further step from you before anything is produced**; recover with
`read --since` if the run ends mid-wait; bring the result back into this
topic; and the developer, not the delivery, is what closes the task.

Neither guide names agforge, a channel, or a topic prefix. That is criterion 3
enforced at the source: if the guides had to spell forge's flow, the harvest
would be decoration.

## Reading forge's introduction as its new reader

The plan asked whether forge's intro has a p5-shaped counterpart. It did, and
worse than autolab's had.

The whole of it was three sentences: what agforge makes, and "open an
`assetplan-…` topic in this instance's channel and describe what you want".
For a human who can look at the board and ask again tomorrow that is enough.
For a supercoder mid-task, with a wall clock running and no human beside it,
it omits everything that decides whether the request ever completes:

- that planning is a conversation, and forge will mention you and wait;
- that registering a Work **generates nothing** — the topic saying "registered
  PA-14" reads exactly like success and is not;
- that execution is a **separate `assetrun-…` trigger the requester fires**,
  which nothing in the old text hinted at;
- that the trigger takes *the next planned Work*, so two assets planned and
  two triggers fired back to back cannot be told apart — plan, trigger, wait,
  then plan the next;
- that delivery lands back in the **`assetplan-` topic**, not the `assetrun-`
  one where the trigger was posted;
- that the URL expires in about an hour while the `[S3KEY]` behind it does
  not, and re-signing beats re-requesting;
- roughly how long each stage takes.

The rewritten `params/intro.md` says all of it, in forge's own voice, and is
posted: `#agents` / `intro-agforge-agstudio1`, stamped `f154f49`. **No agforge
code changed** — criterion 4 holds, and an intro change is data by that
criterion's own terms.

The old marker route hid every one of those facts inside autolab's code. That
is the p6 thesis in one paragraph: the coupling was never really about
`asset_gate`, it was that forge's contract was written in autolab's source
instead of in forge's introduction.

autolab's own introduction gained one paragraph too — a task may be a
delegation, those run longest, and the topic still does not close until the
supervisor says so — so a human supervising a quiet `workrun-` topic knows
why it is quiet. Re-posted the same way.

## Deployment

- pyagag, agautolab, agfront, agforge: all pushed to GitHub.
- `uv sync` in agautolab and agfront; both already on pyagag `d3bd27a`.
- All three listeners kickstarted. autolab's startup line now reads
  `prefixes ('workplan-', 'workrun-', 'bmining-')` — `assetplan-` is gone from
  the sweep, which is Step 2 visible at runtime.
- Both introductions re-posted to `#agents`.

**The `--limit agstudio` playbook run failed, for an unrelated reason.**
`claude_code_agent` asserts a user-scoped Node at
`/Users/eiji/.local/node/bin/npm`; this Mac's npm is Homebrew's
(`/opt/homebrew/bin/npm`) and that path does not exist. The failure is in a
role that runs before `autolab_node` and has nothing to do with this episode
— it would have failed identically yesterday.

That leaves the live checkout honest by other means, which is what the
constraint actually wanted: the agstudio placement's checkout *is*
`pj-agdev/agautolab`, the working tree these commits were made in. It is at
`51301e7`, matching `origin/main`, its `.venv` is synced, and its listener was
restarted from it. Nothing is stale. agautolab1 was not deployed — the plan
made it optional, it runs the gateway and no listener, and the node baseline
gap above wants fixing on its own terms rather than as a side quest here.

## Tests

agautolab 147 → 152. The five added are the four board tests and the
handover's own set in `test_role_run.py` (PATH, identity-as-a-path, the
default identity, the handover reaching the harness, the grant).
