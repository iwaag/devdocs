# agag_builder p3 plan — `agag provision`: the bot is no longer human work

Goal: everything between `agag init` and a listener that answers Front is
done by an agent. The Zulip bot, its credentials file, its channels and its
introduction are created by `agag provision`, and autolab runs the whole
chain — init → provision → intro → listener — on request, so a new agag
agent is one `workplan-` topic away.

Success criteria:

1. `agag provision` (or `agag init --provision`) creates the bot, writes
   `<root>/.local/zulip.env`, subscribes it to `#agents` and its own channel
   (creating the channel, with a description), given one owner-class Zulip
   credentials file named by `AGAG_ZULIP_ADMIN_ENV`.
2. A dedicated `provisioner` Zulip user exists; its env file lives in
   `pj-agdev/.local/zulip/provisioner.env`. Nothing uses `developer.env`.
3. autolab, asked in a `pj-` channel to "create an agag agent named `agping`",
   generates it into the project's `main/`, provisions it, posts its intro,
   starts its listener, and reports. Then Front greets `agping-agstudio1`
   and relays the reply — no human step between the workplan post and the
   reply except answering autolab's questions.
4. `agag init`'s printed checklist shrinks to what is still human: the
   provisioner account (once, already done), Plane per-agent accounts (only
   if wanted), and making the listener permanent (launchd/ansible).

Decisions already made (discussion after p2):

- **Bot creation goes with the agent side (pyagag + autolab), not cagent.**
  cagent stays the side that distributes env files onto nodes (later
  episode, `agag_agent` ansible role generalizing `roles/autolab_node`).
  Creating a Zulip identity is a Zulip operation, so it lives next to the
  Zulip code.
