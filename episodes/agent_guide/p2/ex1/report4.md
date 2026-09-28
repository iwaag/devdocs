# agent_guide p2 ex1 — step 4: an introduction for the Comfy Notifier

pj-agdev `059f4f2` (comfynotify), agautolab `5b2765e` (worker guide). Posted
as `#agents › intro-comfynotify-agstudio1` #15854 (2026-09-28 15:34Z).

## The introduction

`comfynotify/params/intro.md`, posted through `agag.intro.post_intro` like
every agent's:

- **Who it is**: "A tool, not an agent: I hold no conversation and answer no
  question." It watches one ComfyUI job and says in your topic when the job
  has ended, so the run can finish instead of waiting.
- **The one command**, quoted in a code fence, the only place the bot's
  mention appears:
  `@**Comfy Notifier** watch <prompt_id> [a note…]`, posted in the topic
  where the result should land, as a normal message, not the CLI.
- **What comes back**: a 👀 reaction and no post (a post would start the
  topic's owner again). When the job ends, the two lines, and the
  instruction to read `GET /history/<prompt_id>` for the outputs; that
  post is what brings the owner back.
- **What is not a command**: public channels only. Every line that names
  the notifier is read as a command, so a mention inside a sentence is a
  command it cannot read. That gets one line back, once per topic.
  (`commands.parse_command` tries every mentioning line; a sentence's
  first word is not `watch`.) Inside a fence it is not a mention at all.
  It starts no job and holds no queue.

It carries a **roster block with no channel and no prefixes** (pj-agdev
`6556e29`). The first post (#15854) had none, on the theory that a roster
would put the notifier among the instances the relay expects. That was wrong
the other way round: the relay's operation room lists **every** `intro-`
topic as an instance, and one without a roster block becomes an `unknown`
row ("an agent whose work cannot be read"). The cost gauge also named it
`missing`, as it does agobserver, archsage and cagent, which have no root
configured there. After the re-post, `/ops` shows `comfynotify-agstudio1`
with `roster: intro`, no channel, no prefixes, `state: ok`, all counts 0.

`agentchat intro` now lists it ("comfynotify-agstudio1 — A tool, not an
agent: …"). `write_agents_md` harvests it into every run's
`tools/agents.md` with the other introductions.

## When it is posted

The plan said "post it at start-up like the agents do". The agents do not
post at start-up: each has a `python -m <agent>.intro` (archsage `archsage
intro`) run by hand after a change, and a kickstart leaves the board as it
was. For the notifier it is at start-up, as asked, made idempotent. The
daemon compares the board's newest post in its `intro-` topic with the file
(the date/revision stamp aside) and posts only when they differ. So a
restart does not stack identical posts, and an edit to `intro.md` reaches
the board at the next restart. `comfynotify intro` posts unconditionally
(`--if-changed` for the daemon's rule). A failure to post is logged and
never stops the daemon.

The instance name is `.local/instance.toml` (`comfynotify-agstudio1` on this
host, ignored; `instance.example.toml` shows the shape) or
`COMFYNOTIFY_INSTANCE_NAME`, through `agag.instance`, as the agents do.

Checked live: after `kickstart -k`, the log reads `command intake on as
Comfy Notifier (21)` / `introduction posted`. Then `comfynotify intro
--if-changed` answered `introduction unchanged on the board`. No ticket was
in flight at the restart. The notifier skips its own posts, and the mention
is fenced, so the post itself started nothing.

## The pin

comfynotify pinned pyagag `0af761d`; it is now on `174b74e` like the
agents. Its suite: 26 → **28 passed**.

## autolab's worker guide

The paragraph (report1 of p2: WR14, trial key `comfy`, autolab `3dcc590`)
went from 8 lines of usage to a pointer:

> When a ComfyUI generation takes minutes, do not wait for it. Submit it,
> hand it to the Comfy Notifier with its one command posted **in this
> topic** as a normal message, not by running its CLI (2026-09-01: a run
> did, and nothing came back here), record in your report what is pending
> and what to do with its result, then finish. The notifier's introduction
> on the board is the command, what comes back and where it works.

Kept because a trial is behind it: post the command in this topic rather
than running the CLI (the 2026-09-01 run, now named in the text as styles.md
asks), and finish instead of waiting. Moved to the introduction: the
command's exact form, the reaction, the two-line callback,
`GET /history`, public channels only, and quoting in a fence. The guide
file went from 5 211 to 5 105 bytes. Guides are read per serving, so this
is live for the next worker serving. agautolab's suite: **334 passed**.

## Tests

`comfynotify/tests/test_intro.py`:

- it is posted once, then again only when forced or changed;
- its roster names the bot and declares no channel and no prefixes;
- the one fenced line in `intro.md` parses as a `watch` command;
- outside backticks and fences the post names nobody (`@**`);
- it says it is not an agent.
