# agent_standardize p1 — Step 2 report: the instance's channel, `#agents`, and the end of FreeForge

AI-generated (Omni Agent, 2026-08-20).

## What changed

The realm now has the two channels the standard shape needs, and the channel
the shape replaces is archived.

| channel | stream id | folder | subscribers |
|---|---|---|---|
| `agforge-agstudio1` | 34 | `agents` (3) | 13 agforge-agstudio1, 9 Omni Agent, 8 Developer |
| `agents` | 35 | `agents` (3) | 8, 9, 11, 12, 13, 14, 15 |
| `FreeForge` | 5 | — | **archived** |

Both were made with `ZulipClient.create_channel(name, description, principals,
folder_id=…)` from `agag.zulip` — one call each, creating the channel and
subscribing everyone. The realm admin account (`developer.env`) made the calls
rather than the forge bot: a default-role bot may create a channel here, but
subscribing *other* principals in the same call is cleanly an admin action, and
`#agents` needs six accounts that are not the caller.

Descriptions, verbatim, because they are the only thing a stranger agent sees
before posting:

- `agforge-agstudio1` — "Single entrance for the agforge-agstudio1 instance.
  Open a create-… topic to request an asset; any other topic is a question to
  the instance."
- `agents` — "Where agents introduce themselves: one intro-<instance name>
  topic per agent instance, saying what it does and how to ask it."

### Who is in `#agents`

Every active Zulip principal in the realm: Developer (8), Omni Agent (9),
Autolab Agstudio (11), Autolab Agautolab1 (12), agforge-agstudio1 (13), Cagent
(14), Front (15). The retired Devworld Assistant (10) is inactive and was left
out. Subscription is the routing decision here, so the choice is deliberate:
`#agents` is a board every agent can read, not a place requests are served.

### The channel folder

The plan called the folder an optional nicety; it was taken. `channel_folders()`
was checked first (folder creation is not idempotent), found only the two
per-project folders `pj-runsmoke2` (1) and `pj-simpleshooter` (2), and a third
folder `agents` (3) was created — "Agent instance channels and the shared
#agents board". `folder_id` only applies at creation, which is why it had to
happen in the same call as the channels and could not be added afterwards.

This extends the existing "one folder per project" standard with a sibling
category rather than contradicting it: project channels group under `pj-…`,
agent channels group under `agents`.

## FreeForge, retired

```
subscribers before archiving: [8, 13]
topics:                       15
unsubscribe principals=[13]:  removed ['FreeForge']
archive stream 5:             success
```

Archiving keeps all 15 topics and their messages; the channel simply leaves the
active listing and stops being sweepable. It needed the realm admin for the
same reason the `archive_projects` sweep did — only the creator or an admin may
administer a channel.

forge's subscriptions afterwards:

```
agents, agforge-agstudio1, general, ops, pj-foodchain, pj-simpleshooter,
sandbox, work-s2-1
```

`general` and the `pj-*` channels were left alone on purpose. `general` is
where agfront drops its `create-…` requests, and that relay is the *next*
phase's business; the plan explicitly allows it to break rather than be
propped up here. Because `sweep_serve` still matches `create-` in any
subscribed channel, forge in fact keeps serving `general` and the `pj-*`
channels unchanged — nothing was taken away, one place was added.

## Not done, deliberately

- **The Plane `FreeForge` project is untouched.** It is out of scope per the
  plan; it still exists, with its issues, and nothing about this step
  invalidates it. Retiring it is a separate decision from retiring the channel.
- **No code reads either channel name yet.** `sweep_serve` finds `create-`
  topics in `agforge-agstudio1` today only because forge is subscribed — which
  is exactly the "success criterion 1 is nearly free" the plan predicted, and
  it is proved in Step 5, not asserted here. "Listen to every topic of my own
  channel" is Step 3; posting into `#agents` is Step 4.
- **`FreeForge` is still named in the docs** (`agforge/README_DEV.md`,
  `pj-agdev/.local/devenv.md`). The plan puts moving those to past tense in
  Step 5, with the rest of the doc sweep.

Created channels and archived one that in-system agents could arguably manage
for themselves — handoff candidate.
