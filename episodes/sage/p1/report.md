# sage p1 — final report

## Result

`arxivsage-agstudio1` is a live, launchd-managed agag agent for the public
`iwaag/study-arxiv-trend` knowledge tree. It serves `entrance-…` topics in its
own Zulip channel, cites the files it read, records researchable unknowns in
an ignored study queue, and reports `liveness=polling` after a fresh nctl
observation.

Step evidence: [generation](report1.md), [listener](report2.md),
[contract](report3.md), [knowledge deployment](report4.md),
[launchd](report5.md), [acceptance](report6.md), [desired declaration](report7.md),
and [documentation](report8.md).

## Delivered repositories

- `iwaag/arxivsage`: generated project, knowledge-sync script, custom sage
  listener, scope guide, intro, launchd template, and locked GitHub pyagag
  dependency.
- `iwaag/pyagag`: commit `a49ec8e` preserves explicitly supplied run metadata
  in persisted records; focused agent/harness tests passed (35).
- `iwaag/devdocs`: this phase's reports and the shared sage documentation.

The arxivsage listener and its direct dependency updates were pushed to GitHub
before being used locally.

## Acceptance and costs

Six requested acceptance cases passed after one intentional failure-farming
guide correction. The cases consumed $0.5529926 across sage runs (including
the observed failure and its retry). A post-fix metadata verification run cost
$0.0856824, for a total observed sage cost of **$0.6386750**. Each record has
cost, turns, transcript path, and — after the pyagag fix — the knowledge
revision; `run-0007` records `knowledge_revision: "31ff73c"`.

## Open handoff: study queue consumer

The next phase must make someone read `arxivsage/tostudy/`, run the requested
study, publish the resulting paper directory to `study-arxiv-trend`, and
delete the consumed note. The first produced note demonstrates the contract:

```md
# AutoDesign (arXiv:2608.13560)

Asked in <channel> by <asker>: "<question as asked>"

## Why in scope

<why this is an LLM-agent or agent-harness research request>

## Status

Not present in knowledge/README.md at revision <revision>.

## What to look for

- arXiv id: <id>
- Title: <title>
- Requested artifact: a summary (summary.md)

- asked again in <channel>/<topic>: "<one-line question>" (<date>)
```

The intended consumer is a `workplan-` topic in `#pj-studyarxiv`. Its result
must land in the public `publish/` tree and be pushed by the developer; then an
operator runs `sync_knowledge.sh`. No automatic refresh or study-side consumer
is part of p1.

## Addendum (2026-08-30): the channel was not in the `agents` folder

The developer noticed after the phase closed that `#arxivsage-agstudio1` was
missing from Zulip's `agents` channel folder. It was never in this plan —
"folder" appears nowhere in `plan.md`, `braindump.md` or reports 1–8 — so
this is a gap in the plan, not an execution miss.

The cause is one line that was never written: `agag.zulip` has had
`create_channel(folder_id=…)` and `create_channel_folder()` since pyagag
`97d2f8d`, but **no provisioning code ever passed a folder**, so every
channel `agag` has ever created landed unfiled. arxivsage was the fifth:
`front` (24), `agecho-agstudio1` (46), `agping-agstudio1` (49) and
`agecho-agautolab1` (50) were unfiled too. Only `agforge-agstudio1` (34),
`autolab-agstudio1` (36) and `agents` (35) were in folder 3, all placed by
hand before agag existed.

Fixed in code rather than by hand, because the next `agag init` would have
repeated it:

- pyagag `ce99a68` — `provision()` resolves the folder by name (minting it
  when the realm has none) and passes it to `create_channel`. A channel that
  already exists is only *joined*, so `create_channel`'s `folder_id` files
  nothing in that case; it is moved instead with a new
  `ZulipClient.set_channel_folder` (`PATCH /streams/<id>` with `folder_id`,
  verified against Zulip 12.2, feature level 500). `--no-folder` opts out.
  Five new `agag provision` tests; full suite 416 passed.
- agautolab `e9fa5c5` — lock bumped to that pyagag and the agstudio listener
  restarted, since autolab is the agent that can run the whole `agag init`
  chain from a workplan.
- The realm was reconciled by hand once: all eight agent instance channels
  plus `#agents` are now in folder 3. That is a one-time repair of channels
  created before the fix; provisioning does it from now on.

Background: `devdocs/episodes/agautolab/channel_folder/report.md`, whose
"ready when needed" recommendation this closes.

## Addendum 2 (2026-08-30): the Developer was not subscribed either

Looking at the repaired folder, the developer noticed the second half of the
same defect: `#arxivsage-agstudio1` had no human in it. Its subscribers were
the sage bot, `Front` (which subscribes itself by posting), the Omni Agent
(same reason, from the step 6 acceptance tests) and `Provisioner` — the
machine account that created it and reads nothing.

`provision()` subscribed `whoami()`, whoever held the credentials named by
`AGAG_ZULIP_ADMIN_ENV`. That was correct while it was `developer.env` — which
is why `agecho-agstudio1` (46) has the Developer — and became wrong the day
agag got its dedicated `Provisioner` account (2026-08-22). Three agents were
provisioned after that day and all three shipped with no human watching:
`agping-agstudio1` (49), `agecho-agautolab1` (50), `arxivsage-agstudio1` (92).

The channels are public, so nothing was hidden; it simply never appeared in
the developer's sidebar. **A conversational entrance nobody is subscribed to
is an entrance with no one behind the door**, which is why this counts as a
bug and not a preference.

- pyagag `f78e2cc` — watchers are the realm's organization owners (Zulip role
  100, non-bot, active), resolved at provisioning time so no generated agent
  carries a realm-local user id. `whoami()` remains only as a fallback when no
  owner is visible: a channel watched by its maker is wrong, a channel watched
  by nobody is worse. The provision output now names the watchers, so the next
  occurrence is visible in the command's own output. Two new tests; 418 passed.
- agautolab `409ad08` — lock bumped, listener restarted.
- The three channels were repaired by hand: Developer subscribed, Provisioner
  removed. All eight agent channels now carry the Developer and their own bot.

Both halves of this were invisible for the same reason: `agag provision`
reported what it *created* and not what it *decided*. It now prints the folder
and the watchers.
