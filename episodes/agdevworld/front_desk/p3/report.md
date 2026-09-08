# Front Desk p3 — closing a conversation and its related work

pyagag `0e38c25`; agautolab `111ed7f`; agdevworld `a87fdf9`; pj-agdev
pointer moved. Steps 1–4 done; one live completion carried out.

## The braindump, answered

> front desk に、front のトピックとその関連ワークのトピックを resolved に
> したり、(存在するなら)関連 work チャンネルをアーカイブするボタンが欲しい。
> …plane のワークの方もたしか done に自動的にならず残ってしまう状態だった
> はず。これも合わせて done にできるならしたい。

There is a `finish ✔` button on the Front Desk. It previews, in one panel,
every topic it would resolve, every dedicated `work-` channel it would
archive and every Plane Work it would close — each with the record it was
found by — and the click applies exactly that. **And yes, it is
deterministic**: no agent run is bought to work out what is related.

The first live use closed `front-desk-20260908-1600`: Plane **G-13 → Done**
(a mission left `started` with its only Sub-Work completed — the braindump's
own complaint), `✔ workplan-trend6`, `#work-g-13` **archived**, and the Front
conversation `✔`.

## How relatedness is known

Not by names, and not by asking anybody. The realm already records it, in
notes agents write to themselves (`agag.selfnote`) and in Plane's own
identifiers:

- **`[rootchat]` and `[served]`**, walked in both directions to a fixed
  point. The routines board's `children_of` supplies the edge logic; its
  *limits* do not carry over — depth is unbounded here (the node cap is
  reported when reached), and every topic the engine does not hold is read
  from Zulip under **both** its names, because a resolved topic is never
  swept and that is most of a finished session.
- **The blind spot both notes share** — a topic an agent opened for its own
  conversation, which nothing names — is closed by the channel's own topic
  list: forge's `assetrun-<stem>` beside its `assetplan-<stem>`. The stem
  decides only *what to read*; a sibling joins the graph on its own link
  notes or not at all.
- **`[work]`** — one tag, two shapes (`<issue id>` for autolab,
  `<project>/<issue>` for forge) — is now a retained binding on
  `ops.Topic`, so a held conversation answers without a Zulip read.
- **Plane's `external_source`/`external_id`**: a mission Work names the
  `workplan-` topic that planned it (`mission.work_key`). A Sub-Work's
  `parent` corroborates the same mission from the other side.

## What it refuses to do

- **Another conversation's work.** A topic anchored to a different
  `front-desk-` conversation is excluded with that root note's message id —
  p2's reused plan topic, met head-on — and so is anything reachable only
  through it.
- **Invent a completion.** A Work moves only under
  `agag.plane.reason_not_completed`, the rule `agautolab.mission_done`
  applies, moved into pyagag in this phase so the button could ask it about
  one named Work instead of running a whole-board CLI. Unfinished children,
  or a standalone Work with none, are **blocked**. A cancelled Work is left
  exactly as it is. A Sub-Work the walk never reached is listed and counted.
- **Treat silence as done.** A topic that could not be read is a gap and
  blocks. A `work-` channel is archivable only when a mission of *this*
  conversation carries its label **and** every topic it actually holds is
  one of these targets; anything else is named and kept.
- **Claim more than happened.** Plane first, then topics, then channels,
  then the Front conversation — and the Front ✔ is skipped when anything
  above is blocked or failed, because that ✔ is the claim the whole thing is
  finished. The screen says *"partially closed … the conversation stays
  open"* in words.
- **Write what a browser asked for.** The request carries the conversation
  id and the fingerprint of the preview that was read; every target is
  re-derived server-side, and a plan that changed since is refused with the
  fresh one and no write. A test posts foreign topics and channels in the
  body and watches them be ignored.

## Retrying, and reopening

Nothing rolls back and nothing retries itself. Every action is idempotent in
the realm's own terms — a resolved topic answers `already`, a Done Work
answers `already` — so a second click after a partial failure finishes what
is left and repeats nothing. Reopening a closed conversation (posting into
it un-resolves the Front topic) does **not** reopen the work: the resolved
child topics and the archived channel are untouched, and the screen says so
rather than letting a reader assume it.

## What the live run taught

Archiving a channel is what makes it unlistable, so the second preview
reported this operation's own finished work as *"the channel's topics could
not be read"* — a gap where there was none. The realm's channel list tells
the two apart, and it is asked only after a topic list has failed. Fixed and
pinned the same day. That is the one thing four steps of fixtures could not
have produced, and the reason the plan asked for a live run.

## Left standing

- **Plane credentials are the admin key.** The per-agent `autolab` key is
  HTTP 403 on one project's states, so an agent identity cannot drive this
  button today. `AGENTROOM_PLANE_ENV` is deliberately configurable, and a
  relay without it still closes the chat half and says which Works it could
  not see.
- **Reopening was not exercised live** — any post there buys a paid Front
  run. Checked in the code and in the panel instead, and said so.
- No force-completion, and no way to stop a running agent. Both are outside
  this phase, and every payload carries the sentence *"this closes work; it
  does not stop a running agent"*.

## Steps

- [step 1](report1.md) — discovering the exact completion targets
- [step 2](report2.md) — planning and applying the state changes
- [step 3](report3.md) — the completion flow in the scene
- [step 4](report4.md) — integration, live verification, deployment
