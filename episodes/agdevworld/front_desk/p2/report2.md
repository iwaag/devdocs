# Step 2 — character lore and relevant conversation evidence

agfront `9c002ba`. Nothing in pyagag or agdevworld changed; the shared
`remotes_for_home`, `topic_history`, `write_agents_md` and `run_role`
(`extra_meta`) are reused as they are.

## What exists now

- **The character is the lore, pinned per run.** `agfront/src/agfront/settings.py`
  reads the tree `agentroom-settings sync` leaves (`active.json`,
  `revisions/<sha>/manifest.toml`, `characters/<id>/lore.md`) — never Git,
  never the relay. `pin()` runs once at the start of a Front Desk serving,
  before any file is written; `characters.md` (every character's complete
  lore, name, nickname, which agents speak as it, the section that is Front
  marked **this is you**) and `settings.json` (the revision, machine
  readable, for step 3) are copied into the generation workspace. A sync
  that lands mid-run changes the next run only — pinned by a test that
  switches the active revision *during* a run.
- **Where the settings are** is instance configuration, because it is a
  local path: `AGFRONT_SETTINGS_ROOT`, else `settings_root` in
  `.local/instance.toml` (documented in `instance.example.toml`), else the
  sibling checkout's `../agdevworld/.local/settings` — which is this
  checkout's layout, so nothing local needed setting. Any other agent
  workspace on the host reads the same `current/` symlink the same way.
- **The guide no longer describes the character.** The gyaru paragraph is
  gone from `agent/guides/character_talk/guide.md`; it says the section
  marked as you in `characters.md` *is* who you are, that nothing in the
  guide overrides the lore, and what to do when the placement says no
  settings are available (say so in one plain line, answer in ordinary
  friendly Japanese). The p1 lessons stay: whole output is the dialogue, no
  mentions, professional language to agents, never post into
  `routine-<name>`, never open a `workrun-` topic, read before posting.
- **Evidence keeps its ids.** `agfront/src/agfront/evidence.py` renders the
  chatlog and every thread as `# #channel › topic` + one entry per real post
  `[name #id] sender <user id> · <time>` + body — the same conversations
  `format_chatlog` rendered, with what it dropped. The header says when a
  topic is now under its `✔` name (finished), when only the newest
  `HISTORY_MESSAGES` were fetched (with the `agentchat read --since`
  command that reads further), and when a thread **could not be read** —
  written as a file naming the failure rather than skipped, because a
  missing thread reads as "no news". The guide explains the citation form
  and tells Front that the work continues in `workrun-…` topics autolab
  names, to be read with `agentchat read` when they matter. Ordinary
  `front` runs are untouched (`format_chatlog`, `write_threads`).
- **The revision is on the record**: `run_front` passes
  `extra_meta={"settings_revision": …}`, which `agag.agent.run_role`
  stamps into `.local/agent/character_talk/run-NNNN.json`.
- The prompt gains one placement line for the desk role only:
  *The characters are placed beside it in "characters.md" (settings
  revision …); the section marked as you is who you are* — or *No
  character settings are available for this run (reason)*.

## Verification

`agfront` `uv run pytest -q` → **38 passed** (9 new): a desk run gets
`characters.md` with both lores and Front marked, `settings.json`, the
placement line and the stamped revision; the desk chatlog names the topic
and keeps ids while acks and selfnotes stay out; a callback whose remote is
under its `✔` name gets a thread that says so with the report's id
(`[Autolab #5203]`); an unreadable thread is written as such and still
named in the prompt; a bounded history says so; no settings → the run
still happens and is told why; a settings update reaches the next run and
not the one in progress; an ordinary front run is unchanged; the settings
root resolves from env, `instance.toml`, then the default; the guide names
`characters.md` and no longer carries the gyaru description.

Dry run on this host (reads only, no agent run, no post): pinned the real
revision `4f3b55f654c8` from the default path, rendered `characters.md`
(front = 姐さん, autolab = 親方, full lore), and built the evidence files
for `#front` › `front-desk-20260908-1600` with Front's own credential:
11 posts with ids 5171…; remotes found by `remotes_for_home`:
`#pj-ghtrends › workplan-trend6` (6 posts) and `#work-g-13 ›
workrun-task1-g-13`, which came back **under its ✔ name** — the thread
header reads *Now named `✔ workrun-task1-g-13` — resolved (✔): this
conversation is finished* with `[autolab-agstudio1 #5194] sender 11` and
the completion report. The prompt's three placement lines were as intended.

## Notes

- autolab's Zulip display name is `autolab-agstudio1` (user 11), not
  "Autolab"; the manifest maps by the roster's `agent` name (`autolab`),
  which `tools/agents.md` ties to the bot, and `[overrides.characters.autolab]
  senders = ["autolab-agstudio1"]` is available if a direct name match is
  ever wanted. Not set: step 3 has Front choose the character id itself.
- The running listener serves the p1 code until it is kickstarted (step 5);
  the guide is read per run, so its rewrite is live for the next Front Desk
  post already — with the old code that run would get the new guide but no
  `characters.md`, and would answer the "no settings" way. That is the
  reason the kickstart is not deferred past step 3's deployment.
