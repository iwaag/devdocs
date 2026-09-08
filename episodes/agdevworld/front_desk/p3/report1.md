# Step 1 — discovering the exact completion targets

agdevworld `5dc8e0b`. Nothing in agfront, agautolab, agforge or pyagag
changed; nothing writes yet.

## What exists now

`agdevworld/agentroom/src/agentroom/closing.py` answers one question,
read-only: **what would closing this Front Desk conversation touch?**
`discover(topics, ident, realm=…, plane=…)` returns targets, the record each
one was found by, exclusions with their reasons, and the gaps.

- **The walk is the two link notes, in both directions, to a fixed point.**
  It reuses `routines.children_of` — a `[served]` note written in a topic
  names a remote its owner answered, a `[rootchat]` note written in a remote
  names the conversation it was opened for — but not that board's
  assumptions: depth is unbounded (the node cap is 120 and *reaching it is
  reported*), and a topic read late carries notes naming a topic expanded
  early, so the sweep repeats until nothing is added.
- **Resolved topics are read, not skipped.** The ops sweep never reads a ✔
  topic, which is most of a finished session. Every node the engine does not
  hold with a history is fetched from Zulip under **both** names and the two
  answers merged. A name neither read could answer stays `note-only` and
  goes into `gaps.unread` — never into an empty history.
- **The blind spot both notes share is closed by the channel's own topic
  list.** A topic an agent opens for *its own* conversation is named by
  nothing: forge's `assetrun-<stem>` beside its `assetplan-<stem>` was
  invisible to the walk. The shared stem — the writers' own pairing
  (`agforge.anchor.assetrun_topic_name`) — decides only *what to read*; a
  sibling joins the graph on its own link notes or not at all, which a test
  pins with a same-stem topic that carries none.
- **`[work]` is now a retained binding.** `closing.parse_work_note` reads one
  tag in two shapes — `<issue id>` (autolab) and `<project id>/<issue id>`
  (forge) — and `ops.Topic.link` keeps them beside `rootchat` and `served`,
  so a held conversation answers without a read. (A selfnote is not a real
  post and is in no `history`; without this the engine's own memory could
  not have answered.) `ops.Topic` also normalises a root note's topic to its
  bare name now, like every other key in that module.
- **Plane Works come from identifiers, never from prose.** A mission is the
  issue whose `external_source`/`external_id` is `agautolab` +
  `<channel>/<topic>` of a reached `workplan-` topic (`mission.work_key`); a
  task is what a `[work]` note names, and its `parent` is corroborating
  evidence for the same mission. Projects in play are the `pj-<slug>`
  channels reached (matched to `[AUTO]` project descriptions) plus any
  project a forge note names outright. Every child of a mission is listed
  with its state and a `reached` flag — **including one this walk never
  saw**, which is what must block a close rather than be assumed done.
- **A channel is identified by its contents.** A `work-<label>` channel is
  archivable only when a mission Work of *this* conversation carries that
  label **and** every topic the channel actually holds is one of these
  targets. Anything else is reported by name (`unaccounted`) and kept. A
  shared project or agent channel is never a candidate.
- **Ambiguity is returned, not resolved.** Another `front-desk-` topic is
  never a target. A topic whose root note names a *different* desk
  conversation is excluded with that note's message id — p2's reused plan
  topic, met head-on — and so is anything reachable only through it, since
  that work is the other conversation's to close. A target this conversation
  also reached by a link of its own survives the cascade.
- **Plane and Zulip failures are conditions, not crashes.** `PlaneReader`
  wraps `agag.plane` and keeps errors; a credential that reads one project
  and is refused another (which planning measured) produces the Works it
  could see plus a `gaps.plane` line naming what it could not.

## Verification

`agentroom` `uv run pytest -q` → **190 passed** (21 new in
`tests/test_closing.py`), all on fixtures: the whole Front → workplan →
workrun → forge chain through ✔ names, the unnamed sibling, a same-stem
non-relative, a cycle, the node cap, the engine's memory answering with no
read, a shared plan topic and its cascade, an unreadable topic, a
400-message window, a Plane project refused, an unreached Sub-Work, a
cancelled child, and four channel verdicts.

Live read-only checks against the realm and Plane (Developer Zulip
credential, admin Plane key, `topics={}` so every read was paid):

| conversation | topics found | Works | channel |
|---|---|---|---|
| `front-desk-20260908-164810` | desk, `pj-ghtrends/workplan-trend8`, `workrun-task1-g-17` ✔, `workrun-rerun-task1-g-17` ✔ | G-17 mission **started**, child G-18 completed | `work-g-17` archivable |
| `front-desk-20260908-161951` ✔ | desk ✔, `workplan-trend7` ✔, `workrun-task1-g-15` ✔ | G-15 completed, G-16 completed | `work-g-15` archivable |
| `front-desk-20260908-1600` | desk, `workplan-trend6`, `workrun-task1-g-13` ✔ | G-13 mission **started**, child G-14 completed | `work-g-13` archivable |
| `front-desk-20260908-p2live` | desk, `autolab-agstudio1/status-ghtrends-g17` | — | — |
| `front-desk-20260908-ex1` | desk, `assetplan-…-robot`, `assetrun-…-robot` | F2-28 completed | — |

8–14 Zulip calls each with an empty engine memory and no cache; no gaps, no
errors, no exclusions. G-17, G-13 and G-11 are exactly the braindump's
complaint — a mission left `started` with every Sub-Work completed.

## What this leaves for step 2

- The eligibility rule itself (`mission_done.reason_not_finished`) is not
  used yet; discovery reports states, step 2 decides.
- Plane credentials: the per-agent `autolab` key is **HTTP 403 on Ghtrends
  states**; the admin key in `.local/plane-credentials.env` reads it. Which
  credential the relay is given, and reporting a refusal at preview time,
  is step 2's configuration decision.
- Nothing resolves, archives or transitions anything yet.
