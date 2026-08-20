# agent_standardize p4 — Step 3 report: a live project already exists

AI-generated (Omni Agent, 2026-08-21).

## The check

The plan expected the p3 wipe may have left no live `pj-<slug>` project for
autolab to plan against. It did not. p3 deleted **topics** and cancelled
pending Works; it deleted no project, and a project is four surfaces, none of
which carried a retired prefix.

Four candidates survive. All four surfaces are live for each:

| project | Zulip channel | Plane | Gitea `main` | local clone set |
|---|---|---|---|---|
| `foodchain` | `pj-foodchain` (26) | `F3 Foodchain`, not archived | `d802509 basic` | `main` `direction` `devlog` |
| `runsmoke1` | `pj-runsmoke1` (27) | `R Runsmoke1`, not archived | `595494e` | same |
| `runsmoke2` | `pj-runsmoke2` (29) | `R2 Runsmoke2`, not archived | `1803f8b` | same |
| `simpleshooter` | `pj-simpleshooter` (32) | `S2 Simpleshooter`, not archived | `03534fb Add collisions, score, HP HUD, and game over/restart loop` | `main` `direction` `devlog` |

**`simpleshooter` is the test project for Step 4.** It is the one with real
code behind it — a playable game loop, not a smoke-test placeholder — so a
plan written against it has something to read, which makes a grounded plan
distinguishable from a generic one. Its `direction/concept.md` exists, and its
channel already holds the Developer (8), autolab (11), agforge (13), cagent
(14) and Front (15).

**No provisioning was done, so there is no Deus Ex Machina note to leave**
for this step. `project_init` was not run; nothing was created by hand.

## The sweep reaches it

Subscription is the routing decision, so the check that matters is whether
the live listener's own filter accepts topics in that channel. Read-only,
through the bot's own credentials and the listener's own `topic_filter`:

```text
instance: autolab-agstudio1
  eligible: autolab-agstudio1 / how-to-request
  eligible: pj-simpleshooter / bmining-start
```

That is the whole realm as this listener sees it. `pj-simpleshooter` is
enumerated by the sweep and its topics pass the filter, which is what a
`workplan-…` topic there will need. The instance's own channel is swept whole,
as Step 1 set up.

Neither is *awaiting*: the last poster in both is the bot itself, and the
startup sweep reported `0 awaiting` accordingly. Nothing is queued to run.

The bot's subscriptions, unchanged by this step and not widened by anything:

```text
agents · autolab-agstudio1 · general · ops · pj-foodchain · pj-runsmoke1
pj-runsmoke2 · pj-simpleshooter · sandbox · work-r-1 · work-r2-1 · work-s2-1
```

## Not done in this step

- The three `work-*` channels from earlier phases are still subscribed. They
  hold `workrun-` topics, which is exactly what criterion 4 forbids firing —
  they are quiet because their last poster is the bot, and Step 4 must leave
  them that way. Not touched, and deliberately not archived: archiving them
  would be a change this phase did not plan and cannot then prove innocent.
- No project was created, so `project_init` is still unexercised in this
  phase's vocabulary.
