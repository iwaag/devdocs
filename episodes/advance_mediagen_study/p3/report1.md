# Step 1 — Fire, handoff, install, resume

Fired 2026-08-30 09:49Z into `#front › front-routine-mediagen` as the
Developer, carrying the subject inside the firing post — p1 recorded that
you cannot post context and then fire, because **any** post in a `front-`
topic *is* the fire. One post, all the context.

The fire named the subject the plan named — *skeletal animation of an
AI-generated pixel-art character, proven by a rig that actually moves in
Godot or Phaser* — and carried the plan's hints as hints: do not redo p1's
survey, the rig step is text, Godot is not installed and the Developer will
install it, sequence that request **before** parts generation, parts
candidates already on the box, judge twice, reuse p1's requirement so the
two methods compare cell to cell, new subject → new repository.

## What the routine did with it

Front opened `#pj-mediagen › workplan-skeletal-rig-proof` (3292) three
minutes later, carrying the fire's context and the backend literals
verbatim. autolab planned **M-14** with four sub-works:

| task | what |
|---|---|
| 1 | File the Godot host-install request **first**; gather concrete `Skeleton2D`/headless-render facts; stand up `gentest-skeletalRig/` |
| 2 | Generate and judge the AI parts (does not need Godot) |
| 3 | Build the scripted rig and render it |
| 4 | Judge the rendered animation, distil, write up |

The install-before-generation sequencing the fire asked for is exactly what
the plan came back with, and Task 1 filed the request as its first act
rather than its last.

Two things the run decided for itself, both asked for and neither dictated:

- **Godot, not Phaser** — a `Skeleton2D` scene is a `.tscn` a script can
  write directly, where Phaser's only skeletal path is the Spine plugin and
  a Spine export we would have to fabricate first. The fire said "either is
  fine, say which and why"; it said which and why.
- **A correction to the fire's own text.** The fire asserted that Godot
  "runs headless (`godot --headless`) and can render an animation to frames
  with its movie writer". Task 1 came back saying those two do not compose —
  `--headless` disables the rendering server, so the movie writer has
  nothing to write — and that the real invocation is `--write-movie` without
  `--headless`. That is the run contradicting the Developer's hint from
  documentation, before a task could trip on it.

## The handoff, as received

Task 1 finished in `waiting_external` with `gentest-skeletalRig/` committed
and its `localtest.yaml` in that state, and Front relayed the request at
09:58Z:

> **Action needed from you: please run `brew install --cask godot` on this
> Mac.** … Nothing in the mission needs it yet — Task 2 (AI parts
> generation) doesn't touch Godot — but Task 3 … does, so the sooner this
> lands the more it overlaps with Task 2 instead of blocking after it.

Nine minutes from fire to a correctly-sequenced, correctly-argued install
request. This is the part of the standing text that p1 wrote and never got
to exercise: p1's chosen method needed no host change at all.

## The install

Performed immediately, as the plan said the Developer would:

```
$ brew install --cask godot
🍺  godot was successfully installed!
$ godot --version
4.7.2.stable.official.ed1daf0bf
```

`godot` is on PATH as a Homebrew-cask symlink into `Godot.app`. Nothing else
was installed; no service and no port were opened, so this is not a
cagent/nintent conversation — a point p1's plan drew and this install
confirms from the other side.

Reported back into the same `front-` topic (3326) with the version string as
the read-only proof the standing text asks a handoff record to name, plus
the rollback line, plus two notes for Task 3: **re-confirm the
`--headless`/`--write-movie` finding against the real binary** rather than
carrying Task 1's reasoning forward, and a restatement of what is actually
under test — *a `.tscn` written as text by a run, that then moves*.

## The resume

Front relayed the install into the workplan topic (3332) rather than into
Task 2's run topic, because a post in a `workrun-` topic starts a turn and
Task 2 was mid-run. Asked to relay without interrupting, it found the right
place on its own.

Task 2 then produced the parts — and here the mission stalled silently for
26 minutes with nothing running and nobody aware. That failure and its
recovery are the substance of this step and are written up in `report.md`;
in short, Task 2's topic was served twice concurrently, the first serving
resolved it, the second recreated the unresolved name as an **empty
duplicate topic**, and Front's callback lookup found the decoy, saw no
`[selfnote][rootchat]` in it, and dropped autolab's completion post with a
single line in its log. A Developer post at 10:35Z supplied the explanation
— deliberately information and not action, as in p1 — and Front restarted
Task 3 itself, carrying the Godot confirmation and both notes forward.

So the resume worked in the sense the plan meant it: a later serving read
the persisted state, observed the installed binary with ordinary read tools,
and continued without the Developer restating the plan. What it did not do
was get there unaided.
