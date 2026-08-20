# agent_standardize p1 plan — agforge as the first standardized entrance

Goal: give agforge the standardized shape — an instance name, its own channel
as Single Entrance, and an intro post in `#agents` — with placeholder content
where content doesn't matter yet. Form over substance: the shape working end
to end is the deliverable.

Success criteria (both required, nothing more):

1. The Omni Agent posts a `create-` topic in the new `agforge-agstudio1`
   channel and a generation completes there (existing generation path, no new
   behavior).
2. `#agents` contains an `intro-agforge-agstudio1` topic with the intro
   posted from a fixed `.md` file.

Decisions already made (see [../braindump.md](../braindump.md), [braindump.md](braindump.md)):

- Instance naming: `agforge-agstudio1` = `<agent>-<instance label><N>`, label
  is the hostname for now. Zulip and Plane account names unify on this.
- The unlabeled name `agforge` is reserved for a future aggregator front;
  out of scope here.
- One channel per agent instance; requests and questions concentrate there.
  The instance listens to all topics of its own channel.
- `FreeForge` channel is retired. Per-project `pj-*` channels stay for now.
- Breaking-change phase: **no backward compatibility. agfront's relay to
  forge may break and stay broken** until the next phase re-points it.
- Intro content may be canned: a fixed markdown file posted verbatim.

Constraints (deliberately minimal — everything else is implementer's discretion):

1. Secrets stay in `.local/` (existing rule); never in tracked files.
2. pyagag is consumed from GitHub (`pyproject.toml` pins the git source). Any
   pyagag change must be commit → push → re-lock in agforge
   (`uv lock --upgrade-package pyagag`) — never a local path or gitea source
   (localrule.md).
3. This is the experimental LAN realm; no other security hardening is asked for.

## Step 1 — Rename the accounts, seed the identity file

- Zulip: rename the forge bot (user id 13) full name to `agforge-agstudio1`.
  Realm admin credentials: `pj-agdev/.local/zulip/developer.env`. The realm
  keys on numeric ids and hides emails, so renaming keeps all history; the
  bot's email does not need to change. `PATCH /users/13` (or the `/bots/`
  variant if Zulip insists) with `full_name`.
- Plane: rename the `agforge` member's display name to `agforge-agstudio1`
  and update `PLANE_AGENT_SLUG` in `pj-agdev/.local/plane/agforge.env`.
  Ritual reference: `devdocs/episodes/agent_intent/p1/report2.md`; scripts in
  `.local/plane/`. Display-name level is enough — do not recreate the account
  (issues/comments key on the user).
- Identity seed: put the instance name in ONE place the code reads, e.g.
  `agforge/.local/instance.toml` with a single `name = "agforge-agstudio1"`
  key (plus a committed default/example). This is the v1 of the
  self-definition file from the episode braindump — keep it to the name; do
  not design the full schema here.

## Step 2 — Channels: create `agforge-agstudio1` and `#agents`, retire `FreeForge`

- `ZulipClient.create_channel(name, description, principals)` in
  `agag.zulip` creates-or-joins and subscribes in one call (a default-role
  bot may do this — proven earlier). Subscribe at least: forge bot (13),
  Omni Agent, Developer (8).
- Create `#agents` the same way. Optional nicety: a channel folder for agent
  channels (`channel_folders()` / `create_channel_folder()` exist since
  pyagag `97d2f8d`; folder placement only applies at creation, and folder
  creation is not idempotent — look the name up first). Fine to skip.
- Retire `FreeForge`: unsubscribe forge, archive the channel. Archiving needs
  the creator or a realm admin — use `developer.env`, same as the
  `archive_projects` sweep. The Plane `FreeForge` project is untouched (out
  of scope; note it in the report).
- Leave forge's `#general` and `pj-*` subscriptions alone unless they get in
  the way — the next phase re-points agfront, not this one.

## Step 3 — Listener: own channel = entrance

Current shape (`agforge/src/agforge/zulip_listener.py`): `sweep_serve` from
`agag.zulip` sweeps ALL subscribed channels for the global prefixes
`("runcreate-", "create-")`; resolved topics are renamed `✔ …` and stop
matching. That means `create-` topics in the new channel work with **zero
code change** the moment the bot is subscribed — success criterion 1 is
nearly free.

What's new is "listen to every topic of MY channel":

- `sweep_serve`'s `topic_filter` is prefix-only today. The clean cut is to
  let it accept a callable `(channel, topic) -> bool` in `agag.zulip`
  (`sweep_topics` + the event path both consult it), then have agforge pass
  "everything in my own channel, the old prefixes elsewhere". Remember
  constraint 2: pyagag change → push → re-lock.
- Cheaper alternatives are acceptable if preferred (e.g. a second sweep loop
  for the own channel). Implementer's choice; the callable is the hint, not
  an order.
- Handler for non-`create-` topics in the own channel: entrance behavior —
  answer from the canned self-description and, when the post is really a task
  request, point at / open a `create-` topic. A fixed-text reply from the
  same markdown used for the intro is fine for p1 (placeholder allowed).
  Guard the obvious loop: never react to the bot's own posts (the sweep
  already skips topics whose last poster is the bot — that existing rule is
  the loop guard; keep it).
- The old DM route can stay as-is this phase; it is cagent whose DM path is
  slated for removal, not forge's.

Reload ritual after code changes:
`launchctl kickstart -k gui/$(id -u)/com.agdev.agforge-zulip`; log at
`agforge/.local/out/zulip-listener.log`. No launchd/port parameterization
this phase — single instance, existing labels.

## Step 4 — Intro post

- A committed markdown file (e.g. `agforge/params/intro.md` or wherever fits
  the tree) holding the canned self-description: what the instance does, and
  "open a `create-…` topic in `agforge-agstudio1` to request".
- A tiny CLI (`uv run python -m agforge.intro`) that posts the file to
  `#agents`, topic `intro-agforge-agstudio1`, via
  `ZulipClient.send_to_channel`. Run it once by hand for p1.
- Stamp each post with the date and `git rev-parse --short HEAD` so readers
  can tell a stale intro from a current one. The topic is append-only
  history — re-run on behavior updates; no dedup logic needed.

## Step 5 — Prove it and report

1. Post the intro (Step 4) and check it in `#agents`.
2. As the Omni Agent (credentials `.local/zulip/omni-agent.env`), open a
   `create-…` topic in `agforge-agstudio1` with a small asset request; wait
   for the generation reply. Every reacted post is one paid run — that's
   fine, do not economize.
3. Entrance smoke: post a plain question topic (no `create-` prefix) in the
   channel and confirm the canned entrance reply arrives.
4. Write `report.md`: what was renamed/created/retired, message links as
   evidence, and the explicitly deferred items — agfront relay broken, Plane
   FreeForge project untouched, full self-definition schema and
   launchd/port parameterization not started. Update `agforge/README_DEV.md`
   and the `.local/devenv.md` FreeForge section to past tense.
