# sage p1 — step 1 report: generate and provision

`arxivsage` exists as a project, a GitHub repository and a Zulip identity.
Nothing serves yet; step 2 writes the listener.

## Command

Run from `~/projects`, exactly as the plan gives it:

```sh
AGAG_ZULIP_ADMIN_ENV=$PWD/pj-agdev/.local/zulip/provisioner.env \
  uv run --project pyagag agag init arxivsage --yes --provision --like agecho \
  --plan-prefix entrance- --run-prefix "" \
  --description "arxiv sage: answers questions about the papers in study-arxiv-trend; ask in entrance- topics"
```

One command generated and provisioned; no pyagag change was needed.

## What was provisioned

| thing | value |
|---|---|
| bot | `arxivsage-agstudio1-bot@agstudio.local`, **user id 20** |
| credentials | `arxivsage/.local/zulip.env`, mode 0600 (ignored) |
| own channel | `#arxivsage-agstudio1`, **stream id 92** |
| channel description | "arxiv sage: answers questions about the papers in study-arxiv-trend; ask in entrance- topics" (from `--description`) |
| `#agents` | subscribed (stream id 35) |
| instance name | `arxivsage-agstudio1` (`.local/instance.toml`, the hostname default) |

Both subscriptions were read back from the bot's own credentials
(`GET /api/v1/users/me/subscriptions`), not from the provisioner's output.

## Generated tree

```
arxivsage/
  .gitignore  agents.toml  instance.example.toml  pyproject.toml
  params/intro.md  params/channel.md
  agent/guides/entrance_front/guide.md
  src/arxivsage/{__init__,listener,intro}.py
  service/listen.sh
  .local/{instance.toml,agents.local.toml,zulip.env}     # ignored
```

Checks the plan asked for:

- **`--like agecho` landed.** `.local/agents.local.toml` carries the
  `claude_code` `command_glob` and no role overrides (agecho's overlay had
  none to filter). The glob resolves today to the VS Code extension's
  binary — two versions match it, `2.1.250` and `2.1.251`.
- **The guide stub is already in the right place.** `agag init` derives the
  guide directory from the plan prefix (`plan_prefix.rstrip("-")`), so
  `--plan-prefix entrance-` wrote `agent/guides/entrance_front/guide.md`,
  which is exactly where `agag.entrance` looks. No rename.
- **`--run-prefix ""` was treated as absent**, as the plan predicted
  (`args.run_prefix or _ask(…)` in `agag/init.py`), so the generated
  `listener.py` says `run_prefix="arxivsagerun-"`. Left as generated in this
  commit; step 2 deletes it — sage has no runs.

## Version control

`git init`, then one commit, then the public repo:

```sh
gh repo create iwaag/arxivsage --public --source arxivsage --push
```

`https://github.com/iwaag/arxivsage` — public, default branch `main`,
`origin/main` at `0a54c84`. The `agag_agent` Ansible role refuses non-GitHub
sources and the gitea mirror goes stale silently, so GitHub is the source of
record from the first commit, not from the first deployment.

`.gitignore` gained two lines before that commit, on top of the generated
`.local/ .venv/ __pycache__/`:

```
knowledge/
tostudy/
```

Written with a trailing newline (checked): a `.gitignore` appended to without
one silently merges the last existing pattern with the new line.

They are ignored for different reasons and the commit message says both: the
knowledge is *another repository*, and `tostudy/` is a queue whose files
others delete. Neither belongs to arxivsage's history.

## Note for later steps

The classifier blocked `gh repo create` on the first attempt; the developer
granted it by hand. Every later step commits *and pushes* (localrule.md), so
that grant is load-bearing for the rest of the phase.
