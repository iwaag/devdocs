# p9 step 4 — the proof run

AI-generated (Omni Agent, 2026-08-21). Steps
[1](report1.md) · [2](report2.md) · [3](report3.md).

**Four attempts.** The p9 mechanism itself — anchored `workrun-` topics,
callbacks read off the chat, replies at home, served markers, restart safety
— worked on every one of them, including the three that failed. What stopped
those three was around it, and running the mission is what found each one:
five pre-existing defects, all fixed, all with a regression test. The fourth
attempt met every criterion.

## Before any attempt: cancelling p6

The plan's own first step. Posted in `pj-simpleshooter/workplan-p6-assets`;
autolab created `cancel.flag`, cancelled S2-10 with all three sub-works
(S2-11/12/13), archived `work-s2-10` and resolved the topic (messages 898–901,
07:03:51 → 07:04:07). FreeForge **F2-17 was already Cancelled** — nothing to
do by hand.

What the plan's cleanup did *not* cover, and should have: the p6 topics in
**forge's** channel, `assetplan-simpleshooter-enemy-sprite` and
`assetrun-simpleshooter-enemy-sprite`. They cost attempt 1 (below) and then,
after I resolved rather than deleted them, they cost attempt 3 four extra
runs. Resolving is the wrong tool here: a resolved `assetrun-` topic whose
plan is still registered retires that plan permanently, because
`open_assetrun` is idempotent by the work note and will not reopen it. The
plan said the residue "may be deleted first"; it should have said deleted,
and it should have named forge's channel.

## Attempt 1 — the exchange that stopped with nobody named

One developer post (902, 07:06:15). Front took "see the whole thing through"
literally, skipped its proposal step and drove the mission itself.

What worked, first time:

- **Anchoring.** autolab opened three `workrun-` topics, each with its
  `[rootchat]` and `[work]` notes *before* the visible task description
  (908–916). The description stayed the last real post, so opening a topic
  fired nothing.
- **Callback at home.** `mention in 'pj-simpleshooter'/'workplan-p9-assets'
  serves front/front-20260821-p9-assets` — Front read its home off the chat
  and answered there.
- **The served marker, live** (923, 934, 943), written into home each time.
- **Criterion 3's first restart.** The autolab listener was restarted at
  07:09:41, while forge was generating. `full sweep: 0 awaiting, 1
  mentioning` — and the one mention was a p1-era topic in an archived
  project, dropped as "carries no root note of ours". **Nothing was resumed
  that should not have been**, which is precisely what step 1 was built for.

Then it stalled, and the cause was a real defect. The supercoder went to
forge; forge's assetplan front had a *question* rather than a spec, so no
generator run followed, so `serve_topic` had an empty body and posted no
final reply — and the final reply is the only post that carries
`@**<requester>**`. Forge asked a routing question (941) that named nobody.
autolab was never brought back. Three agents idle, nothing wrong, nothing
moving.

Fixed in agforge `7e1749b`: when no generator run follows, the front's answer
**is** the reply and the skeleton names the requester; when a spec follows,
the answer is posted early as before and the registration is the naming post.
One name either way.

## Attempt 2 — the supervisor who thought it had already started

Two developer posts: the request (950) and the go (953). Front proposed going
direct to forge and committing the files itself, which it has no tools to do;
the go permitted the work and said so, without naming an agent for it. Front
read the board and routed to autolab on its own.

autolab planned two tasks (not three — its own call, folding integration into
each), anchored both, and Front approved the plan. Then autolab's own
covering note said "both tasks will now proceed in parallel through their
`workrun-…` topics", and Front believed it: it waited, reported "no progress
yet" twice, and the mission stopped with two correctly anchored, never
triggered topics.

The mechanism was not at fault — a `workrun-` topic *does* wait for a post,
and autolab's introduction says so. What was missing is that the
deterministic handler text never said it where the supervisor reads it: it
said `opened work-s2-24/workrun-task1-s2-24` and stopped. agforge learned the
same lesson in p8 and says "posting in `assetrun-…` starts it".

Fixed in agautolab `69be43e`, in both places a supervisor looks:

