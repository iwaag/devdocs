# zulip_command — Plan

Braindump: `braindump.md`. The desire: any agent should get a ComfyUI job
watched by **posting one Zulip line**, instead of being handed the
`comfynotify` tool — hand-distributing a CLI to every agent is management
the notifier was supposed to remove. And: see whether the command survives
the Front relay.

Decision from the check that preceded this plan: **not a new "@command"
convention — a native bot mention.**

```
@**Comfy Notifier** watch <prompt_id> [free text, kept as the note]
```

posted in any public-channel topic. The notifier tickets that job with the
**posting topic as the callback destination**, acknowledges with an emoji
reaction, and later posts the existing two-line terminal record there. No
LLM anywhere, same as the notifier itself.

Experimental, non-public environment; destructive phase, no backward
compatibility. **MUST NOT** lines are the only prohibitions; everything else
is advice the implementer may override with a stated reason.

## Why this shape, in one paragraph

Every agent can already post to Zulip; none of them reliably has
`comfynotify` on PATH (`advance_mediagen_study` p6 ex1 measured autolab's
runs seeing neither the binary nor `AGFORGE_COMFYUI_URL`, mechanism still
undiagnosed). A mention is the one interface that needs no handover, and
pyagag already treats "a mention reaches a bot losslessly, even across
downtime" as a solved problem. The command carries only the `prompt_id`, so
the ComfyUI URL question moves to the daemon, where it belongs.

## Facts the implementer should not have to rediscover

- **`ZulipClient.mentions()` (pyagag `agag/zulip.py`) is the intake.** It is
  the `is:mentioned` narrow: it sees mentions in public channels the bot is
  not subscribed to, and a mention that arrived while the daemon was down is
  still returned after restart. Filter exactly as `sweep_rootchats`'s
  consumer does: `type == "stream"`, not our own `sender_id`, skip
  `is_selfnote(content)`, skip topics already resolved (`✔ ` prefix) if the
  job is to post there — actually do *not* skip resolved topics: follow the
  rename the way `agentchat` does, a command posted just before a resolve
  must still be honoured.
- **The mention keeps being returned forever.** Answering a mention does not
  consume it (`agent_standardize` p8/p9 learned this the hard way), so the
  daemon needs its own high-water mark — persist the highest processed
  message id and ignore anything at or below it. One JSON state file beside
  the tickets; the archived-ticket discipline (never post twice across a
  restart) applies to command intake identically.
- **An ack must not be a post.** A bot post into a `workrun-`/`assetrun-`
  topic re-serves its owner — that is the resume mechanism itself — so a
  "watching…" reply would wake the agent early and burn a paid run. Zulip
  reactions are not messages and trigger nothing:
  `POST /messages/<id>/reactions` with `emoji_name` (say `eyes`). pyagag has
  no reaction helper; one small `ZulipClient.add_reaction(message_id,
  emoji_name)` there is the clean home for it (pyagag change, push, bump the
  dependants), or call the endpoint from comfynotify directly if touching
  pyagag is not wanted this episode. Either is fine; say which.
- **A malformed command is the one case that *should* post back** — the
  poster needs to know, and waking it is correct. Keep it to one line
  (`comfy command not understood: …`), no mention of the poster needed —
  the post itself re-serves the topic owner.
- **Mentions inside code fences are not mentions.** Zulip does not render
  them and does not set the `mentioned` flag, so "quote the command in a
  code fence when discussing it" is the whole loop-prevention story for
  relays and reports. Conversely a *live* mention inside a relayed Front
  post fires normally, and the callback lands where that post landed —
  which is what the braindump wants to try.
- **Private channels are out of reach** (`#front` included): the narrow only
  returns messages the bot can read. Public `work-*` / instance channels
  are where every run topic lives, so this costs nothing today; note it in
  the usage line rather than engineering around it.
- comfynotify today: tickets in `.local/tickets/`, daemon sweep every 5 s,
  two-line terminal post, full record archived in `done/`. The daemon's
  launchd env carries only `AGENTCHAT_ZULIP_ENV`
  (`.local/zulip/comfy-notifier.env`, the bot Zulip credentials — the same
  file a `ZulipClient` needs for `mentions()`). `comfynotify watch` (CLI)
  stays: it is the daemon's own test surface and the no-Zulip path.
- Dependency route if pyagag is used in-process:
  `pyagag = { git = "https://github.com/iwaag/pyagag.git", branch = "main" }`,
  as agforge/agautolab already declare. Reuse `ZulipClient.from_env`.
- The guides to touch afterwards:
  `agautolab/agent/guides/workrun_supercoder/guide.md` and
  `agforge/agent/guides/assetrun_generator/guide.md` — both currently teach
  the CLI. Guides are read per run; no listener restart needed.

## Prohibitions

- **MUST NOT ack a valid command with a message post.** Reaction or nothing.
- **MUST NOT process the same command twice across a restart** (double
  ticket → double callback → double serving; the high-water mark plus the
  existing ticket-archive rule must cover every crash window).
