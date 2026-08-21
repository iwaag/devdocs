# p10 Step 4 — Prove and report

## 1. Fixtures

The first pass used p9's own residue, which still showed exactly the state
the plan wanted: forge had four delivered plans with both topics unresolved,
and autolab had two finished missions whose `workplan-` topics were open and
whose mission Works sat in `unstarted` — `S2-30` among them, p9's open item.

The Front pass then needed fresh ones (§4), so two trivial fixtures were
seeded exactly as the plan allows: one forge plan (a 64x64 pixel-art star
icon → `F2-21`) and one autolab mission (`workplan-p10-text-file` in
`pj-runsmoke1` → Work `R-4`, one Sub-Work `R-5`, writing `main/text.md`).
Both were driven to completion and left un-closed.

## 2. Omni pass

`p10-status-check`, one topic in each entrance channel.

### forge — the survey

> Four plans on record, all delivered and marked Done in FreeForge:
> F2-20 Stage-1 BGM Loop · F2-17 Enemy Sprite · F2-16 Pixel-Art Treasure
> Chest Icon · F2-15 64x64 Pixel-Art Green Potion Bottle Icon. Plus one
> earlier plan (paper-airplane-icon) already marked finished (✔). Nothing
> currently in progress or awaiting action.

Correct and complete against the real topic list, and it read the `✔` rename
as "somebody closed this out" without being told which topics those were.

### autolab — the survey

It named both open missions with their projects, said which tasks were done
and which had not started, quoted the two commits the finished mission
landed, and volunteered the contract:

> This is finished but hasn't been closed out yet — I haven't resolved
> anything or run `mission_done` on my own initiative, since that's only done
> when asked.

### The close-out

forge resolved six topics — three plan/run pairs plus the standalone
`assetplan-treasure-chest-icon` — and deliberately left `how-to-request`
alone as "not asset work".

autolab resolved `workplan-p9-assets-4` and `workplan-enemy-spawn-patterns`
and ran `mission_done`, which moved **S2-30** and **S2-6** to Done. Verified
independently: `mission_done --dry-run S2-30` now answers `already Done`.
**p9's open item is closed, and it was closed by the agent that owned it.**
`workplan-shield-pickup-icon` was left open, correctly.

## 3. Two defects the live runs found

### (a) forge answered twice

The first forge run wrote its survey into the topic with `agentchat send`,
and then the skeleton posted the run's closing message as the reply. The
guide's last line already said not to; it was not enough. One line, both
guides:

> Your reply is the last thing you say in this run, and it is posted into
> this topic for you. Never `agentchat send` into this channel — doing that
> posts your answer twice.

Guides are read from disk per run, so no restart. Every serving since has
posted once.

### (b) autolab surveyed one project and reported on all of them

Asked by Front where its plans stood, autolab answered:

> I have two plans, both for the `simpleshooter` project …

