# Step 2 — the three failure shapes, live

All of it in one public scratch topic, `#general > zulip-command-step1`,
commands posted from the Omni Agent account. Every message below is a real
Zulip post; nothing is reconstructed.

## The topic, as it happened

```
[17:02:25] Omni Agent (4464):
@**Comfy Notifier** watch ca370ad6-… step1 hand-posted proof
[17:02:27] Comfy Notifier (4465):
comfy success ca370ad6 in 0s — 1 outputs · step1 hand-posted proof
prompt_id `ca370ad6-…` — read `GET /history/<prompt_id>` for outputs

[17:04:00] Omni Agent (4466):
@**Comfy Notifier** watch not-a-real-prompt-id step2 nonsense id
[17:04:03] Omni Agent (4467):
@**Comfy Notifier** please keep an eye on my render
[17:04:05] Comfy Notifier (4468):
comfy command not understood: post `@**Comfy Notifier** watch <prompt_id> [note]` in a public-channel topic
[17:04:16] Comfy Notifier (4469):
comfy lost not-a-re in 11s — not in queue or history — ComfyUI probably restarted · step2 nonsense id
prompt_id `not-a-real-prompt-id` — read `GET /history/<prompt_id>` for outputs

[17:04:49] Omni Agent (4470):
@**Comfy Notifier** watch step2-unreachable-probe step2 comfy unreachable
[17:06:59] Comfy Notifier (4471):
comfy unreachable step2-un in 124s — ComfyUI unreachable: <urlopen error [Errno 61] Connection refused> · step2 comfy unreachable
prompt_id `step2-unreachable-probe` — read `GET /history/<prompt_id>` for outputs

[17:07:22] Omni Agent (4472):
@**Comfy Notifier** watch 0b977567-… step2 restart mid-wait
[17:09:25] Comfy Notifier (4473):
comfy unreachable 0b977567 in 121s — ComfyUI unreachable: <urlopen error [Errno 61] Connection refused> · step2 restart mid-wait
prompt_id `0b977567-…` — read `GET /history/<prompt_id>` for outputs

[17:09:47] Omni Agent (4474):
@**Comfy Notifier** watch adfb8d57-… step2 restart mid-wait, take two
[17:10:32] Comfy Notifier (4475):
comfy success adfb8d57 in 41s — 4 outputs · step2 restart mid-wait, take two
prompt_id `adfb8d57-…` — read `GET /history/<prompt_id>` for outputs
```

## The three shapes

- **Nonsense `prompt_id` → `lost`.** Three polls, 11 s, one post. Exactly the
  path a run gets when it mistypes an id or ComfyUI has restarted under it.
- **ComfyUI unreachable → `unreachable`.** It *was* cheap to arrange after
  all, so it was not skipped: the installed plist's `AGFORGE_COMFYUI_URL` was
  pointed at `http://127.0.0.1:9` (a closed port), the daemon reloaded, the
  command posted, and 124 s later — the real `COMFYNOTIFY_UNREACHABLE_S`
  threshold of 120, not a shortened one — the callback named the connection
  refusal. The URL was then restored.
- **Malformed command → one line back.** `@**Comfy Notifier** please keep an
  eye on my render` produced 4468 and nothing else: no ticket on disk, and
  **no reaction on 4467**, which is the visible difference between "I did not
  understand you" and "I am watching this".

## The ack rule, measured

Reactions on the six commands, read back from Zulip:

| message | 4464 | 4466 | 4467 | 4470 | 4472 | 4474 |
|---|---|---|---|---|---|---|
| reaction | 👀 by 21 | 👀 by 21 | *none* | 👀 by 21 | 👀 by 21 | 👀 by 21 |

User 21 is the Comfy Notifier bot. Every valid command got exactly one
reaction and exactly one message — its terminal callback. The only bot post
that was not a callback is the error line. Nothing in this topic could have
served a run early.

## Restart mid-wait

`adfb8d57-…` (1024², 40 steps, batch 4 — a 41 s job) was submitted, commanded
at 17:09:47, and the daemon was restarted while it was still in
`queue_running`. On the way back up the log printed `command intake on as
Comfy Notifier (21)` and **no `command mark seeded` line and no re-processing
of 4464/4466/4467/4470/4472** — the mark was read from disk at 4474. The
still-open ticket was picked up by the new process and posted once, at
17:10:32. Across three daemon restarts in this topic: six commands, six
reactions, five callbacks, zero duplicates.

## The one thing that went wrong, and it is worth keeping

Message 4473 was not planned. The first attempt at the mid-wait restart
watched a job that really did finish `success`, and the callback said
`unreachable`. The cause was not the notifier:

> **`launchctl kickstart -k` does not re-read the plist.** It restarts the
> job launchd has already loaded, so the `AGFORGE_COMFYUI_URL` that had just
> been restored on disk was silently ignored and the fresh process kept
> polling the dead `127.0.0.1:9` from the unreachable test. The ticket written
> in that window recorded the dead host, and its callback was correct about
> what the daemon could see.

Reloading needs `launchctl bootout` followed by `bootstrap` — and `bootout`
is asynchronous, so bootstrapping immediately after it fails with
`Bootstrap failed: 5: Input/output error`; wait a few seconds and retry.
`launchctl print gui/$(id -u)/<label> | grep AGFORGE_COMFYUI_URL` is the
check that the value is really loaded. This is now written into
`.local/devenv.md`, where the notifier's reload instruction used to say
`kickstart -k` and nothing else. The old instruction was not wrong for
restarting the process; it was wrong for changing its environment, and this
episode is the first time this project changed a running job's env.

Every callback in this step was truthful about the world the daemon was in.
The failure was in how the world was changed, and it is exactly the kind of
thing the three-shapes exercise exists to catch.

Done: each shape produced exactly one visible outcome, the log tells the
story, and the posts are transcribed above as they appeared.
