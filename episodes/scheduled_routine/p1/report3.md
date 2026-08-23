# scheduled_routine p1 — Step 3 report: schedule

## plist

`pj-agdev/devenv/launchd/com.agdev.routine-imgprompt.plist.in`
(pj-agdev commit "hourly launchd job for routine imgprompt"). Same
`__PROJECTS_ROOT__` convention as the other services:

- `ProgramArguments`: `/bin/sh <root>/pj-agdev/devenv/routine/trigger.sh imgprompt`
- `StartCalendarInterval` `Minute=0` → hourly, on the hour. `RunAtLoad`
  false, so bootstrapping does not fire an extra run.
- stdout/stderr → `agfront/.local/out/routine-imgprompt.log` — the only
  record of a fire that failed to post.

Installed: `sed` the template into `~/Library/LaunchAgents/`, `plutil -lint`,
`launchctl bootstrap gui/$(id -u) …` (15:48Z). Stop = `launchctl bootout
gui/$(id -u)/com.agdev.routine-imgprompt`. No lock, by design.

## First scheduled fire

`routine-imgprompt.log`:

```
2026-08-23T16:00Z trigger imgprompt -> #front/front-routine-imgprompt
sent message 1489 to #front > front-routine-imgprompt
```

`zulip-listener.log`:

```
2026-08-23T16:00:03Z serving 'front'/'front-routine-imgprompt'
2026-08-23T16:00:03Z front topic 'front'/'front-routine-imgprompt'
```

Three seconds from calendar tick to Front run.

## Overlap observation (first data point)

The 15:46Z manual run was still open (its forge Work F2-23 had failed twice
on an unreachable image backend, see report4) when the 16:00Z fire landed
in the same topic. Front did **not** start a second theme: it read the topic,
recognised the stuck run, and re-triggered F2-23 instead, saying it would
open a fresh theme only if that also failed. Appending runs to one topic
therefore gave Front the context to make a sane overlap decision without a
lock. Whether this holds when the earlier run is mid-generation rather than
failed is still to be seen (Step 5).