- **Identity is a path, not a value** (p2's `AGENTCHAT_ZULIP_ENV` rule): who
  may provision is whoever has `AGAG_ZULIP_ADMIN_ENV` in their environment.
- **Plane stays on the shared key** (`pj-agdev/.local/plane-credentials.env`,
  as forge uses it). Plane has no user-creation API; per-agent accounts are
  out of scope.
- **One guard survives**: provision refuses when a bot with that email
  already exists. Owner-class keys can regenerate other bots' keys
  (`POST /bots/{id}/api_key/regenerate`), which silently kills a running
  agent — that is the irreversible side, so the check stays. Everything
  else (channel exists, already subscribed) is idempotent, not an error.
- No backward compatibility. `agag init`'s checklist text, the p1 report's
  manual recipe and anything in READMEs describing bot creation by hand are
  rewritten, not kept alongside.

Constraints: secrets in `.local/`; pyagag → push → `uv lock
--upgrade-package pyagag` in agautolab at least (the one that runs it) →
push → submodule pointer. Cost is not a concern.

## Facts checked at planning time

- **The recipe already ran once, by hand** (`p1/report4.md` "Checklist as
  executed"): `POST /api/v1/bots` as the Developer → bot user, creds to
  `agecho/.local/zulip.env` (copy in `pj-agdev/.local/zulip/agecho.env`);
  subscribed to `#agents`; `#agecho-agstudio1` created with a description,
  bot + Developer subscribed; `python -m agecho.intro`; `listen.sh`. That is
  the spec of `agag provision`, line for line.
- Zulip API: bots are created by a **user** key (`POST /api/v1/bots`, form
  fields `full_name`, `short_name`, `bot_type=1`; the response carries
  `api_key` and `user_id`... check the response shape against this realm —
  some versions return only the id and need a follow-up `GET /users/{id}`
  plus `POST /bots/{id}/api_key/regenerate` to obtain a key). Channel
  description edits are `PATCH /streams/{id}` and need a user with enough
  role — a bot gets HTTP 400 (standardize p10), an owner does not.
  `ZulipClient.create_channel` (`agag/zulip.py:325`) already creates +
  subscribes with `principals`; `subscribe_channels(names, principals)`
  covers `#agents`. Missing: `create_bot`, `update_channel_description`,
  `find_user_by_email`.
- `pj-agdev/.local/zulip/` holds one env per identity
  (`agecho.env … developer.env, developer.password, omni-agent.env`). The
  `developer.env` is the owner key used in p1. The provisioner is created
  with `manage.py create_user` in the container (Step 1); the API's
  `POST /users` would need `can_create_users` granted via `manage.py` and
  is not worth it for one account.
- `agag init` (`pyagag/src/agag/init.py`): `--yes` exists; the checklist is
  `checklist(plan)` at line ~166; `.local/instance.toml` is generated; no git
  since `2827f3a`. `ZulipClient.from_env(path)` reads
  `ZULIP_URL/EMAIL/API_KEY` — the same three keys `provision` must write.
- autolab: supercoder runs with `skip_permissions=True` under claude_code and
  has `Bash(uv:*)`, `Bash(sh:*)`, `Bash(agentchat:*)`; `agag` is in its venv
  bin and on the run's PATH (`agag.agent.chat_environment`). `.local/` of
  the generated agent is ignored by the project repo's `.gitignore` (from
  the template) so `zulip.env` never reaches Gitea/GitHub.
  `AgentSpec.extra_environment` is where autolab adds `AGAG_ZULIP_ADMIN_ENV`.
- autolab's project `main/` is a Gitea clone (`agstudio.local:3000/autodev/<slug>.git`);
  since `agag init` no longer `git init`s, generating into `main/<agent>/` is
  one folder in that repo. That is the intended shape (p2 check).
- The generated listener is a foreground process; autolab's supercoder can
  `nohup`/`setsid` it for the test but it dies with the node. Making it
  permanent is checklist item 4 and out of scope.
- `claude` on PATH: the generated agent needs `.local/agents.local.toml`
  with the claude_code `command_glob` (p1 failure 2). autolab can copy its
  own (`agautolab/.local/agents.local.toml`) — say so in the workplan, or
  make `agag init` accept `--like <sibling root>` to copy it (small, worth
  it; p1 marked it as a candidate).

## Step 1 — the provisioner account (human, once)

No real e-mail is needed: the realm already runs on `@agstudio.local`
addresses. Create the user inside the Zulip container, which skips e-mail
confirmation and the (unconfigured-SMTP) invite flow entirely; the shape is
the one `lighter_agag_listen/p1/report4.md` used for `manage.py shell`
(`./manage.py` wrapper does not forward env — use `docker compose exec`):

```sh
cd pj-agdev/.local/zulip-selfhost
docker compose exec -T -u zulip zulip \
  /home/zulip/deployments/current/manage.py create_user \
  --realm agstudio.local provisioner@agstudio.local "Provisioner" \
  --password-file /dev/stdin <<< '<password>'
docker compose exec -T -u zulip zulip \
  /home/zulip/deployments/current/manage.py change_user_role \
  --realm agstudio.local provisioner@agstudio.local admin
```

Then `POST /api/v1/fetch_api_key` (email + password) gives the API key;
write `pj-agdev/.local/zulip/provisioner.env` (`ZULIP_URL/EMAIL/API_KEY`,
plus `ZULIP_CA_BUNDLE` if the other env files carry it). Verify with
`curl -u email:key $ZULIP_URL/api/v1/users/me`.

Administrator is enough for bots and stream edits; owner is not required.
If bot creation is refused, check the realm's `bot_creation_policy`. The
`--realm` value is the subdomain — confirm it against an existing user's
realm if the command complains.

## Step 2 — `ZulipClient` additions (pyagag)

In `agag/zulip.py`: `create_bot(full_name, short_name) -> dict` (returns at
least `user_id`, `email`, `api_key`; do the follow-up calls if the realm's
response lacks the key), `user_by_email(email) -> dict | None`,
`update_channel_description(stream_id, description)`. Tests against a fake
`call`, as the existing ones do. Hint: `bot_type=1` is generic bot; the bot
email is `<short_name>-bot@<realm domain>` — read it from the response
rather than composing it.

## Step 3 — `agag provision`

`pyagag/src/agag/provision.py`, registered beside `init` in `agag.cli`.

Inputs: the project root (default `.`, must hold `agents.toml` and
`.local/instance.toml` — the instance name comes from there), and
`AGAG_ZULIP_ADMIN_ENV` (`--admin-env` overrides).

Does, in order, each step idempotent except the first:

1. `user_by_email` for the bot email → if it exists, **stop** with a message
   naming the existing user (the one guard). Otherwise `create_bot`.
2. Write `<root>/.local/zulip.env` with `ZULIP_URL` (from the admin env),
   `ZULIP_EMAIL`, `ZULIP_API_KEY`, `ZULIP_CA_BUNDLE` if the admin env has one.
   Mode 0600.
3. Subscribe the bot to `#agents`.
4. `create_channel(instance, description, principals=[bot, admin])` — the
   description comes from a new `params/channel.md` in the template
   (one paragraph; `{instance}` substitution like `intro.md`), or from
   `--description`. If the channel exists, `update_channel_description`.
5. Print what was done and the next commands: `uv run python -m <agent>.intro`,
   `service/listen.sh`.

Optional and recommended: `agag init --provision` runs Step 3 right after
generating, and `--like <root>` copies `.local/agents.local.toml` from a
sibling. Then the agent-side sequence is two commands.

Rewrite `checklist()` to list only what is still human (see criterion 4).
Update `README.md` and `docs/` where they describe the old manual steps.

## Step 4 — autolab gets the key and the know-how

- `agautolab/src/agautolab/role_run.py` (or `SPEC.extra_environment`):
  `AGAG_ZULIP_ADMIN_ENV=<pj-agdev>/.local/zulip/provisioner.env` into the
  supercoder run environment. Path, not value.
- `uv lock --upgrade-package pyagag` in agautolab; push; pj-agdev pointer.
  agforge/agfront may stay on `a8fa481` — nothing they run changed — but
  bumping all three in one commit is cheaper to reason about later.
- Guide: one or two lines in the supercoder guide (or a new
  `agent/guides/…` entry, whichever autolab's guide layout uses) saying that
  `agag init <name> --yes --provision --like <sibling>` creates an agag agent
  and that `agag --help` is the usage. Tool Giving, not a procedure.
- Restart `com.agdev.agautolab-zulip`.

## Step 5 — live check

Post in an existing `pj-` channel (`pj-runsmoke1` has the fixtures from p2):

> workplan-agping: Create a new agag agent named `agping` in this project
> with `agag init`, provision its Zulip bot, post its introduction and start
> its listener in the background. Report the instance name and the intro
> message id.

Let autolab ask whatever it asks. When its report arrives, ask Front:
"say hello to agping-agstudio1 and tell me what it replied". Expected
transcript shape is p1 Step 4's log (`report.md` of p1).

Where it will probably fail first, in order: the bot-creation response shape
(Step 2 follow-up calls), `claude` not on the generated agent's PATH
(`--like`), the listener started in the run's foreground and killed with the
run (`nohup … &` with output to `.local/out/`), `#agents` harvest timing
(Front reads `tools/agents.md` at run start — call Front after the intro
post, as in p1).

## Step 6 — record

`p3/report.md`: the provisioner setup as done, the provision transcript,
autolab's workplan/workrun log excerpts and every question it asked (those
are the next guide lines), Front's exchange with agping, and the final
checklist text. Note whether `agping` stays (fixture #2) or is removed
(archive its channel; deactivate its bot via `DELETE /bots/{id}` — this is
the one place the owner-class key does something irreversible on purpose).

## Out of scope

- cagent distributing env files onto nodes / the `agag_agent` ansible role
  (next episode; this phase produces the files it would distribute).
- Per-agent Plane accounts.
- launchd/ansible for the new agent; agecho under launchd.
- Front posting its own intro (p2 left open).
