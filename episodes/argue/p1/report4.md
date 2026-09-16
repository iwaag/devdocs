# Step 4 — deploy, demonstrate, report

Date: 2026-09-16 JST (timestamps UTC where they come from Zulip or logs).
Plan: [plan.md](plan.md) step 4. Previous: [report3.md](report3.md).

## Environment, before and after

`nctl status` ok, `nctl drift` **converged 46 / error 0** before the
change; after replacing arxivsage's four rows with archsage's
(`.local/argue-p1-archsage-{1,2}.yaml`, two passes because a
`desired_agent` cannot reference a placement created in the same batch) and
`nctl reconcile agstudio --refresh-observation --max-rounds 1 --yes` +
`nctl agents observe`: **converged 46 / error 0** again, `archsage-agstudio1`
observed with `channels=[agents, archsage-agstudio1]`, arxivsage gone from
the observation. The operator file `.local/desired-state.yaml` carries the
same replacement.

## Deployed

| What | How |
|---|---|
| `#argue` | stream **169**, created with the provisioner, Developer (8) and Front (15) subscribed |
| archsage | `com.agdev.archsage-zulip` bootstrapped from `service/com.agdev.archsage-zulip.plist.in` (installed copy kept at `archsage/.local/`); introduction posted with the sage list (`intro-archsage-agstudio1`) |
| Front, autolab, forge, Observer, cagent | listeners restarted (`kickstart -k`, nothing in flight — checked for harness grandchildren first); introductions re-posted (the "In an argue" paragraphs; Front's argue convention) |
| pyagag pins | every consumer at `ac1262a` after three bumps during the day (`92f6839` → `811d6bb` → `ac1262a`); `uv.lock` committed in agfront `03d2682`/…, agautolab, agforge, agobserver (pj-agdev `ae42c7b`), cagent (pj-clusterintent `20ca10e`/…), archsage `12d810c`/… |
| arxivsage | listener booted out, its plist moved to `pj-agdev/.local/launchd/retired/`, `intro-arxivsage-agstudio1` ✔ (off the board: `agentchat intro` lists seven agents, none arxivsage), Nautobot rows deleted. Checkout and channel left as history. |

Every restart's startup recovery queued **0** conversations except the one
that was meant to (below): the new mention rule replayed no legacy mention.

## The live conversation: `#argue › argue-desk-garden` (argue **7149**)

Origin `#front › front-argue-p1-20260916T0600Z` (message 7147, posted with
the Developer credential). All human turns below were posted **with the
Developer credential by the Omni Agent**; the desire check passed
mechanically (sender 8 is not a bot) and this is *not* evidence of human
authorship — the fixtures in step 1 are. Front's own `argue` runs, and every
other agent's, are real.

| Time | Who | What |
|---|---|---|
| 06:00:15 | Front (`front` run-0630) | opened the argue with `agentchat argue open desk-garden`, anchor 7149, invitation 7150, and told the origin where |
| 06:01:12 | Developer | the desire (7152): an unattended, cheap, 3D-printed desktop box growing sprouts/microgreens to a repeated harvest, replicable by others |
| 06:01:30 | Front (argue run-0001) | `[selfnote][desire] 7152 by 8` recorded by the listener from the block; *said* it would bring in archsage but named nobody — **finding 1** |
| 06:02:05 | Developer | "bring archsage in" |
| 06:02:21 | Front (run-0002) | `@**archsage**` with the three-part question |
| 06:03:39 | archsage (`archsage` run-0001, **Fable 5.1**, 13 turns, $0.76, 77 s) | searched the arxiv tree (17 papers, no match), gave the analysis from general knowledge and said so, six research questions, **defined a new sage `growbox`** with an empty tree, recommended build-first with food safety studied alongside |
| 06:03:55 | Front (run-0003) | judged project, asked the human to confirm study-vs-project |
| 06:04:23 | Developer | project; ask cagent about the cluster and the growbox sage about its tree |
| 06:04:53 | Front (run-0004) | one post naming **two participants**: `@**cagent**` and `@**archsage** sage:growbox` |
| 06:05:08 | archsage as **`sage:growbox`** (`sage` run-0002, Sonnet 5, $0.10) | `**[sage:growbox]**` header; "my tree is empty" (`sagetree revision` → `empty`), queued `growbox-project-start` in its study queue |
| 06:05:19 | Front (run-0005) | noted growbox, "still waiting on `@**cagent**`" |
| — | cagent | **nothing**: its display name is `Cagent`; Zulip rendered `@**cagent**` as text and `mentions_bot` was case-sensitive — **finding 2** |
| 06:06:37 | cagent listener restarted on pyagag `811d6bb` | `recovery (startup): 1 conversation(s) queued from the index` → served the invitation left outstanding across the restart — the plan's restart case, live |
| 06:07:20 | Cagent (`argue` run-0001, Sonnet 5, 7 turns, $0.18) | cluster reality from `cagent actual`/`drift`: ollama, minio, postgres, DHCP present; no MQTT/timeseries; opened with `@**Front**` — **finding 3** |
| 06:07:55 | Front (run-0006) | wrote `GOAL.md`, `agproject open desk-garden --kind project --doc GOAL.md`: `#pj-desk-garden` created (folder of its own, members 8/11/15, description `[AUTO] project: desk-garden; project; opened from argue argue/argue-desk-garden; …`), goal posted (7173), `workplan-setup-desk-garden` anchored to the argue (7174/7175) |
| 06:07:55–06:09:08 | autolab (`superdirector` run-0205, 16 turns, $0.37) | one setup serving, no mission: `autodev/desk-garden` with `main/GOAL.md` committed and pushed, `README_PROJECT.md` naming the channel, the topic and argue 7149; flagged the `direction/`/`devlog/` repositories it found — **finding 4**; reply named Front |
| 06:09:08 | Front's mention route | `mention in 'pj-desk-garden'/'workplan-setup-desk-garden' serves argue/argue-desk-garden` — the callback served the argue home with the argue role |
| 06:09:26 | Front (run-0007) | the outcome (desire 7152, why project, artifacts, next work and owner, open questions) with `outcome: project / target: pj-desk-garden / complete: true`; the listener verified channel + `goal` + autolab's answer, wrote `[selfnote][outcome] project pj-desk-garden`, told the origin (7181), **resolved** the argue |
| 06:10:02 | Front restarted | `recovery (startup): 0 conversation(s)`: the completed conversation stays quiet |

Nothing downstream was started: `#pj-desk-garden` holds `goal` and
`workplan-setup-desk-garden` only, no `workrun-`, no routine run; the
growbox sage has no study repository (the outcome names creating one as
the owner's next call).

## Model runs by logical role

| Agent / role | Runs | Model | Cost |
|---|---|---|---|
| Front `front` (opening) | 1 | Sonnet 5 | $0.072 |
| Front `argue` | 7 | Sonnet 5 | $0.656 |
| archsage `archsage` | 1 | **Fable 5.1** | $0.762 |
| archsage `sage:growbox` | 1 | Sonnet 5 | $0.096 |
| cagent `argue` | 1 | Sonnet 5 | $0.179 |
| autolab `superdirector` (setup) | 1 | Sonnet 5 | $0.373 |
| **Total for the argue** | **12** | | **$2.14** |

(The arxiv sage's $0.17 run in step 2 was a test from this shell and is not
in the table.) Records: `agfront/.local/agent/{front,argue}/`,
`archsage/.local/agent/{archsage,sage}/`, `pj-clusterintent/.local/agent/argue/`,
`agautolab/.local/agent/superdirector/`; each argue record carries `argue`
/ `invitation` / `speaker` beside the harness facts.

Redundancy check: one archsage run for the whole argue (called once,
reused), one sage run, one cagent run, seven Front servings — one per
human turn or participant reply, which is the owner's contract. The only
wasted spend was Front's run-0001 that recorded the desire and invited
nobody, costing one extra human turn.

## Findings, and what changed because of them

1. **"I will bring in archsage next" invites nobody.** Front recorded the
   desire and announced the invitation instead of writing it. Guide fix
   (agfront `03d2682`): say what you do in the same reply that does it;
   invite archsage in the very reply whose block records the desire.
2. **`@**cagent**` is not `Cagent`.** Zulip mentions are exact; the
   invitation reached nobody and the discussion stalled on it. Fixed in
   pyagag `fdc32d0`: `mentions_bot` and `argue.mentions_of` match the name
   case-insensitively (the mirror sees every post whether or not Zulip made
   a pill of it), and Front's guide now points at the roster block's
   `bot:` line as the name to mention. The stall is also what produced the
   restart case: cagent restarted on the fix and its recovery found the
   invitation.
3. **Front named cagent twice before it could answer once** ("still
   waiting on @**cagent**"). pyagag `811d6bb`: one reply per logical
   speaker per serving — the newest invitation is answered with all of them
   in front of the run, the earlier ones reacted to.
4. **cagent opened with `@**Front**`.** Harmless (Front owns the topic; the
   owner route wins, so no second run), but a habit the shared guide now
   names: do not mention the facilitator either (pyagag `ac1262a`).
5. **autolab's `init_project` made `direction/` and `devlog/` before it read
   the request** and reported them as a puzzle. That is autolab's standing
   first-serving layout for a workspace without a pattern marker; the
   `study_industry` episode avoided it by writing `README_PROJECT.md` by
   hand first. Not changed here; `agproject` could write the marker through
   autolab's entrance in a later phase, or the setup request could name a
   pattern. Recorded as an open item in `report.md`.
6. Front's first argue reply echoed a `[Front #7153] sender 15 · …`
   header from the evidence-format chatlog. Guide line added; no code.

## Handoff notes

- **Deus Ex Machina notes**: *did the `#argue` channel's creation and the
  Developer's subscription for agent Front — handoff candidate* (`agentchat
  argue open` creates the channel; subscribing the human needs an admin
  credential Front does not hold); *did the archsage bot's display-name
  rename after provisioning — handoff candidate* (`agag provision` names the
  bot after the instance); *posted every human turn of the demo as the
  Developer* — a human's turns are the one thing that cannot be handed to
  an agent, and the report says so above.
- Local notes updated: `pj-agdev/.local/devenv.md` (archsage, `#argue`,
  the retirement, the restarts), `pj-clusterintent/.local/localenv_memo.md`
  (the desired-state change and cagent's mention route).
  `devdocs/README_DEV.md` gained the archsage and argue sections and lost
  the arxivsage one.
