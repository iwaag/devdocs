# agforge follow-up — the assetrun can use the notifier now

`report4.md` found that agforge's `assetrun` could not act on the notifier
paragraph its guide carried, and could not act on the older CLI version
either. `todo_done.md` recorded the choice: give the generator what it needs,
or take the paragraph away. The Developer chose to give it what it needs.

## What was added

Three verbs, all behind `agforge`, which `[roles.generator]` already grants:

```
agforge video submit --prompt "…"    queue it, print the prompt_id, return
agforge music submit --prompt "…"    the same, for music
agforge comfy fetch <prompt_id> --into result    download a finished job
```

`comfy_async.py` splits the existing generators along the seam `comfy_video`
already had — `submit()` on one side, the wait and the download on the other
— and adds no new workflow. `fetch` never polls: by the time anything calls
it the notifier has already said the job reached a terminal state, and a
fetch that quietly blocked would put the waiting back inside the run that
just avoided it. The three pointless fetches (still queued, job errored, no
output files) each say so in one line.

**Image is deliberately not included.** It goes through SwarmUI, which has no
ComfyUI `prompt_id` to give, and `agforge image generate` returns in about ten
seconds — measured, not assumed, while writing this. It is not the blocking
problem.

## What was *not* added: a Zulip voice for the generator

The generator still has no `Bash(agentchat:*)` and no credentials. Giving it
one would have been the obvious reading of "let it post the mention", and it
would have broken the line forge is built on — the generator makes files, the
front talks — to buy a general outbound voice for one sentence.

Instead the hand-off is a file, and the **listener** does the talking:

1. The run calls `submit`, writes `{"prompt_id", "note"}` into `pending.json`,
   and finishes. It never waits.
2. `assetrun_topic.serve` sees `pending.json` and — this is the whole trick —
   makes the notifier command *its own reply*: no extra post, no extra API
   call. It renames the file to `watching.json`, delivers nothing, and leaves
   the Work open.
3. The notifier's callback is itself a post in that topic, so it triggers the
   next run, which reads the id from `watching.json`, runs `comfy fetch`,
   deletes the file, and finishes normally. That run is the one that delivers
   and closes the Work.

One property worth naming: because the command is only posted *after* the run
ends, the callback can never arrive mid-run. autolab's shape can — task 1 of
`report4.md` did exactly that.

## Two defects the live run found

Neither was visible in the unit tests, and both are the sort of thing only a
real run produces.

**The command must be read from its line, not from the message.** The
listener's reply is a short report whose *second* line is the watch line. The
notifier parsed the whole message and read the first word — `running` — and
refused it. Fixed in `comfynotify`: every line that mentions the bot is tried
in turn, the first that parses is the command, and the note stops at the end
of that line. Without this the whole hand-off is inert.

**A notifier callback is not a requester.** The collecting run is woken by
the notifier's post, so the last voice in the topic is a bot — and forge
reads "who triggered this run" from the conversation. It delivered my music
to `@**Comfy Notifier**`, and I was never named or served. This is
`report4.md`'s misdirection defect again, now inside forge and caused by the
very mechanism this change adds. Fixed where the fact is lost: `watching.json`
carries the trigger across the gap, read before the generator runs (the guide
has that run delete the file), and the delivery prefers it. The module's own
doctrine already said *"the trigger is a fact about the past, so it is read
from the past"* — the past is now two runs ago.

## Live proof

Three attempts in `agforge-agstudio1 > assetrun-async-music`, and the first
two are worth keeping because each shows the failure the fix removes.

1. **18:04** — I forgot to restart the listener, so the *pre-change* path ran:
   `result/ is empty; delivering the answer text` and `work F2-27: Done yes`.
   The Work was closed with nothing in it. That is precisely what `pending.json`
   exists to prevent, demonstrated by accident.
2. **18:08** — the hand-off worked end to end (queue → watch line → callback in
   35 s → collect → deliver), but the delivery named `@**Comfy Notifier**`.
3. **18:12** — with the trigger remembered:

| time | what |
|---|---|
| 18:12:20 | trigger posted |
| 18:13:13 | run queues `822320ee`, writes `pending.json`, ends (52 s, 7 turns) |
| 18:13:13 | the listener's reply carries `@**Comfy Notifier** watch 822320ee … calm piano loop`; Work left open |
| 18:13:46 | `comfy success 822320ee in 31s — 1 outputs · calm piano loop` |
| 18:13:47 | that post triggers the collecting run |
| 18:14:17 | `result/ holds 1 file(s); zipped and uploaded`; `work F2-27: Done yes` |
| 18:14:16 | delivered to the plan topic naming **`@**Omni Agent**`** |

`watching.json` at the moment of the hand-off:

```json
{"note": "calm piano loop",
 "prompt_id": "822320ee-e7bb-40d4-89b2-cd1c8a877b6c",
 "trigger": "@**Omni Agent**"}
```

The workspace afterwards holds `result/ComfyUI_00007.mp3` and no
`watching.json` — the collecting run deleted it, as its guide says.

Notably, forge worked out the new path from its **toolset document alone**:
its plan reply said *"`agforge music submit` queues the job and returns a
`prompt_id` immediately, which gets written into `pending.json` per the async
guide's convention"*. No hand-holding in the request.

## Tests and cost

`agforge`: 211 pass (8 new for the async verbs, 5 for the hand-off and the
remembered requester). Three pre-existing failures were fixed on the way:
`tool_environment` had gained `COMFYNOTIFY_BIN` as a hard-coded third
directory, so the tests that pass absent directories to prove "nothing is
added" had been failing on any machine with a real comfynotify venv; it is
now overridable like the other two. The third was a stale argv expectation
after pyagag's agcode grew `--max-tokens`.

`comfynotify`: 24 pass (2 new for line-wise parsing).

8 agforge runs, **$0.69**, including the two failed attempts. Four ComfyUI
music jobs, ~30 s each.

## What this leaves

`comfynotify`'s CLI is still not in the generator's grant, and does not need
to be — nothing in this path runs it. The remaining open item from
`report4.md` is unchanged and is not forge-specific: **nothing in the system
notices that a conversation has stalled or become a loop.** Both defects here
were found by watching, not by any agent reporting them.