- `opened <channel>/<topic>; post there to start it`
- `mission <label> is now In Progress; each task waits for a post in its own
  `workrun-…` topic, and nothing runs until somebody makes it`

## What the mechanism did, on every attempt

None of the three failures was the p9 mechanism. It behaved identically and
correctly each time, which is the thing the phase set out to establish.

**A `workrun-` topic says what it is for.** Every task topic autolab opened
carried its two notes before the visible description — for example
`work-s2-30/workrun-task1-s2-30`:

```
1078  [selfnote][rootchat] pj-simpleshooter/workplan-p9-assets-4
1079  [selfnote][work] 2fdbd0ac-92e7-4b75-96f2-1d854e2f4081
1080  # Ask agforge-agstudio1 to plan and generate the enemy sprite …
```

The description is the last real post, so opening a topic fired nothing and
each topic sat waiting for a trigger. Every serving then read its project off
the root note's channel and its task off the work note. The `work-` channel
description was written and never read.

**A callback is read off the chat and answered at home.** Both directions,
repeatedly:

```
mention in 'pj-simpleshooter'/'workplan-p9-assets-4' serves front/front-20260821-p9-assets-4
mention in 'agforge-agstudio1'/'assetplan-simpleshooter-enemy-sprite' serves work-s2-30/workrun-task1-s2-30
```

No ledger file exists any more, and nothing a run wrote to disk was consulted
to resolve either of those.

**A served callback is marked.** Both agents, throughout — Front into its
`front-*` conversation, autolab into the `workrun-` topic:

```
1116  [selfnote][served] agforge-agstudio1/assetplan-simpleshooter-enemy-sprite 1109
1126  [selfnote][served] agforge-agstudio1/assetplan-simpleshooter-enemy-sprite 1120
```

**One naming per delivery** (criterion 2). The delivery named the trigger and
the run report named nobody:

```
1120  agforge-agstudio1/assetplan-…  @**autolab-agstudio1**  result of "Plan: Enemy Sprite (simpleshooter)" …
1122  agforge-agstudio1/assetrun-…   running "Plan: Enemy Sprite (simpleshooter)" … delivered to …
```

autolab was served exactly once by that delivery, and the supercoder ran once
against the repository.

**Recovery is safe** (criterion 3, first half). The autolab listener was
restarted at 07:09:41 while forge was mid-generation:

```
07:09:41Z agautolab zulip listener starting …
07:09:42Z full sweep: 0 awaiting, 1 mentioning, 34 calls spent, 972 left in the window
07:09:42Z serving mention in 'pj-assetpipe1'/'create-asset_…'
07:09:42Z mention in … carries no root note of ours; ignoring
```

Zero runs resumed. Without step 1's served marker this sweep would have
re-served every anchored topic whose last post named autolab — which, since
the reply now goes home, is every exchange it has ever had.

## Attempt 3 — a delivered asset that reached nobody

Two developer posts. Front routed to autolab, autolab planned two tasks, and
**Front triggered both** — the `69be43e` wording landed. The plan's own gate
worked too: task 2 answered "Please complete previous work (S2-28)".

