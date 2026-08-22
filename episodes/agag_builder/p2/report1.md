# agag_builder p2 — step 1: agfront's instance name

Decision: **(a)** — `front-agstudio1`, one file, no new `AgentSpec` notion.

## What was found

- The plan's "agfront already introduces itself in `#agents`" is not the
  case: `#agents` holds `intro-agecho-agstudio1`, `intro-autolab-agstudio1`,
  `intro-agforge-agstudio1` and nothing for Front. Front has no
  `params/intro.md` and no `intro.py`. There was no name to keep, so the
  name follows the convention (`<agent>-<host><N>`).
- Front's entrance is `#front` ("The Developer's conversation with Front"),
  served through its `front-` topics; a channel named `front-agstudio1`
  does not exist. With `plan_prefix="front-"` the skeleton's
  `topic_filter` (`channel == instance_name() or topic.startswith(prefixes)`)
  gives exactly today's sweep: every `front-…` topic wherever Front is
  subscribed, plus an own channel that is simply absent.

## Done

- `agfront/.local/instance.toml` — `name = "front-agstudio1"` (local only).
- `agfront/instance.example.toml` — the shape, committed: agfront `fbab21a`.

No intro re-post: there was none. Whether Front should introduce itself on
`#agents` at all is left open (it is the Developer's agent, not a service
other agents are meant to call); Step 3 adds `intro.py` so the command
exists, but nothing is posted until somebody decides it should be.
