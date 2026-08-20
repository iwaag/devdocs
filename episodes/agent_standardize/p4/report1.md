# agent_standardize p4 — Step 1 report: autolab becomes an addressable instance

AI-generated (Omni Agent, 2026-08-21).

## What changed

autolab now has the same standardized form p1 gave agforge: a named instance,
its own channel as the entrance, and the machinery to introduce itself. The
difference from p1 is that the second copy is not a copy — the shared parts
moved into `pyagag` and both agents call them.

### The Zulip rename

`PATCH /users/11` with `full_name=autolab-agstudio1`, as the realm admin.
The bot's email (`autolab-agstudio-bot@…`) is untouched, so every existing
message, subscription and channel membership still keys on user 11.

```
11 'Autolab Agstudio'  ->  11 'autolab-agstudio1'
```

`Autolab Agautolab1` (user 12) is deliberately **not** renamed. That node runs
the gateway and no Zulip listener, so it owns no channel and answers nothing:
naming it would advertise an entrance that does not exist.

### Plane: deferred on purpose

`.local/plane/autolab.env` is the credential file the **agautolab1**
deployment is also pointed at (`AUTOLAB_NODE_PLANE_CREDENTIALS_SOURCE`).
Renaming its display name would relabel that node's issue authorship as this
Mac's instance, which is worse than the current under-specified name. The
plane identity split stays open; recorded in the phase report.

### The identity seed

- `agautolab/.local/instance.toml` — `name = "autolab-agstudio1"` (git-ignored)
- `agautolab/instance.example.toml` — the committed example
- `agautolab/src/agautolab/instance.py` — `instance_name()`, three values over
  the shared reader
- `agautolab/tests/test_instance.py` — three cases

Local-only for the same reason as forge's: the label is the host, and
`devdocs/README_DEV.md` keeps host information out of tracked files. With no
file the plain `autolab` is used — wrong for an instance, but wrong out loud.

### The own channel and the sweep

Zulip channel `autolab-agstudio1` (stream 36) exists, in the `agents` folder
(id 3), with the Developer (8), the Omni Agent (9) and the autolab bot (11)
subscribed. The listener's routing is now:

```
own channel (`autolab-agstudio1`)  -> every topic
other subscribed channels          -> workplan- / workrun- / assetplan- / bmining-
```

`topic_filter()` is the same two-line callable forge has, over the callable
`topic_filter` pyagag has accepted since p1 step 3. The startup log is its own
witness:

```text
agautolab zulip listener starting (pull sweep: all topics in 'autolab-agstudio1',
  prefixes ('workplan-', 'workrun-', 'assetplan-', 'bmining-') elsewhere)
```

**The own channel executes nothing.** Its check is the *first* branch of
`dispatch`, ahead of `workrun-`, so no topic name in that channel can reach a
handler — the entrance answers with a redirect and nothing else. A test
asserts exactly that, failing if any handler is called. This is what makes the
entrance safe to leave open while criterion 4 forbids firing a `workrun-`.

The redirect is the placeholder reply the plan asked for: it names the
instance, says work goes in a `workplan-…` topic in the project's own
`pj-<slug>` channel, gives the reason (the channel is what says which project
the work is for), and offers to answer "which projects exist" here. Because
the shared sweep skips a topic whose last post is the bot's own, that reply is
also the loop guard.

### Intro machinery: lifted, not copied

The plan's preferred option. `agag.intro` holds `revision()`, `intro_topic()`,
`intro_text()` and `post_intro()`; `agag.instance` holds the identity read.
`agforge.intro` and `agforge.instance` were switched to them **in the same
change**, not later, and keep their module names and the
`python -m agforge.intro` CLI. `agautolab.intro` is now ~25 lines of its own
paths. pyagag's README documents both, with the wiring call.

## Evidence

A plain question at the entrance, answered by the redirect:

| message | sender | topic | content |
|---:|---|---|---|
| 705 | Developer (8) | `how-to-request` | "what are you, and how do I ask you for development work?" |
| 706 | autolab-agstudio1 (11) | `how-to-request` | the redirect naming `workplan-…` and `pj-<slug>` |

## Verification

- pyagag: **263 passed** (9 new, in `tests/test_intro.py` and
  `tests/test_instance.py`).
- agforge: **189 passed** after re-locking onto the shared modules — the same
  count as p1 step 4, which is the point: the CLI and its behavior did not move.
- agautolab: **178 passed** (7 new: the filter, the entrance's refusal to
  execute, the reply's content, and the intro post).
- The launchd listener was reloaded with `launchctl kickstart -k` and logged
  the new policy on a clean startup sweep.

## Delivery

- pyagag `7a00e7b` — *Share instance identity and the intro board across
  agents* — pushed to GitHub `main`.
- agforge `d16ba50` — re-locked from `c33d1eec` to `7a00e7b6` — pushed.
- agautolab `76eb0f0` — re-locked from `97d2f8dc` to `7a00e7b6` — pushed.
- pj-agdev `b6d9b5f` records both submodule revisions.

No local path dependency and no local Git remote was introduced.

## Not done in this step

- `params/intro.md` is a one-line placeholder; Step 2 writes the contract and
  posts it. Nothing is in `intro-autolab-agstudio1` yet.
- agautolab1 is not redeployed. Its gateway does not read any of this, and
  `--limit agstudio` deploys into this working tree.
- The entrance reply is a code-level placeholder, as forge's was after p1.

Renamed an account, cut a channel and seeded an identity file that an
in-system autolab could argue it should maintain itself — handoff candidate.
