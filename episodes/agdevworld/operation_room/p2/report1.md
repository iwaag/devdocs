# Step 1 — the intro contract now carries a roster

The observer this phase builds has to reproduce `agag.agent.topic_filter` in
order to say who owes a reply, and p1 proved it cannot read either half of it:
the sweep prefixes are compiled into each agent's `AgentSpec`, and the
instance's own name lives in that node's ignored `.local/instance.toml`.
Guessing them was p1's most expensive mistake — 7 phantom stalls for Front, 59
for a second autolab bot, 66 rows of pure fiction.

So the introduction says it. It was already the contract for *where to write*,
already re-posted after a behavior change; the roster rides on that habit
rather than inventing a second one.

## The format

A fenced block at the end of the post, after the agent's own prose and before
the `Posted:` stamp:

````
## Roster

For an observer, not a reader. …

```agag-roster
schema: ag.agent-roster.v1
instance: front-agstudio1
agent: front
bot: Front
bot_id: 15
channel: front-agstudio1
prefixes: front-
```
````

Fenced, because a fence is not prose a human has to skim past and — the same
property that lets a report quote `@**Comfy Notifier** watch` without firing
it — nothing inside one is a mention. `key: value` because the whole thing is
a single-pass parse and the observer should need no library to read it.

Documented in `pyagag/docs/agent-roster-v1.md`, which is the reader's
reference; the post carries its own two-line explanation of what it is for.

**Four fields and a name each earn their place:**

- `bot` is the Zulip **full name**, and it is the field the design would most
  easily have got wrong. The mention route matches `@**<full name>**`, that
  name is chosen at provisioning time, and for one live agent it is not the
  instance name at all: `Front` is the bot of `front-agstudio1`. An observer
  deriving it from the instance would have missed every mention of Front on
  the realm.
- `channel` is stated **even when no such channel exists**. It is what
  `topic_filter` matches (`channel == instance_name`), not a claim about the
  realm. Front declares `front-agstudio1`; no channel of that name is on the
  board. p1's first pass invented the opposite ("Front owns `#front`") and got
  seven stalls for it. The poster states the rule; checking the realm is the
  reader's job.
- `prefixes` and `instance` are the two values that have no read path at all
  from outside.
- `bot_id` saves the reader a `users` call and gives it a key that survives a
  rename.

## Where it is generated

`agag.agent.roster_for(spec, client)` — the `AgentSpec` for the channel and
prefixes, the bot's own Zulip profile for the name and id. Nothing is written
by hand and nothing is configured twice, so a re-post is current by
construction. `intro_main` calls it; an agent that already posts its
introduction gained the roster without changing a line of its own code.

A test in `tests/test_agent.py` asserts the block against `topic_filter`
itself: every prefix the roster declares must match under the listener's own
filter, and the declared channel must admit any topic. If the routing changes
and the roster does not, that test fails rather than the operation room
quietly going wrong.

## The rule for the reading side

`parse_roster` returns `None` for a post with no block, and **the caller must
keep that as `unknown`**. An intro without a roster is an instance whose
routing nobody knows — not an instance with no prefixes. A default here would
re-create exactly the guessed roster this block abolishes. It is asserted as a
test, stated in the doc, and Step 4 renders it as its own colour.

## What was re-posted

Every instance's own mechanism (`python -m <pkg>.intro`), run in its own
checkout against its own credentials. Nothing was written on an agent's behalf.

| instance | roster | how |
|---|---|---|
| `front-agstudio1` | ✅ | `agfront.intro` on agstudio |
| `autolab-agstudio1` | ✅ | `agautolab.intro` on agstudio |
| `agforge-agstudio1` | ✅ | `agforge.intro` on agstudio |
| `arxivsage-agstudio1` | ✅ | `arxivsage.intro` on agstudio |
| `agecho-agstudio1` | ✅ | `agecho.intro` on agstudio |
| `agecho-agautolab1` | ✅ | `agecho.intro` on the agautolab1 node, listener restarted after |
| `agping-agstudio1` | ❌ | **no project left to run it from** |

**Front's introduction is new.** p1 recorded that Front had no `intro-` topic,
which is why the roster could not be one clean source and the probe carried a
hard-coded table beside it. `intro-front-agstudio1` now exists, and the busiest
agent on the realm — 326 served notes — is on the board with the rest.

**`agping-agstudio1` is a fixture that outlived its code.** Its bot and its
`intro-` topic are on the realm; its project is on neither machine (it was
created inside a runsmoke mission workspace that has since been cleaned up), so
there is no existing mechanism to re-post with and the plan forbids writing one
for it. It stays on the board with no roster, which is the honest answer and
makes it Step 4's live test of rule 3: an agent whose routing is unknown must
not be drawn as an agent with nothing to do.

## Deus Ex Machina note

The Omni Agent ran each instance's `intro` module and, on the agautolab1 node,
its `git pull` / `uv sync` / service restart. The text and the roster are the
agents' own; the *triggering* is work an agent could do for itself when told
its introduction is stale — handoff candidate.

## Changed

- `pyagag` — `agag/intro.py` (`Roster`, `roster_block`, `parse_roster`,
  `intro_text(roster=…)`), `agag/agent.py` (`roster_for`, `intro_main`),
  `docs/agent-roster-v1.md`, tests. 454 tests pass.
- `agfront`, `agforge`, `agautolab`, `arxivsage`, `agecho` — `uv.lock` moved to
  the new pyagag.
