# Step 4 — prove it through the relay

The braindump's experiment: does one Zulip line survive Front's game of
telephone. It does, twice, unaltered — and getting there cost two design
defects that only a live relay could have found.

## The chain

Developer → `#front` > `front-comfy-command-relay` → Front → an agent's own
channel → that agent's run → the notifier → back into the run topic → the run
served again. The literal handed to Front was:

```
@**Comfy Notifier** watch <prompt_id> apple relay test
```

with the instruction to relay it *verbatim, unfenced, without backticks*.

## First vehicle: agforge, and why it could not be one

The plan allowed an `assetrun-` on agforge as the cheap stand-in. It is not a
possible vehicle at all. Forge said so itself (`assetplan-red-apple`, message
4495, plus an `idea.md` naming the tool additions it would need), and reading
`agents.toml` and `role_run.py` afterwards confirms it and sharpens it — the
run's own account was right about the conclusion and vague about the cause:

- **It cannot post to Zulip. This is the hard blocker and it is deliberate.**
  `[roles.generator].allowed_tools` has no `Bash(agentchat:*)` — only
  `[roles.front]` does — and `assetrun_topic.run_generator` calls `run_role`
  without `home=`, so the run gets no `AGENTCHAT_ZULIP_ENV` and no `agentchat`
  on PATH. That is forge's whole delivery model: the generator makes files
  under `result/`, the front does the talking. A mechanism whose interface is
  "post a line" is unreachable from the half of forge that generates.
- **It cannot obtain a `prompt_id` from what it is told about.**
  `agforge image generate` is synchronous and returns a time-limited download
  URL as its last line; the `toolset-image` document never mentions ComfyUI at
  all. And `AGFORGE_COMFYUI_URL` is deliberately withheld:
  `role_run.tool_environment` hands the run exactly **one** allowlisted value
  (`ACE_STUDIO_CLI`) plus PATH, and the URL stays in `agforge/.local/.env`
  where only agforge's own CLI code reads it. The run does hold
  `Bash(curl:*)` and `Bash(python3:*)`, so this half is a *knowledge* barrier
  rather than a permission one — it could `POST /prompt` if it were ever told
  where ComfyUI is.

And a third thing, which is the one worth carrying forward: **the old CLI
paragraph was equally un-runnable in forge, for a different reason again.**
`tool_environment` prepends `comfynotify/.venv/bin` to the generator's PATH,
so unlike autolab's runs forge's runs really do see the binary — but
`Bash(comfynotify:*)` is not in the generator's grant, so claude_code would
have refused it. Autolab could not find the binary; forge could find it and
was not allowed to run it. Two guides carried one paragraph that neither run
could execute, for two unrelated reasons, and nobody noticed until a run was
asked to actually do it.

The request was withdrawn and the mission moved to autolab, which submits
ComfyUI graphs itself and whose supercoder run is given `home=` and therefore
`agentchat`.

## The relay, measured

Front relayed the block **verbatim on both hops** — into
`agforge-agstudio1 > assetplan-red-apple` (4488) and into
`pj-mediagen > workplan-comfy-command-relay` (4504) — and autolab passed it
into the task file from there. None of the three predicted paraphrase
failures happened:

- the mention was never "corrected" into prose;
- it was never wrapped in a code fence by a relaying agent;
- in Front's *recaps to the Developer* it was quoted inside backticks, which
  is exactly right and is why those recaps fired nothing.

The run's own report is the cleanest evidence of what arrived:

> Posted directly into this task's own topic via `agentchat send`, as a normal
> chat message (no code fence): `` @**Comfy Notifier** watch
> b09133ad-… slow relay test ``. Sent as message id 4547. The `comfynotify`
> CLI was not run. … No polling or sleeping was done after submission or
> after posting the mention.

Both runs also recorded `AGFORGE_COMFYUI_URL` as present
(`http://agpc.local:8188`) — the env that `advance_mediagen_study` p6 ex1
measured as unset inside a supercoder run. Whatever that was, the mention
route does not depend on it.

## Task 1 — the round trip

| time | what |
|---|---|
| 17:21:51 | Front posts `Start it` in `work-m-51 > workrun-task1-m-51` |
| 17:23:07 | the run posts `@**Comfy Notifier** watch 5d2301a1-… apple relay test` (4524), acked 👀 |
| 17:23:08 | notifier posts `comfy success 5d2301a1 in 0s — 1 outputs · apple relay test` (4525) |
| 17:23:23 | autolab served again in that topic |

