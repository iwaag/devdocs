# refactor p1 step 2 — replacement through conversation operations

## What the step asked

Support retiring an old plan/task conversation and opening its replacement;
make one such workflow usable by the agent and record what it carries
forward; keep conversation identity separate from its reusable display name;
and make retired work leave the execution queue. Verify with focused tests:
a reused topic name, a late callback to the old work, restart recovery
without duplicate serving, and a deleted origin.

Identity was already separated from the display name — step 1 had to do it
early, because nothing else could carry the record. This step is what that
separation was for.

## What changed

### Revision has two moves now, and they are different sizes

Writing `plan.md` again **revises** a mission in place: task files match by
serial, a completed task stays completed, and the conversation carries on.
That was step 1's model and it is unchanged.

`replace.flag` beside `plan.md` is the other move, for when the request
itself was wrong: **retire this conversation and open its replacement.**
Four things happen, in this order, and the order *is* the workflow
(`worklog.replace_mission`):

1. every unfinished task is cancelled and resolved, and the finished ones are
   read out to be carried forward;
2. the mission is marked `replaced` and its `work-m<id>` channel archived;
3. its conversation is renamed to `✔ retired-<name>-m<id>` — out of the
   `workplan-` vocabulary **and** resolved;
4. the replacement is opened under the name step 3 released.

Step 4 cannot precede step 3. Zulip has one topic per name in a channel, so a
replacement created first would not stand beside the old conversation — it
would **merge into it**. `tests/realm.py` models that, which is why the test
suite would notice if the order were ever reversed.

A replacement with no `plan.md` is refused (`NO_REPLACEMENT_PLAN`): it would
retire the request and put nothing back.

### Retirement is a rename plus a resolve, and both are load-bearing

The plan's hint was that "a renamed topic is not necessarily retired:
listeners may still match its prefix or own its whole channel", and that the
retirement operation must be made observable to the listener. It is made
observable twice over:

| operation | what it takes away |
|---|---|
| rename out of `workplan-`/`workrun-` | no topic filter matches the name any more |
| resolve (`✔ `) | `sweep_topics` skips the topic without reading it |
| archive `work-m<id>` | the channel leaves every subscription, so its task topics are never walked |
| cancel each unfinished task | still true if the archive fails |

Two of those are belt and two are braces, on purpose: the coarse operation
(archive) removes a whole mission's queue presence in one call, and the fine
one (cancel) is what remains correct when the coarse one does not take.

### The link back is an id

`[selfnote][replaces] <anchor id>` in the replacement's conversation, and an
id rather than a name for a reason that is the point of the whole step: **the
replacement is wearing the retired conversation's name.** A name-shaped
pointer would resolve to the replacement itself.

`Mission.replaces` and `Task.replaces` carry it; `worklog.predecessor`
follows it. A rerun topic — the surface for a task changed after it was
completed — carries the same note naming the completed task it reworks, so
the finished record and its rework read as one chain rather than two topics
that happen to share a serial.

What the replacement **carries forward** is a visible post, because it is
what a human reads to understand why the topic they were writing in became a
different mission: the finished tasks by reference (`work-m5512/workrun-…`),
the dropped ones by name, and the reason the superdirector wrote into
`replace.flag`. Finished work is never re-created as a completed task row in
the new mission — the new mission starts with no tasks at all, and the plan
that follows is expected not to re-ask for what is listed there.

### One accident the state note prevents

`mission_done` gained a fourth reason a mission may not be closed. Retiring
cancels the unfinished tasks, and `mission_tasks` filters cancelled tasks
out — so what is left of a replaced mission is exactly the tasks that *did*
finish. A replaced mission looks, on the numbers alone, like a mission whose
every task completed. The `[selfnote][state] replaced` note is what says
otherwise; the retired topic's name is never consulted, because a name is not
a fact about work.

### Shared change (pyagag), one function

