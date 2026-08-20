# agent_standardize p2 plan — agfront learns agforge from its intro

Goal: the first proven case of one agent recognizing and using another.
agfront, asked by the developer in a `front-*` topic, understands from
agforge's intro how to request an image and opens a `create-` topic in
`agforge-agstudio1` on the developer's behalf.

Success criteria (all three, nothing more):

1. A `front-*` conversation: developer asks for an image → Front proposes how
   (grounded in the harvested agent list) → developer permits → Front opens a
   `create-…` topic in `agforge-agstudio1` via `agentchat` and reports back.
2. agforge acknowledges the request in that topic. **`runcreate-` and the
   generated result are NOT required** — create is the scheduling half and
   that is enough.
3. Attributability: no agforge routing hardcoded in agfront's code or guide.
   The channel/topic knowledge reaches Front only through the harvested intro
   list. (Grep-level check: `agforge-agstudio1` appears in neither
   `agfront/src` nor the guide.)

Decisions already made ([braindump.md](braindump.md) + discussion):

- Front stays a singleton named `front`; no instance label, and deliberately
  **no `intro-front` topic in `#agents`** — Front is developer-only and
  whether other agents may call it is an open question, not a p2 answer.
- Zulip access for agents-at-run-time is a shared `agentchat` CLI in pyagag,
  so every agent gets the same read/write path later.
- The agent list is **generated mechanically by the handler right before
  each Front run** — a deterministic script assembling intros by string
  manipulation, not an agent step, not a hand-maintained file. Marked for
  later rethink; this is the first cut.
- p1's `create.md` command-file route and the `#general` derived-topic post
  are removed. Breaking-change phase; agfront was already allowed to break.
- The old safety ("Front never names the channel/topic") is deliberately
  retired; its replacement is the intro: Front finds the right entrance by
  reading, which is exactly the capability p2 exists to prove.

Constraints (deliberately minimal):

1. Secrets stay in `.local/`; the run gets its Zulip identity via an env
   variable pointing at the front bot's env file — never inline values.
2. pyagag changes: commit → push to GitHub → `uv lock --upgrade-package
   pyagag` in agfront (localrule.md). Never a local path/gitea source.
3. Experimental LAN realm; no further hardening asked for.

## Step 1 — `agentchat` in pyagag

- New module (e.g. `agag/chat.py`) + `[project.scripts] agentchat = …`.
- Subcommands, minimum: `send <channel> <topic> <text…>`,
  `read <channel> <topic>` (recent messages, sender + timestamp),
  `topics <channel>`. No `wait`/polling — deferred; p2 stops at create.
- Credentials: `AGENTCHAT_ZULIP_ENV` names the bot env file;
  `ZulipClient.from_env` does the rest. Whoever's env is set is who speaks —
  that is the whole identity model.
- Zulip lets a bot post to and read any public channel **without
  subscribing**, so agentchat must not touch subscriptions (subscription
  stays the sweep-routing decision, untouched this phase).
- `--help` is the tool's only documentation and the guide points agents at
  it. Write it as usage doc with examples — self-describing per Tool Giving;
  a bare argparse synopsis would be an Unexplained Chainsaw.
- Deterministic tests, push, re-lock agfront (and nothing else this phase).

## Step 2 — Intro harvest before each run

- A deterministic function in agfront (handler side): fetch the `#agents`
  channel's `intro-*` topics, take each topic's **latest** post (intros are
  append-only; the newest carries the current revision stamp), and assemble
  one markdown file by string operations — a header per agent, body verbatim,
  plus a generated-at line. No model call anywhere in this path.
- The listener writes it to the run workspace as `tools/agents.md` right
  before invoking the role, so the guide's "read files in tools/" finds it
  and every run sees the intro state of that moment. The existing generation
  workspace (`.local/topics/<channel>/<topic>/<N>/front/`) is the natural
  place; committed files stay clean.
- Reuse the listener's existing `ZulipClient` (front bot can read public
  `#agents` unsubscribed). Skip `✔`-resolved and non-`intro-` topics.
- Empty harvest (no intros found) should not crash the run: write the file
  with an honest "no agents known" line — Front can then refuse gracefully.
- Marked for rethink later (pull vs. push, caching, per-agent files); do not
  design that now.

## Step 3 — Rework the Front run

- Remove the `create.md` branch and the `#general` post from
  `zulip_listener.py`, and the `Write`-for-`create.md` tool carve-out in
  `role_run.py`. The `front-*` sweep + `serve_topic` loop stays as is.
- Give the run what the new guide assumes: `agentchat` executable on PATH
  (the uv environment provides it once agfront depends on the new pyagag)
  and `AGENTCHAT_ZULIP_ENV`→`agfront/.local/zulip.env` — set the exact env
  var name consistently with Step 1. Bash (or an equivalently direct way to
  run the command) is allowed; keep restrictions minimal, this is the
  experiment.
- Multi-turn shape needs no new code: guide's ask-permission-first flow works
  as consecutive runs of the same topic (`serve_topic` re-checks after each
  run). Pin in a test only what agfront decides: the harvest file lands in
  `tools/`, and the run env carries the agentchat identity.
- Guide is already rewritten; check implementation against it, not vice
  versa. Reload ritual: `launchctl kickstart -k
  gui/$(id -u)/com.agdev.agfront-zulip`; log at
  `agfront/.local/out/zulip-listener.log`. Every `front-*` post is one paid
  run — fine, do not economize.

## Step 4 — Prove it and report

1. Developer posts an image wish in a fresh `front-*` topic; expect a
   grounded proposal naming forge's entrance (from the harvest, not from
   code).
2. Developer replies with permission; expect the `create-` topic to appear in
   `agforge-agstudio1` (Front's own posting identity), forge's ack, and
   Front's honest report of what was sent and where.
3. Run the attributability grep (success criterion 3) and quote it.
4. `report.md`: message links for both sides, the grep, and deferred items —
   `agentchat wait`/result round-trip, harvest redesign, `intro-front`
   decision, agentchat rollout to autolab/cagent, `runcreate` execution.
   Update `agfront` docs and `.local/devenv.md` (the `#general` route
   paragraph is now history).
