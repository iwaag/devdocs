# Step 3 report — `agag init`

pyagag commit `cecd21b` (pushed). 400 tests pass.

## What was added

- `agag/cli.py` → `[project.scripts] agag = "agag.cli:main"`; `agentchat`
  untouched.
- `agag/init.py`: `Plan` (agent, instance, plan/run prefixes, roles, profile,
  dest), `plan_from_args` (flags first, then a prompt with a default when
  stdin is a tty, `--yes` skips), `files(plan)` (path → content),
  `generate` (refuses an existing root; `git init` + `git add -A`, no
  remote), `checklist(plan)` printed at the end.
- `agag/templates/*.in` rendered with `string.Template`, shipped via
  `importlib.resources` (hatch `force-include`).
- `tests/test_init.py` (5): defaults, name/role validation, the exact file
  list, the listener importing with its `SPEC` rooted at the project, a valid
  v2 config per role, the CLI end-to-end with `git init` and `.local/` ignored.

## Generated shape (`agag init agecho --yes`)

```
agecho/
  .gitignore                 3 lines
  .local/instance.toml       1   (name = "agecho-agstudio1"; ignored)
  agents.toml               22   (ag.agent-config.v2, sonnet + stub, roles with allowed_tools)
  instance.example.toml      8
  params/intro.md           14   ({instance} + prefix + TODO)
  agent/guides/agechoplan_front/guide.md   5 (TODO stub)
  pyproject.toml            21   (pyagag from GitHub)
  service/listen.sh          6
  src/agecho/__init__.py     0
  src/agecho/intro.py        7
  src/agecho/listener.py    23   (the only code: AgentSpec + listener_main(SPEC, {}))
```

No `entrance_front/guide.md` is generated on purpose: the entrance uses the
built-in default with the prefixes filled in, which is what Step 4 tests.

## Decisions

- Default grant for every generated role is
  `Read,Write,Edit,Glob,Grep,Bash(agentchat:*)` — enough for the entrance
  and for a plan front; widening is an `agents.toml` edit.
- `roles` must include `front` (the entrance runs it); otherwise free.
- Zulip channel creation is **not** automated: at init time there is no bot
  credential yet, so it stays on the checklist with the account.
- Questions are five, not four — the destination is asked too, since
  `pj-agdev/` is not where a throwaway goes (plan, Step 4).