- **MUST NOT put the ComfyUI host, IP or model filenames in tracked files.**
  The default URL goes into the launchd plist env
  (`__PROJECTS_ROOT__`-templated, values live only in the installed copy)
  or is read from `agforge/.local/.env`; tracked code sees an env name.

Everything else — command grammar details, emoji choice, state-file format,
pyagag-or-direct for the reaction — is the implementer's call.

## Steps

### Step 1 — command intake in the daemon

Add a mention sweep to `comfynotify`'s daemon loop (same 5 s cadence, same
process): read `mentions()`, drop already-processed ids, parse
`watch <prompt_id>` (accept `watch_comfy` as an alias since the braindump
spelled it that way; trailing free text becomes `note`), write a normal
ticket with `channel`/`topic` = the command post's location, react to the
command post, advance the mark. Unparseable command → the one-line error
post. The ComfyUI URL comes from the daemon's own env
(`AGFORGE_COMFYUI_URL`, added to the plist template) since the command
carries none.

Tests in the existing suite's style: a fake mentions feed → ticket written
with the right destination and note; alias accepted; junk command → error
post and no ticket; the same feed replayed → nothing happens; a mention
older than the mark → nothing; a code-fenced "mention" never reaches the
daemon (that one is a documentation assertion, not a test — Zulip filters
it before the narrow).

Done when: tests pass, `launchctl kickstart -k`, and a hand-posted command
in a scratch topic gets a reaction within ~10 s, a ticket on disk, and the
two-line callback when the job finishes — with a real (finished) prompt_id
this is one sweep.

### Step 2 — the three failure shapes, live

Same scratch topic, three posts: a nonsense prompt_id (`lost` after three
polls), a valid command while ComfyUI is unreachable if cheap to arrange —
otherwise skip with a note (`unreachable` is already unit-covered), and a
malformed command (error post). Then restart the daemon mid-wait on a real
running job: exactly one reaction, one callback, no re-processing of any
earlier command in the topic.

Done when: each produces exactly one visible outcome and the log tells the
story; `report2.md` shows the posts as they appeared.

### Step 3 — hand the vocabulary over, retire the CLI from the guides

Replace the `comfynotify watch` paragraph in both guides with the mention
line:

> When a ComfyUI generation takes minutes, do not wait for it. Submit it,
> post `@**Comfy Notifier** watch <prompt_id>` **in this topic** as a
> normal message, record in your report what is pending, then finish. The
> notifier reacts to your command, and posts back here when the job ends —
> two lines naming the state and the `prompt_id`; read
> `GET /history/<prompt_id>` yourself for outputs. Public-channel topics
> only. When *quoting* the command rather than issuing it, put it in a code
> fence.

Update `.local/devenv.md` (command syntax, state file, the plist's new URL
env) and the two-line note in `devdocs/README_DEV.md`. The braindump says
command syntax is taught per-request in chat for now — so do **not** build
an `#agents` introduction or entrance for the bot; it stays a tool.

Done when: both guides carry the new paragraph and neither mentions the CLI.

### Step 4 — prove it through the relay

The braindump's experiment. One real generation driven the normal way (a
`workrun-` task via Front is the faithful reproduction of ex1's shape; an
`assetrun-` on agforge is an acceptable cheaper stand-in), where the fire
tells the run to use the mention command — and the command string travels
**through Front's relay** as part of the instructions. Watch for exactly
the relay hazards this design predicts:

- Front's relay post itself containing a live mention (it fires from
  Front's topic — harmless if Front relays into the run topic, noteworthy
  either way; record where the callback landed).
- The paraphrase "correcting" the mention into prose, the same class as
  p5's substitution — the fire should mark the line as a literal.
- The run posting the command inside a code fence out of tidiness, which
  silently disarms it. If that happens, the reaction never appears — which
  is the visible symptom the ack exists to provide; note the recovery.

Measure the same three timestamps as ex1 (job end → callback post → second
serving) and the run count. Read `transcript.jsonl` to confirm the run
posted the command and did not poll.

Done when: one dataset-producing (or asset-producing) job has round-tripped
agent → mention → callback → second serving with no human post between
command and callback, and `report4.md` has the timeline.

### Step 5 — report

`report.md`: whether one Zulip line now replaces the tool handover; the
ack/no-post rule as observed; the relay result (did the command survive
Front, and what fired where); what remains of the PATH problem (it should
now be moot for consumers — say so explicitly if true, and whether the
`extra_environment` mystery is still worth diagnosing at all); costs; Deus
Ex Machina lines.

## Out of scope

- Any second command (`status`, `cancel`, `free`). The grammar leaves room;
  build none of them until a run asks.
- Notifying about anything but ComfyUI; multi-backend URL routing (one box,
  one env value today).
- An entrance, introduction, or channel for the Comfy Notifier bot — it is
  a tool, and the moment it needs an introduction it has become an agent.
- Diagnosing the `extra_environment` PATH loss — this episode makes it
  irrelevant to notifier consumers; diagnose it only if something else
  still needs that PATH.
