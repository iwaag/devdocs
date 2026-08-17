# p1 step 2 — agfront scaffold

Done. `pj-agdev/agfront` held only `LICENSE` and `.git`; it now holds the
configuration and the guide, and nothing else. The code is step 3.

## Files

| Path | What it is |
|---|---|
| `pyproject.toml` | package `agfront` (`src/agfront`), one dependency `pyagag` from the GitHub source, `agfront-zulip-listener` console script, pytest testpath |
| `agents.toml` | `ag.agent-config.v1`, `project = "agfront"`, the three standard profiles (`local`, `sonnet`, `stub`), `[roles.front] profile = "sonnet"` |
| `.local/agents.local.toml` | machine overlay: the Claude Code binary glob and the ollama base URL, copied from agforge's overlay |
| `agent/guides/front/guide.md` | Front's whole instruction — one file |
| `.gitignore` | `.local/`, `out/`, `.venv/`, `__pycache__/`, `.pytest_cache/` |

The dependency spelling is agforge's: `dependencies = ["pyagag"]` plus
`[tool.uv.sources] pyagag = { git = "https://github.com/iwaag/pyagag.git",
branch = "main" }`. No local path source, no gitea.

`profile = "sonnet"`, not `local` — the local rule says do not economize on
agent credit, and a front that routes badly is worse than no front.

## The guide

One file, guide and dispatch table together, in the terse style the other
agents' guides use. It says three things:

1. **Who Front is** — the Developer's relay. It routes; it does not do the
   work. Reply in the Developer's language.
2. **How to dispatch** — write `dispatch.md`, first line the topic name, rest
   the message; it is posted to `#general`. One table row today:
   "development work advanced" → `run-<something unique>`. And the honest
   part the plan asked for: *nothing comes back to this conversation* — the
   agent that takes the work replies inside the `run-` topic, so Front tells
   the Developer which topic it used and stops there.
3. **When no row fits** — refuse plainly and say what would be needed.
   Guessing at a nearby row is worse than refusing.

The channel is not in the file format, only in the table text: `dispatch.md`
carries a topic, and the handler owns the channel constant. Front can
therefore reach exactly the one channel it is subscribed to, which keeps
"subscription is the routing decision" true rather than decorative. A second
outbound channel is a later phase's change to both the table and the handler.

## Choice recorded for step 3

Of the two dispatch mechanisms the plan offered, this takes **(a) the command
file**: the run writes `dispatch.md`, the handler posts it. It matches
agautolab's `new_mission.md` pattern, needs no new CLI, and is testable
against a fake client. (b), a posting tool, is the purer Tool Giving and is
the natural upgrade once Front has more than one row to reach.
