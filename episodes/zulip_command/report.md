# zulip_command — Report

The desire (`braindump.md`): stop handing `comfynotify` to every agent that
might need it, and let any agent get a ComfyUI job watched by posting one
Zulip line. And: see whether that line survives being relayed through Front.

Both hold. The line is

```
@**Comfy Notifier** watch <prompt_id> [free text kept as the note]
```

posted as a normal message in any public-channel topic. The notifier tickets
the job with that topic as the callback destination, acknowledges with a 👀
reaction, and posts its two-line terminal record there when the job ends. No
LLM anywhere; the daemon is the same 5 s sweep it always was, with a second
sweep beside it.

Step reports: `report1.md` (intake), `report2.md` (the three failure shapes),
`report3.md` (guides and docs), `report4.md` (the relay).

## Does one Zulip line replace the tool handover?

Yes, and the evidence is the run's own words rather than mine. A mediagen
supercoder run, told only through two relay hops what to post, wrote in its
report: *"Posted directly into this task's own topic via `agentchat send`, as
a normal chat message (no code fence) … The `comfynotify` CLI was not run …
No polling or sleeping was done."* It submitted to ComfyUI itself, posted one
line, and finished. Ninety seconds later the notifier's callback brought it
back.

Both guides now teach the line and neither mentions the CLI. The CLI still
exists as the daemon's test surface and the no-Zulip path; nothing is handed
to anybody.

## The ack rule, as observed

The prohibition was: never acknowledge a valid command with a message post,
because a bot post in a run topic re-serves that topic's owner and burns a
paid run. Held throughout. Across every command in this episode, each valid
one carried exactly one `eyes` reaction from the notifier bot and exactly one
message — its terminal callback. Reactions raise no message event and no
mention; nothing was ever woken by an acknowledgement.

The one deliberate exception, the "command not understood" post, turned out
to be the most expensive line in the episode, and it proves the rule rather
than weakening it:

- It reached `#front`, which is **not private** — `invite_only: False`, a
  public channel restricted by convention — so the Developer's own request to
  Front was read as a command and answered in Front's entrance.
- That post re-served Front. Front answered the last speaker, which was now
  the notifier, and named it. That fired again. One Sonnet run per lap.
- Stopped by hand after two laps, then fixed properly: **a topic is told once
  and never again**. Watched working in the same session on three different
  topics.
- The residual cost is not the post but the misdirection: Front, forge and
  autolab each answered the notifier instead of their real requester, and once
  that stalled the mission until a human nudge.

If this mechanism grows a second command, the lesson is already paid for: a
bot that speaks in an agent's topic is not a bystander, it is a participant,
and every line it says costs somebody a turn.

## Did the command survive Front?

Yes — verbatim on both hops, into forge's channel and into mediagen's. None
of the three predicted paraphrase failures occurred: the mention was not
turned into prose, not fenced by a relaying agent, and Front spontaneously did
the right thing in its recaps to the Developer by quoting it inside backticks,
which fires nothing. The braindump's worry about a long game of telephone was
not borne out; what broke was elsewhere.

Two things did break, both now fixed:

1. The refusal loop above, from `#front` being public.
2. **The callback did not follow a resolved topic.** A run that ends by
   closing its own topic renames it while its render continues; the ticket
   remembered the open name and the callback opened an empty topic beside the
   real conversation. The daemon now resolves the live name immediately
   before posting (pyagag's `live_topic_name`), proven live.

One vehicle turned out to be no vehicle: agforge's assetrun **cannot** use
this or the old CLI, because its only image tool is synchronous and returns a
URL rather than a `prompt_id`, and the run cannot post to Zulip. That
paragraph has never been actionable in an assetrun. Worth its own decision
later — either give forge an async submit and a chat post, or take the
paragraph out of its guide.

## What remains of the PATH problem

For consumers of the notifier: **nothing. It is moot, and this is the
episode's quiet win.** A Zulip post needs no binary on `PATH` and no
`AGFORGE_COMFYUI_URL` in the run's environment; the command carries a
`prompt_id` and the daemon supplies the rest from its own launchd env. The
guides no longer name a binary, so no run can fail for not finding one.

Whether the `extra_environment` mystery is still worth diagnosing: **almost
certainly not on its own account.** Both supercoder runs in step 4 reported
`AGFORGE_COMFYUI_URL` present as `http://agpc.local:8188` — the very variable
p6 ex1 measured as unset — so whatever ate it was not a stable property of the
harness, and the one consumer that cared no longer cares. Diagnose it only if
something else starts losing an environment variable; then it is a real bug
with a second witness, not a ghost.

## Costs

- Steps 1–3: no agent runs. Development, unit tests, hand-posted commands.
- Step 4: 15 agent runs, **$2.58** — 9 Front ($1.30), 3 autolab superdirector
  ($0.47), 2 supercoder ($0.61). Roughly $0.55 of that exists only because of
  the refusal-post loop and its misdirection.
- ComfyUI: eight small jobs across all steps, all terminal, none kept.

## Deus Ex Machina

Four, all recorded where they happened:

- **The loop was stopped by hand.** The notifier daemon was booted out mid-
  experiment when it and Front started answering each other. No agent could
  have noticed; the cost was accruing in a channel neither of them reads as a
  budget. An outsider with `launchctl` was the only available brake.
- **A nudge restarted a stalled mission.** Autolab addressed its plan reply to
  the notifier instead of Front, so Front was never called back and the
  mission stopped with nobody waiting on anybody. One Developer post
  (message 4518) restarted it. Without that it would have sat there
  indefinitely — there is no watchdog for "an agent answered the wrong
  participant".
- **A stale-env ticket was deleted by hand** in step 2, after
  `launchctl kickstart -k` was found not to re-read the plist.
- **The whole of steps 1–3 is Omni Agent work** on a tool the in-system agents
  use but cannot change.

The first two are the interesting ones, and they are the same shape: the
system has no way to notice that a conversation has become a loop, or that it
has quietly stopped. Both were visible to a human in seconds and invisible to
every agent involved.

## Out of scope, still out

No second command was built (`status`, `cancel`, `free`); the grammar leaves
room. No multi-backend URL routing. No entrance, introduction, or `#agents`
topic for the Comfy Notifier — it is a tool, and the moment it needs an
introduction it has become an agent.