The delegation then hit p6 residue I had *resolved* rather than deleted, and
the two agents argued about who reopens a run topic for a Cancelled Work.
Front broke the deadlock itself (1018: "that pairing is stuck — start a fresh
topic"), the supercoder opened `assetplan-…-v2`, forge registered F2-19,
generated the sprite and delivered it. Zero human posts through all of that.

The delivery named **Front**, not autolab. `deliver_to_origin` found "the
trigger" with `handoff_mention`, which re-reads the run topic — and Front had
posted "how is it going?" into it mid-generation, becoming the last speaker.
Front had no root note in the *plan* topic, so its own callback was dropped
too, and a finished sprite sat in a topic nobody was waiting on.

Two fixes came out of it:

- **agforge `dabea97`** — `trigger_mention` reads `context.history`, the
  conversation as it stood when the run was served. The trigger is a fact
  about the past, so it is read from the past.
- **agfront `df8da45`** — the guide now says that reading a topic is free and
  posting into one is what makes that agent run, so a check-in mid-generation
  starts their whole job again. Front's later runs say "I'll wait rather than
  poll further"; the line landed.

## Attempt 4 — the proof

**One developer post.** 137 messages in the trail from the go (1072) to
Front's closing report (1207): one from the Developer, **zero from the Omni
Agent**, everything else agents.

| # | at (UTC) | where | who | what |
|---|---|---|---|---|
| 1072 | 07:39:54 | `front/front-20260821-p9-assets-4` | **Developer** | two assets, into the project; route it through whoever develops projects; permission for the whole thing |
| 1074 | 07:40:24 | `pj-simpleshooter/workplan-p9-assets-4` | Front | `[SELFNOTE][rootchat] front/front-20260821-p9-assets-4` |
| 1078–1089 | 07:41:43 | `work-s2-30/workrun-task{1..4}` | autolab | four tasks, each **two notes then the description** |
| 1093, 1097 | 07:41:57 | `…/workrun-task1-s2-30` | Front | the trigger — Front's own, off `opened …; post there to start it` |
| 1098 | 07:42:16 | `agforge-agstudio1/assetplan-simpleshooter-enemy-sprite` | autolab | the request, after its root note |
| 1108 | 07:43:17 | `…/assetrun-simpleshooter-enemy-sprite` | forge | anchored, `This topic runs F2-17 … Post here to start it` |
| 1120 | 07:48:51 | `…/assetplan-…` | forge | **`@**autolab-agstudio1**`** — the delivery, naming the trigger |
| 1122 | " | `…/assetrun-…` | forge | the run report, **naming nobody** |
| 1124–1125 | 07:49:18 | `work-s2-30/✔ workrun-task1-s2-30` | autolab | reports, closes, resolves — **task 1 `✔`** |
| — | 07:49:51 | | Front | agrees, triggers task 2 |
| 1133–1134 | 07:51:16 | `✔ workrun-task2-s2-30` | autolab | `aba17f9` — `assets/enemy.png` + `main.js` — **task 2 `✔`** |
| … | 08:18–08:25 | | | the same shape for the BGM: 1173 delivery names autolab once, 1175 names nobody, **task 3 `✔`** |
| 1202–1203 | 08:27:54 | `✔ workrun-task4-s2-30` | autolab | `52828fb` — `assets/stage1-bgm.mp3`, looping on first keydown — **task 4 `✔`** |
| 1207 | 08:29:09 | `front/front-20260821-p9-assets-4` | Front | tells the developer both assets are in the project, with both commit ids |
| | 08:29:09 → 08:34:15 | | | **silence** |

The assets are real and in `pj-simpleshooter/main`:

```
52828fb Add stage-1 BGM and play it as a looping track on first input
aba17f9 Draw enemies with the enemy sprite instead of a flat rect
```

All four Sub-Works are `completed` in Plane and all four have a devlog record
(`devlog/s2-30-…/task-{1,2,3,4}`).

### The bug the run found in the middle of itself

Between task 2 and task 3 the mission stopped for 26 minutes, and no log said
anything was wrong. Task 2 reported completion naming Front and resolved its
own topic a second later; Front's callback arrived holding the name the topic
had **when the post was made**, `rootchat_home` read that bare name, found no
messages at all — not "no note of ours" — and dropped it.

That is a race every task hits, because the post that names the supervisor is
the post that finishes the conversation.

**pyagag `938d3ab`** — `topic_history_across_resolve`; `rootchat_home` and
`note_served` both read through it. `agentchat wait` and `read --since` have
followed the rename since `5bda102`; this is the same rule where the callback
is decided. The write side had the mirror bug — a task that closes renames its
own topic, so the served note was posted under a name that no longer existed
and opened a second topic beside the real one holding only that note. That is
what `work-s2-30/workrun-task1-s2-30` is, and `live_topic_name` (moved into
pyagag from agautolab) fixes it.

**The recovery then did its job with no human post.** Restarting the three
listeners was the whole intervention: Front's startup sweep found the five
callbacks it had been dropping, and the mission resumed at task 3. The memory
is in the chat, so a listener that learns to read it again remembers where it
was.

## The criteria

| criterion | verdict |
|---|---|
| 1. Developer → Front → `workplan-` → go, then zero human or Omni posts to three tasks `✔`, the assets in the project, and Front telling the developer | **met** — one developer post, four tasks, two commits |
| 2. exactly one supercoder run per callback; the delivery must not start two; the startup sweep must not start any | **met** — 1120/1173 name the trigger once, 1122/1175 name nobody, and both restart sweeps started zero runs |
| 3. recovery is safe, mid-delegation and after everything is `✔` | **met, on the second try** — see below |
| 4. `agag.participation` unimported, agautolab on current pyagag, `AGENTCHAT_LEDGER` gone | **met** ([report3](report3.md)) |

### Criterion 3 in detail

**Mid-delegation**, 07:09:41, while forge was generating:

```
full sweep: 0 awaiting, 1 mentioning, 34 calls spent, 972 left in the window
serving mention in 'pj-assetpipe1'/'create-asset_…'
mention in … carries no root note of ours; ignoring
```

Nothing resumed. Without step 1's marker this would have re-served every
exchange the agent had ever had.

**After everything was `✔`**, 08:34:15, five minutes into silence — and it
**failed**. It served two finished exchanges again. The mark was there and
`sweep_rootchats` skipped them correctly; they came back through
`sweep_mentions`, which had no notion of "already answered".

That route used to silence itself: answering a mention meant posting in the
topic it was made in, and Zulip stops offering a mention once it is read. p8
sent the answer home instead, so `is:mentioned` returns the same posts for the
rest of the topic's life. p9 gave the mark to one of the two recovery routes
and left the other, and this restart is what found it.

**pyagag `65bd3b2`** — both routes take the marks, looked up once in
`sweep_serve`. What an agent has already answered is a fact about the agent,
not about the route the recovery came in on. Re-run at 08:39:11:

```
agautolab: full sweep: 0 awaiting, 1 mentioning …  → carries no root note of ours; ignoring
agfront:   full sweep: 0 awaiting, 1 mentioning …  → carries no root note of ours; ignoring
agforge:   full sweep: 0 awaiting, 0 mentioning
```

Both mentions are topics from other episodes that carry no root note. Nothing
of this mission was re-served, and no agent ran.

No agent runs were spent by the failing restart either — the topics it served
were resolved, so `serve_topic` found "no messages" and posted its empty
reply. It cost posts and a stray topic, not money.

## Cost of the phase

| | |
|---|---|
| attempts | 4 (three cancelled, one clean) |
| defects found by running it | 5, all pre-existing, all fixed |
| supercoder runs (whole phase) | 40 records total, 10 in attempt 4 |
| forge generations | 7 records total, 2 in attempt 4 |
| Front runs | 65 records total |
| assets delivered | 2, both committed to `main/` |

## Open, and noted in passing

- **The mission Work is never closed.** All four Sub-Works are `completed`,
  and S2-30 is still `unstarted` — Front triggered the tasks directly rather
  than through `start.flag`, and nothing transitions a mission when its last
  task finishes. The workflow completes; Plane's picture of it does not.
- **`work-s2-30/workrun-task1-s2-30`** is a stray unresolved topic holding one
  selfnote and one empty reply — the write-side rename bug, fixed after it was
  made. Harmless (a topic of only selfnotes awaits nobody) and left as
  evidence.
- **A cancelled plan Work is revived silently.** forge keys a plan by
  `<channel>/<topic>`, so a new `assetplan-` topic of the same name upserts
  onto the old Work — attempt 4's sprite ran as F2-17, which p6's cleanup had
  Cancelled.
- **`unknown toolset 'toolset'; skipped`** appears in every forge assetplan
  run of this phase. The front writes a `toolsets.csv` header row that
  `toolsets.parse_names` treats as a name. Cosmetic, not this phase's.
- **Deleting beats resolving for residue.** Resolving p6's forge topics made
  the topic names poisonous — a registered plan whose run topic is resolved
  can never be run again, because `open_assetrun` is idempotent by its work
  note. Attempt 4 was clean because the topics were deleted outright.

*Did the p6 and inter-attempt cancellations, the residue deletion and the two
listener restarts for the agents — housekeeping and the criterion-3 test, not
handoff candidates.*
