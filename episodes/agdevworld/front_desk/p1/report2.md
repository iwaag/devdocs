# Step 2 — `character_talk` and event-driven turns

agfront `1be5b8f` (pj-agdev `e21c88e`). Nothing in pyagag changed.

## What was added

- **`agents.toml`**: `profiles.character_talk` (claude_code on
  `anthropic/claude-sonnet-5`, front's model as the initial reference) and
  `roles.character_talk` with front's exact grant —
  `Read,Glob,Grep,Bash(agentchat:*),Bash(rtschedule:*)`. Its own profile so
  the Front Desk can be moved to another harness or model in
  `.local/agents.local.toml` without touching the ordinary entrance.
- **`agent/guides/character_talk/guide.md`**: the voice and the turn. Two
  voices — Japanese gyaru with emoji to the developer, ordinary professional
  language in every `agentchat send` — and a rule the old guide never
  needed: **no `@**name**` in the reply**, because `#front` is public and a
  mention there summons that agent into the conversation. The event-driven
  turn is spelled out: read, request, reply, exit; the next developer post or
  an agent's callback is the next run; `agentchat read --since` follows a
  topic across ✔ for judging evidence. `agentchat wait` is not mentioned
  (pinned by a test). It also records the ghtrends finding from planning:
  a routine's standing request is the developer's newest post in
  `routine-<name>`, not the topic's newest post.
- **`zulip_listener.py`**: `role_for(channel, topic)` — `character_talk`
  for a `front-desk-` topic, `front` otherwise — chosen from the **home**
  conversation, so `serve` (direct posts) and `handle_mention` (callbacks,
  which resolve home through `rootchat_home` and call the same `serve`)
  make the same choice without a second decision point. `front_prompt` and
  `run_front` take the role; the workspace is `…/<N>/character_talk/` and the
  run record `.local/agent/character_talk/run-NNNN.json`, so a Front Desk
  run is told from an ordinary one by where its evidence is.
- **`params/intro.md`**: says what a `front-desk-<conversation id>` topic
  is. Re-posting it to `#agents` is part of step 4's deployment.

No listener code routes the new topics: `front-desk-` is inside the `front-`
prefix the sweep already serves, which is why the plan recommended that
shape.

## Tests (agfront, 29 passed)

- a `front-desk-` topic runs `character_talk` with the character guide and
  not the front guide; its chatlog and reply are the ordinary ones;
- an ordinary `front-*` topic still runs `front` with the front guide;
- `role_for` is the prefix and nothing else — a remote `workrun-` topic
  never decides it;
- a callback whose root note names a Front Desk home runs `character_talk`,
  places the remote thread beside the chatlog, replies **at home** in the
  desk topic and writes the `[served]` mark there;
- a callback into an ordinary conversation keeps `front`;
- a failure names the role (`failed during character_talk: …`);
- the guide exists, names both voices, has `agentchat read --since` and no
  `agentchat wait`;
- `character_talk` resolves with front's grant and a profile of its own.

## Not done here

The running listener still serves the old code until step 4 restarts it
(`launchctl kickstart -k gui/$(id -u)/com.agdev.agfront-zulip`). The guide
itself is read from disk per run, so later wording changes need no restart.
