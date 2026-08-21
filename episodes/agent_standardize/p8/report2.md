# p8 step 2 — agforge: `assetrun-` the workrun way

AI-generated (Omni Agent, 2026-08-21). Commit `b6f6c8e` in `agforge`, pushed
to GitHub. pyagag re-locked `065de12` → `6d5853d`. Intro re-posted (message
868 in `#agents` > `intro-agforge-agstudio1`). Listener kickstarted and
sweeping as user 13 at 05:26Z.

## The topic is opened by the plan that owns it

`plane.register_plan` now returns a `Registration` — the sentence it always
returned, plus the Work's identity — because the caller needs to name the
Work again. `assetplan_topic.open_assetrun` uses it: when the generator
writes a `plan.md` and it is registered, agforge opens
`assetrun-<the same stem>` in the same channel and posts three things, in
order:

```
[selfnote][rootchat] <channel>/assetplan-<stem>
[selfnote][work] <project id>/<issue id>
This topic runs FF-12 "Draw the bird". Post here to start it, saying
anything you want done differently; the result is posted back here and in
assetplan-<stem>.
```

The first two are invisible everywhere (`agag.selfnote`), so a reader sees
one line. Because forge is its own last **real** speaker there, opening the
topic does not fire it — which is exactly the property step 1's
`last_real_sender` had to guarantee. It is idempotent on the work note: a
second generation of the same plan finds its topic already anchored and only
says where it is. `src/agforge/anchor.py` holds the format and the two
readers (`own_work`, `own_rootchat`), plus the one-stem-two-topics name rule.

The `assetplan-` reply now carries the run topic's name as a section, and
still mentions the requester through `serve_topic`'s ordinary handoff.

## A trigger is answered by reading the topic

`handle_assetrun` goes through `agag.topics.serve_topic` like every other
handler — ack, chatlog, always answer, a reply naming the last speaker, the
post-run re-check. Inside it:

- **Which Work**: `own_work(history)` → `works.work_by_id(project, issue)`.
  A topic with no work note is told so (`UNANCHORED_REPLY`) and runs nothing;
  a Work that has been deleted from Plane is named, not guessed around.
- **What to do**: the topic's conversation is written into the Work's
  workspace as `chatlog.md`, beside `plan.md`, and the generator's guide now
  says the last message is what the person who started it asked for *now* —
  follow it where it is more specific than the plan, ignore it where it says
  nothing.
- **Where the result goes**: both topics. The `assetrun-` one through the
  skeleton's reply, the `assetplan-` one through `deliver_to_origin`, whose
  destination is the root note (`work.origin()`, p1's external key, still
  answers for a Work planned before this phase). Both posts name whoever
  triggered the run.

## What was deleted

`works.next_work`, `eligible_works`, `project_marked`, `sub_work_serial` and
their tests — the whole selection policy. It existed because the topic said
nothing about itself; it is what made "one trigger, one Work — let the
delivery land before the next one" the requester's burden, so that sentence
is gone from the introduction too. The `FORGEAUTO` label still separates
agforge's Works from autolab's in Plane; nothing reads it to decide what to
run any more.

## Introduction, as re-posted

> **Making it is yours to trigger** — When I register the plan I open its run
> topic — `assetrun-<the same name>` in the same channel — and say so in the
> plan topic. Post there to start it, and say anything you want done
> differently this time; I read that post the way I read the plan. The topic
> knows which Work it runs, so there is no queue for you to keep track of.
>
> **What "done" looks like** — The result is posted into **both** topics …

## Tests

191 pass (`uv run pytest`), from 188. Rewritten: `test_works.py` around the
lookup instead of selection, `test_plane.py` around `Registration`,
`test_assetrun_topic.py` around an anchored topic. New: the two anchors and
the idempotence of `open_assetrun`, the chatlog reaching the generator, the
delivery reaching both topics, the root note outranking the external key, and
the unanchored/missing-Work replies.

## One consequence worth watching in step 4

Both posts name the trigger, so a Front that triggered a run is mentioned
**twice** — once in `assetrun-…`, once in `assetplan-…` — and its listener
serves each mention as a separate run. That does not loop (Front answers at
home, and nothing sweeps `front-`), but it may cost two replies to the
developer where one would do. Kept as the plan wrote it, and named here so
the proof either shows it or does not.
