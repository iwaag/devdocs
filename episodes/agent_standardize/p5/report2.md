# agent_standardize p5 — Step 2 report: making a long run legal

AI-generated (Omni Agent, 2026-08-21).

Three things had to change before Front could be asked to supervise: a budget
that survives somebody else's work, an instruction to stay, and a board that
says what staying is for.

## The timeout, measured first

Front's budget was `FRONT_TIMEOUT_SECONDS = 360`, written when Front "reads a
conversation, reads the board, and posts a message or two". The realistic leg
it now has to outlive is autolab's own, and the run records say what that is:

```text
role            n    min      med      max     (seconds, .local/agent/*/run-*.json)
run            22    5.5    105.6   1140.0
supercoder     15    9.9     22.8    157.3
superdirector  18    9.0     24.1     83.7
director       12   13.8     34.4     77.6
front          23    8.0     12.9     42.2
```

`run` reaching **1140 s** against autolab's own `WORK_TIMEOUT_SECONDS = 1200`
settles it: one stretch of work can occupy nearly twenty minutes, and a
supervised request is several of those in a row — autolab's listener is
serial, so the three tasks queue behind each other. 360 s could not span even
one leg.

`FRONT_TIMEOUT_SECONDS = 3600`. The comment carries the reasoning and the
price: agfront's listener is serial too, so a run this long is also how long
the Developer's next `front-*` post waits to be served. That is the trade —
a supervision that survives the task, against a front that answers while it
is busy. Front's own runs stay ~13 s median; what is long here is waiting,
not working.

## The guide: two sentences, no vocabulary

```text
Some requests are not finished when you have sent them. If the developer asked
you to see something through, stay with it: wait for the other agent, answer
what it asks you, and when the work really looks done, tell it so - some agents
wait for that before they finish. Only then report back.
If you run out of time before it ends, say honestly where it stands and which
message you last saw, so the next reply can carry on from there.
```

"Some agents wait for that" is deliberate. The guide must not learn that
**autolab** wants an agreement post in a `workrun-` topic — that is the
knowledge p2 and p4 insist travels by reading the board. What the guide adds
is only the disposition: don't treat a sent request as a finished one, and
fail out loud rather than silently. The second sentence is the fallback shape
the plan allows, written where Front will actually read it.

## The board: where the close-out contract belongs

The plan named only the guide, but the guide could not carry this: autolab's
introduction never said that **its** close-out waits for the requester.
`serve_run` writes the report, marks the Plane issue Done and resolves the
topic only if the run produced `report.md`, and `workrun_supercoder/guide.md`
produces it only "if the developer agreed that the task was done". That is a
real contract that existed nowhere a reader could find it. A supervisor could
have waited forever beside an agent that was also waiting.

So `agautolab/params/intro.md` gained a section — *"While a task runs,
somebody has to be there"*: progress arrives as posts, questions are answered
in the same topic, **the task does not close until you say it is done**, and
tasks run in order (`Please complete previous work` is a queue, not a
failure). Re-posted with `uv run python -m agautolab.intro` → message
**725** in `#agents` > `intro-autolab-agstudio1`.

That placement is the episode's own rule, not a workaround: p4's README_DEV
line says the `#agents` post *is* the contract, and `agents_md.py` harvests
the newest post of each intro topic immediately before every Front run — so
Front's next run reads 725 with no deploy, and nothing about autolab was
compiled into agfront.

## Deployed

```text
$ launchctl kickstart -k gui/$(id -u)/com.agdev.agfront-zulip
2026-08-20T17:04:25Z agfront zulip listener starting (pull sweep, prefix 'front-')
2026-08-20T17:04:25Z sweeping as user_id=15 (front-bot@agstudio.local)
2026-08-20T17:04:25Z full sweep: 0 awaiting, 19 calls spent, 988 left in the window
```

The autolab listener (`com.agdev.agautolab-zulip`) was left running: the intro
is data it posts, not code it reads. The three `workrun-task{1,2,3}-s2-6`
topics in `#work-s2-6` are still exactly as p4 left them — autolab's own
posts 717/718/719, nobody else's.

## Commits

- **agfront** `7d308c5` — timeout + guide. Tests **31 passed**.
- **agautolab** `cd54dad` — the introduction's supervision section.
- **pj-agdev** — submodule pointers.

All pushed to GitHub.

## What is untested until step 3

Whether a claude_code run actually sits for an hour is not something a config
value proves. The measurement the braindump asks for happens next, and either
answer is the deliverable.
