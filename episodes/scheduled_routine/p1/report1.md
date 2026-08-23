# scheduled_routine p1 — Step 1 report: routine definition lives in Zulip

## Where

`#front` › `routine-imgprompt` (message 1445, posted 2026-08-23 15:45 UTC by
the Developer account via `agentchat send` with `developer.env`).

The recommended home was taken as is: chat, in Front's own channel, under a
topic **without** the `front-` prefix. Verified before choosing:

- Front's listener reports `pull sweep: all topics in 'front-agstudio1',
  prefixes ('front-',) elsewhere`. There is no channel `front-agstudio1`
  (`agentchat topics front-agstudio1` → HTTP 400), so in practice only
  `front-`-prefixed topics are served. `#front` already holds an unserved
  unprefixed topic (`agentchat-wait-check`), which shows the same thing.
- After the post the listener log shows no sweep or serving of
  `routine-imgprompt`; its last lines are still the 13:17Z startup.

Plane and a repo file were not used: the routine is meant to be edited where
the Developer already reads everything else.

## How it is edited

- Edit the post in Zulip, or append a new post to the topic. The trigger
  message (Step 2) says "the standing request is the **latest** post in
  `#front › routine-imgprompt`", so appending supersedes; earlier posts
  remain as history of how the routine changed.
- Nothing parses the text. It is a request in the Developer's voice, read by
  Front as chat.

## The definition as posted

> **Routine `imgprompt` — standing request** (the latest post in this topic is the current definition; earlier posts are history)
>
> Pick a visual theme for today. Vary it from previous runs — read the earlier runs in the run topic (`front-routine-imgprompt`) before choosing. Ask forge for 4 images of that theme, each with a clearly different prompt (different composition, mood, or technique — not four near-duplicates). Give forge the full spec up front in one `assetplan-` post so it does not have to ask: 1024x1024 PNG, one image per prompt, style as you choose for the theme; then trigger the run(s) yourself.
>
> Post in the run topic, for each image: the prompt, the download URL and the `[S3KEY]`. Then read my comments on the earlier runs in that topic and say, in two lines, what you would try next and why.
>
> Do not score the images yourself — I will, in plain text in the run topic. Do not wait for me to answer anything in this routine: if something is unclear, choose sensibly, say what you chose, and carry on.

Two additions to the plan's suggested text, both pre-empting hints the plan
itself gives: the forge spec (size/format/count) is stated so forge's
"what did you leave open" reply is not needed, and Front is told not to wait
on the Developer, since nobody is at the keyboard when a scheduled run
fires. Whether Front still asks anyway is a Step 4 observation.
