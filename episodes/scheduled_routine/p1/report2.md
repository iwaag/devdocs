# scheduled_routine p1 — Step 2 report: trigger

## Script

`pj-agdev/devenv/routine/trigger.sh <name>` (commit in pj-agdev, see git
log). It resolves everything relative to its own location — no absolute
paths — and does one thing: `agentchat send front front-routine-<name>
"<text>"` using `agfront/.venv/bin/agentchat` and
`.local/zulip/developer.env`. `AGENTCHAT_HOME` is unset on purpose so no
`[rootchat]` selfnote is written (a scheduler has no home conversation).

Posted text:

> Routine `imgprompt`, run of 2026-08-23T15:46Z. The standing request is the
> latest post in #front › `routine-imgprompt`; this topic holds the earlier
> runs and my comments on them. Do it.

## Identity

The **Developer's own account** (`developer.env`). Reasons: zero
provisioning; "Front replies to the Developer" stays literally true; no bot
listener exists that could sweep `front-` by accident. The cost is that the
log cannot tell a scheduled fire from a human post except by the message
text — acceptable for p1, and the `routine-<name>.log` (Step 3) is the
scheduler's own record anyway.

## Topic shape

One topic per routine, runs appended: `front-routine-imgprompt`. Chosen
because the standing text tells Front to read earlier runs and the
Developer's comments on them — one topic is that history with no lookup.
If it gets too long to read, a dated topic per run is the fallback
(noted in plan, not needed yet).

## First manual run

`trigger.sh imgprompt` → `sent message 1446 to #front > front-routine-imgprompt`.
`agfront/.local/out/zulip-listener.log` within a second:

```
2026-08-23T15:46:39Z serving 'front'/'front-routine-imgprompt'
2026-08-23T15:46:39Z front topic 'front'/'front-routine-imgprompt'
```

No change to Front, pyagag, forge or autolab was needed. What Front did with
the run is Step 4's material.