`ZulipClient.rename_topic(message_id, new_name)` — the general form of what
`resolve_topic` always was, which now calls it. Nothing else in the shared
layer changed: the `[rootchat]` note keeps its channel/topic text, and no
legacy-format adapter was needed.

## The four verification cases

`agautolab/tests/test_replacement.py`, against `tests/realm.py` — which since
this step also renames topics and archives channels the way Zulip does.

**A reused topic name.** After a replacement, reading `pj-demo/workplan-ship-it`
gives the new mission; the old one is still itself, with its plan and its
`replaced` state, found through its anchor under the name it was moved to.

**A late callback to the old work.** A delegation names its home as
channel/topic text, so the question is whether that text still finds the task
it was written for. It does, and by construction rather than by luck: a task
topic's name is minted from its mission's **anchor id**, so the replacement's
tasks live in another channel entirely and no later work can want the name.
Retirement of a task therefore resolves the topic rather than renaming it,
and `topic_history_across_resolve` follows the `✔ ` — so the late answer
lands on the cancelled task, where `run_target` refuses to run anything.

**Restart recovery without duplicate serving.** Run through the real
`agag.zulip.sweep_topics`, with a human post in every topic that still
exists — the only way a topic can await a reply at all. The sweep returns the
replacement's conversation and nothing of the retired one's, and the archived
work channel is absent from the subscriptions the sweep walks.

**A deleted origin.** Deleting the retired mission's anchor makes
`predecessor` answer `None` — not the replacement, which is the thing now
standing where the retired conversation stood.

Two further tests run the real handler end to end: `replace.flag` beside a
plan retires the mission, records the plan into the replacement's
conversation, opens the replacement's own task surfaces in its own channel,
and names the requester where the replacement opens; `replace.flag` without a
plan changes nothing at all.

## Evidence

```
$ cd pyagag && uv run pytest -q
543 passed in 68.74s

$ cd pj-agdev/agautolab && uv run pytest -q
235 passed in 0.95s
```

pyagag 543 (was 542, +1 for `rename_topic`); agautolab 235 (was 224, +11).
`uv.lock` was re-pinned to pyagag `218cb31`.

## What a run already under way does

Nothing. Retiring does not interrupt anything: the listener serves one topic
at a time, so a replacement decided while a task is running is applied after
that run returns, and a run already under way finishes against the work it
started on. That is the plan's second option ("let it finish against the old
work"), taken deliberately; forced interruption is out of scope.

## Guides and instructions

`agent/guides/workplan_superdirector/guide.md` gains a section separating
adjustment from replacement, saying that replacing needs a plan in the same
run and that already-finished work must not be re-asked for.
`params/intro.md` says the same thing to a human requester: what changing a
plan does, what scrapping one does, and where the retired conversation went.

## Limitations carried into the next steps

- **Step 3 is still untouched.** `agdevworld/agentroom` discovers autolab work
  through `PlaneBoard`/`PlaneReader`, so a replaced mission and its
  replacement are invisible to the operation room, and `accepted` is still a
  state nothing writes. The `replaces` relation exists precisely so the
  traversal there has something to walk.
- **One replacement route, at the mission level.** Replacing a single task
  in place has no separate route: the rerun topic covers the case where a
  completed task's text changed, and anything else is a re-plan. The plan
  permits this ("one supported replacement route is enough").
- **The retired conversation's name is not recycled twice.** A second
  replacement of the same request would produce
  `✔ retired-workplan-ship-it-m5602` beside `✔ retired-workplan-ship-it-m5512`
  — distinct because the label is an anchor id, but the channel accumulates
  retired topics. Nothing prunes them.
- **Nothing has been deployed or run live.** No listener has been restarted
  on this code and no real mission has been replaced through it; that is
  step 4.
- **Still not Plane removal.** Unchanged from step 1: `agag.plane` stays for
  forge and cagent, and `nctl`/Nautobot Plane identities are a later phase.
