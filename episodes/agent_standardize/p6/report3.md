# agent_standardize p6 — Step 3 report: what a delegating run is given

AI-generated (Omni Agent, 2026-08-21).

agautolab `65850ef`, agforge `f154f49` (text only). Step 2 removed the route;
this is what replaces it, and none of it names agforge.

## The board reaches both runs

`write_agents_md` from `agag.intro` (Step 1) is now called in two places in
`zulip_listener.py`: in `serve` before the superdirector run, and in
`serve_run` before the supercoder run. Both write `tools/agents.md` into that
serving's own generation workspace, and both prompts name the file by
absolute path beside the chatlog. `context.step = "harvest"`, so a board read
that fails is reported into the topic as "failed during harvest" like any
other step.

Both roles need it and for different reasons: the superdirector to know
whom a delegation could go to while it is writing the plan, the supercoder to
know how to reach them while it is carrying one out. Nothing is cached — the
harvest happens per serving, which is what lets an agent that changed its
entrance this morning be reachable this afternoon with no deploy.

## The runs can speak

`role_run.py` gained `tool_environment()`, lifted in shape from agfront's:

- `PATH` gets the directory holding the interpreter the listener runs under —
  `.venv/bin` in a `uv` project, where the `agentchat` console script is
  installed. No deployment path is written down anywhere.
- `AGENTCHAT_ZULIP_ENV` names `agautolab/.local/zulip.env`, so a delegating
  run speaks as `autolab-agstudio1`. The identity travels as a *path*; the
  secret stays in `.local/`.

It is applied in `run_role` to every role, not only the supercoder — one seam
rather than a per-role table that would drift. `Bash(agentchat:*)` was added
to `WORKING_ALLOWED_TOOLS` as documentation of what a role reaches for; the
runs use `skip_permissions` under `claude_code`, so it is not what makes the
command work.

## Timeouts

`WORK_TIMEOUT_SECONDS`: **1200 → 3600**, the p5 precedent
(`FRONT_TIMEOUT_SECONDS` 360 → 3600 for the same reason). forge's own path is
`360 + 900 + 1200` s worst case, and a delegating supercoder spends nearly
all of its run waiting for that.

Both options in the plan were taken, not one: the ceiling is raised **and**
the standalone-task shape plus `agentchat read --since` remains the recovery
path. The ceiling reduces how often resumption is needed; it cannot remove
the need, because the supervisor on the other side may be slower than any
ceiling. Real wall-clock numbers are Step 4's to report.

`SUPERDIRECTOR_TIMEOUT_SECONDS` and `DIRECTOR_TIMEOUT_SECONDS` were defined
as `WORK_TIMEOUT_SECONDS` and would have been dragged to 3600 with it. Both
now say `1200` explicitly, with the reason: planning and brain-mining wait on
nobody, and a planning run that hangs should fail in twenty minutes, not in
an hour.

## The two guide lines

`workplan_superdirector/guide.md` — after the task-splitting instructions:

> Other agents exist, and the file of introductions this prompt names above
> is the list of them […] If a part of the mission is that agent's work
> rather than this project's coding work, delegate it: write into the task
> which agent to ask, what to ask them for, and what has to come back.
>
> Make each delegation its own task. A task that waits on another agent
> should wait alone, so a task that needs what comes back is the next task
> and reads the result from the delegation task's topic.

`workrun_supercoder/guide.md` — before the work instructions:

> If the task says to ask another agent for something, that is the work of
> this task. […] Read the one you need and follow it; nothing about that
> agent is written down here.
>
> Talk to them with `agentchat` (`agentchat --help` explains it). Delegating
> is a supervision, not a message: post the request, wait for the reply in
> the same run, answer what they ask, and keep going until they say the
> request is complete. Waiting is `agentchat wait`, in this run — do not end
> the run and hope somebody picks it up.

Both say *what an agent is for* and *where to read about it*, never who is
there. "Waiting happens inside the run" is p5's lesson repeated verbatim in a
new place, because it was learned expensively there.

## forge's introduction, read as its new reader

The plan asked for this to be checked, and the check found four gaps. Its old
introduction was four sentences: what forge makes, and "open an `assetplan-…`
topic". A supercoder following only that would have planned an asset and then
waited forever, because:

1. **A registered plan does nothing until it is fired.** Generation happens
   only when something is posted into an `assetrun-…` topic. Nothing in the
   old intro said the word.
2. **The button takes the next unfinished planned Work**, not necessarily
   yours — `works.next_work()` sorts by creation time across every
   `FORGEAUTO` project. So the discipline is plan, trigger, *wait for the
   delivery*, then plan the next.
3. **The delivery lands in the `assetplan-` topic, not the `assetrun-` one**
   (`deliver_to_origin`). A requester watching the button would see the
   summary and never the asset.
4. **What "done" is**: a post carrying a URL and an `[S3KEY] <key>` last
   line, the URL good for about an hour and the key good indefinitely
   through `POST /api/resign`. The old intro never mentioned either.

The rewritten introduction says all four, plus the fact that forge does not
resolve the requester's topic and does not need to be told the requester is
finished — which is the *opposite* of autolab's contract, and exactly the
kind of thing that must be posted rather than assumed. Re-posted to `#agents`
at revision `f154f49`.

Criterion 4 holds: `git log` shows one agforge commit and it touches
`params/intro.md` only.

autolab's own introduction was re-posted too (`65850ef`), with one added
paragraph: a task may be a delegation, such tasks run longest, and the
close-out contract is unchanged. A supervisor who does not know that reads a
quiet twenty minutes as a stall.

## Tests

143 → 156. The new ones:

- the planning run's workspace gets `tools/agents.md` and the prompt names it
- the same for the task run
- an empty board still lets a run happen (the honest sentence, not a failure)
- the board is re-harvested per serving — an entrance changed between two
  servings is visible in the second
- `agentchat` is first on the run's `PATH`
- the identity is a path, not a value, and defaults to `.local/zulip.env`
- every launched run carries the handover
- the supercoder's grant lists `Bash(agentchat:*)`

The fake Zulip clients in `test_zulip_listener.py` gained a `board` dict and
serve `#agents` reads without recording them, so every pre-existing
call-sequence assertion still means what it meant.

## Deployment

- Pushed: agautolab `65850ef`, agforge `f154f49`.
- Both introductions re-posted to `#agents`.
- All three listeners restarted (`launchctl kickstart -k`). autolab's startup
  line now reads `prefixes ('workplan-', 'workrun-', 'bmining-')` — the
  `assetplan-` sweep is gone from the running process, not only from the
  source.

**The `--limit agstudio` playbook run failed, and did not need to succeed.**
It stops in the `claude_code_agent` role, which asserts a user-scoped npm at
`/Users/eiji/.local/node/bin/npm`; this Mac's node is Homebrew's at
`/opt/homebrew/bin/npm`. That is a pre-existing mismatch in the agstudio
placement, not something this phase introduced, and the role that would
matter here never runs. It also does not affect the constraint it was there
to satisfy: the inventory gives agstudio
`autolab_node_repo_dest: …/pj-agdev/agautolab`, which *is* the live working
tree, already at `65850ef` and pushed. The live checkout is honest.

agautolab1 was not deployed — optional per the plan, it runs no listener, and
nothing in this phase changes the gateway.

Handoff candidate (Deus Ex Machina note): re-posting both introductions was
done by the Omni Agent. Each agent has an `intro` module of its own and could
post after its own behavior change; nothing does it automatically today.