One second between command and callback — because the 512², 8-step job had
already finished. That is my test design's fault, not the mechanism's: the
callback arrived *inside* the run rather than after it, so this leg does not
prove the second serving. Hence task 2.

## Task 2 — the callback that arrives after the run has ended

| time | what |
|---|---|
| 17:26:23 | Front posts `Start it.` |
| 17:27:24 | the run posts `@**Comfy Notifier** watch b09133ad-… slow relay test` (4547), acked 👀 |
| 17:27:37 | the run reports to `@**Front**`, resolves its topic, **ends** |
| 17:28:54 | job ends; notifier posts `comfy success b09133ad in 88s — 8 outputs · slow relay test` (4554) |
| 17:28:54 | **autolab is served again by that post** |

**Job end → callback → second serving is one sweep.** Between the command
(17:27:24) and the callback (17:28:54) the topic contains nothing but the run
itself: no human post, no supervisor post, exactly as the experiment
required. The run posted the command and finished; the notifier brought it
back. That is the whole claim, and it holds.

## What the relay broke, and what it cost

### 1. `#front` is not private, and the refusal post is a loop engine

The plan assumed `#front` was out of reach. It is `invite_only: False` — a
public channel restricted by convention — so the Developer's *request to
Front*, which quoted the command line live, was read by the notifier as a
command aimed at it, refused, and answered in Front's own entrance topic.
That post re-served Front; Front, answering the last speaker, addressed
`@**Comfy Notifier**`; that fired again. One paid Sonnet run per lap.

The daemon was stopped by hand after two laps and taught the rule it now has:
**a topic is told once that it is not understood, and then never again**. The
fix was watched working within the same episode — Front, forge and autolab
each named the notifier again afterwards and each got `already told …, staying
quiet`. The loop cannot run any more; its worst case is one extra post and one
extra serving per topic, ever.

The same mechanism has a second cost that survives the fix: **the refusal
misdirects the reply**. Front, forge and autolab each answered the notifier
instead of their real requester, because a Zulip agent answers whoever spoke
last. Once, that broke the chain outright — autolab's plan reply named the
notifier, Front was never called back, and the mission stalled until the
Developer nudged it (message 4518, the one Deus Ex Machina of this step).

### 2. The callback did not follow a resolved topic

Task 2's run closed its own topic at 17:27:37, renaming it to
`✔ workrun-task2-m-51`, while its job still had a minute to run. The ticket
remembered the open name, so the callback at 17:28:54 opened a **fresh, empty
`workrun-task2-m-51`** beside the real conversation. Autolab was served — the
mechanism worked — but into a topic with no task in it, and answered "this
topic is not bound to any task".

The plan's own facts section had already said the intake should follow the
rename "the way `agentchat` does"; I implemented that for reading commands and
not for posting callbacks. Fixed: the daemon now resolves the topic's live
name through pyagag's `live_topic_name` immediately before posting, with a
unit test and a live proof — a job was commanded in `general >
zulip-command-rename`, the topic was resolved while it rendered, and the
callback landed in `✔ zulip-command-rename` (message 4562).

This is not a relay artefact. Any run that finishes by closing its own topic
— which is the normal, encouraged ending — renames the conversation while its
render continues.

## Cost

15 agent runs, **$2.58**, from the first Front post to the last supervision:
9 Front runs ($1.30), 3 autolab superdirector runs ($0.47), 2 supercoder runs
($0.61). Of those, three Front runs and one superdirector run exist only
because of the refusal-post loop and its misdirection — roughly $0.55, or a
fifth of the step, spent on a defect the step was designed to find.
ComfyUI: four small jobs, all `success`, no image kept.

Done: one image-producing job has round-tripped agent → mention → callback →
second serving with no human post between the command and the callback, and
the timeline above is the record.

## Postscript — the mission was closed, and one tidy-up misfired

Front relayed the close-out and autolab answered: *"M-51 is closed out: both
run topics were already resolved from the work itself, and `mission_done` now
marks M-51 Done in Plane."*

The two empty `workrun-…-m-51` topics that the pre-fix callback created are
still there. Trying to tidy them was a mistake worth recording: posting a
"stray topic, nothing here" note into each one **woke autolab twice** (the
same re-serving rule this report is about, applied to me), and `agentchat
resolve` then refused with *"is already resolved"* — because a `✔ <topic>`
of that name exists, resolve treats the conversation as closed even though
the stray open topic beside it is a different one. Two runs spent making
nothing better. They were left alone.

Two lessons, both cheap to state and apparently easy to forget: a human
posting into an agent's topic pays the same price a bot does, and `agentchat
resolve` cannot close a stray topic that shadows a resolved one.