It had missed `pj-runsmoke1` entirely — the one project holding a finished,
un-closed mission. Its wording ("since our last check-in", "still stuck at
the same point as before") says where the answer came from: its own earlier
reply, still readable in its channel. Front then concluded, from a sound
reading of a wrong premise, that autolab had nothing to close.

**The guide caused it.** The line

> Look up only what was asked; a survey of everything is slow and usually not
> the question

was written to stop needless reading. For "where do all your plans stand",
every project *is* what was asked. It now bounds depth rather than breadth,
and two lines say the rest:

> Asked where your plans stand, list **every** `pj-` channel and look in
> each: a project you did not look at is one you cannot report on.
>
> Start from the channel list every time. Your own earlier answers in this
> channel are history — they say what was true when you wrote them, not what
> is true now.

forge got the history line too; it is the same trap, and its channel list is
one command.

### (c) …and the reason (b) took a fixture to notice

There was no record of what a run looked at. The entrance kept a cost report
and an answer, and **an answer that skipped a project reads exactly like one
that found nothing in it.**

Both entrance servings now write `transcript.jsonl` beside the chatlog. That
needed a pyagag change to be worth anything: streaming was reachable only
through `on_event`, the live-progress seam, so a caller wanting the *record*
rather than the progress got `-p`'s single result document — turn counts and
a dollar figure, and not one word about the run. `run_harness(stream=True)`
(pyagag `1db9150`) asks for the stream with nobody watching, and
`transcript_path` then captures the whole thing.

Verified on a live entrance question afterwards — 67 lines, 15 tool calls,
in order:

```
agentchat channels --prefix pj-
agentchat topics pj-foodchain … pj-runsmoke1 … pj-runsmoke2 … pj-simpleshooter
agentchat channels --prefix work-
agentchat read pj-foodchain mission_base1
… 9 more reads, narrowing to what the question needed
```

That is the behaviour the guide fix asked for, and it is now visible rather
than inferred.

## 4. Front pass

**Attempt 1** — "Check both agents' work status and clean up anything
finished." Front found both entrances from the introductions alone and read
both `p10-status-check` conversations, then reported, accurately:

> Someone (an earlier "Omni Agent" run) already did this exact task moments
> ago. … Since both cleanups already happened, I didn't re-post.

Good behaviour, spoiled experiment: **the Omni pass had left the answer lying
in the very channels Front was told to consult.** It proves the routing half
of criterion 3 and nothing about the instruction half. The Omni topics were
resolved and the two fresh fixtures seeded.

**Attempt 2** — Front opened `p10-status-check-2` in each entrance channel,
each with its `[selfnote][rootchat]` note, and asked. forge answered naming
Front; Front was called back, told forge to close out; forge resolved
`assetplan-p10-star-icon` and `assetrun-p10-star-icon` itself. autolab
answered with defect (b) — Front, reasoning correctly from it, sent no
close-out request, and `R-4` stayed open. **This is the round that caught the
bug, and it caught it because Front reported faithfully what it was told.**

**Attempt 3**, against the fixed guide — Front opened `p10-status-check-3`,
autolab answered:

> **pj-runsmoke1** — `workplan-p10-text-file` (R-4) was actually finished …
> but the topic and mission were still open. I just resolved the topic and
> ran `mission_done`, marking R-4 Done in Plane.
> … Only one item needed closing, and that's now done.

with the other three projects each named and correctly triaged. Verified
independently:

```
$ agentchat topics pj-runsmoke1
✔ workplan-p10-text-file
$ python -m agautolab.mission_done --dry-run R-4
R-4 not moved: already Done                     (exit 0)
```

Front then reported it to the Developer in its own `front-*` topic, adding
the two items needing the Developer rather than an agent.

**Zero Omni posts after the fixtures were seeded.** Everything from
`front-p10-cleanup-2` onward was Developer → Front → entrance → Front →
Developer.

## 5. Attributability

`agfront/src` and `agfront/agent/guides`, grepped for both entrance channel
names and every topic prefix of both agents:

| term | `agfront/src` | its guide |
|---|---|---|
| `agforge-agstudio1`, `autolab-agstudio1` | 0 | 0 |
| `workplan-`, `workrun-`, `assetrun-`, `pj-` | 0 | 0 |
| `assetplan-` | 1 | 0 |

The single hit is a comment in `zulip_listener.py` explaining why the sweep
filter is `front-` only ("the other side of the same asymmetry is agforge,
which sweeps its own `assetplan-` topics and never `front-`"). No code reads
it and no prompt carries it. Front reached both entrances three times over
knowing nothing but `#agents`.

## 6. Costs

| run | agent | wall | cost | turns |
|---|---|---|---|---|
| survey (Omni) | agforge | 26 s | $0.2143 | 13 |
| survey (Omni) | agautolab | 62 s | $0.4191 | 18 |
| close-out (Omni) | agforge | 33 s | $0.2840 | 12 |
| close-out (Omni) | agautolab | 36 s | $0.2312 | 10 |
| survey (Front) | agforge | 17 s | $0.1256 | 4 |
| close-out (Front) | agforge | 11 s | $0.1323 | 5 |
| survey (Front, defect b) | agautolab | 50 s | $0.3246 | 18 |
| survey+close-out (Front, fixed) | agautolab | 220 s | $0.4755 | 20 |
| transcript check | agautolab | 50 s | $0.3142 | 16 |

Entrance runs: **$2.52** for nine questions. Front's own six runs across the
three attempts: **$1.65**. Every role on `sonnet`, every run recording it.

Two things the numbers say. The whole-board sweep is the expensive one —
220 s and 20 turns against forge's 11 s and 5 turns, because forge's board is
one channel and autolab's is one channel per project. And the cheap runs are
the ones answering a *narrow* question: Front's "what have you got" cost
forge $0.13. Scan cost at tens of projects is real and is carried into the
deferred list.

## 7. Success criteria

1. **forge** — met. Survey from a live scan of its own topics; "resolve the
   finished ones" verified and resolved them, twice, under both drivers.
2. **autolab** — met, after (b). The same survey across every project, and
   the close-out pass resolves the finished topics *and* closes the mission
   Work — `S2-30` under Omni, `R-4` under Front. p9's gap is closed by
   construction and demonstrated twice.
3. **agfront** — met on the third attempt. Front reaches both entrances from
   the introductions alone, instructs them, and the entrances do the work.
   Attempt 2 is kept in this report because it is the honest record: Front
   was right, its source was wrong, and that is how (b) surfaced.
4. **Terse guides** — met. 8 body lines each, same register as forge's
   existing ones.

## 8. Deferred candidates

- **Proactive tidying.** Not this phase, deliberately. The entrance closes
  what it is told to close; a cron-like sweep would be the entrance deciding
  somebody else's conversation is over.
- **Scan cost at tens of projects.** autolab's full sweep is already 20
  turns at four projects. `agentchat channels --prefix work-` gives the
  mission list in one call; what costs is `topics` per channel and `read`
  per topic. A "what changed since <message id>" shape would bound it.
- **Humans vs agents at the entrance.** forge's Front-driven answers were a
  third the price of its Omni-driven ones for the same board, because Front
  asked a narrower question. Nothing in the entrance distinguishes the two
  callers today, and nothing yet says it should.
- **An agent cannot fix its own channel description.** The forge channel
  still advertised the retired `create-` prefix; neither the forge bot nor
  the Omni Agent may administer the channel (HTTP 400), only the developer
  account. *Did X for agent Y — handoff candidate.*
- **Front replies once per callback.** Two entrances answering one request
  gave the Developer three near-identical reports in one topic. That is p8's
  design working, not a p10 defect, but it reads as noise.
- **`brandump.md`** is still spelled that way in this phase's folder.
