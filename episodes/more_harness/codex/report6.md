# Report 6 — proof: front, the gateway window, and one workrun on `codex` (2026-09-05)

Step 6 of `plan.md`. Everything the plan asked to see was seen, in about
three minutes of wall clock, all through the listeners started in step 5
(launchd, not a shell). Timestamps are UTC; the local clock was past
midnight, 2026-09-05. All three roles ran on `openai/gpt-5.6-terra` at
effort `medium`, with `-s danger-full-access`.

## Front on `codex` — `#front > front-codex-trial`

`[roles.front] profile = "codex"` in agfront's overlay, one post as the
Developer (message 4856, 16:52:03). Front's reply landed at 16:53:43; the
wait was the restart backlog (one re-served `front-routine-publish` run on
sonnet first). It described itself and named autolab, correctly and
without contacting anyone — but also named **`cagent`**, which is not in
the `tools/agents.md` it was given (agecho ×2, agforge, agping, arxivsage,
autolab). Not a harness fault; noted as the kind of thing a model does
with a name it knows from elsewhere.

`agfront/.local/agent/front/run-0545.json`:

| field | value |
|---|---|
| harness / provider / model | `codex` / `openai` / `openai/gpt-5.6-terra` |
| outcome | `done` |
| duration_ms | 16926 |
| num_turns | 2 (tool items) |
| usage | input 45720, cached 38144, output 346, reasoning 62 |
| cost_usd | absent |

So `~/.codex/auth.json` is reachable from a launchd-started process, as the
plan's `env -i` probe predicted; no key plumbing was needed.

## Gateway window on `codex` — `POST /window`

`[roles.front] profile = "codex"` in agautolab's overlay, one `curl` to
`:8791/window`. `window/run-0019`: `outcome: done`, 26.0 s, `num_turns 4`,
`usage` present (input 70734 / cached 59136 / output 599 / reasoning 110),
`is_error: false`. The gateway needed no change. The answer itself was
thin — "This is the primary Codex agent node (`/root`) …" — a model
describing itself rather than the node it was asked about, after four tool
calls. The agy run on the same question read the README and described
the node. One sample; noted, not pursued.

## One `workrun-` task on `codex` — `pj-runsmoke2`

`[roles.supercoder] profile = "codex"` in agautolab's overlay. The mission
was opened in `pj-runsmoke2 > workplan-codex-trial` (planning on `sonnet`,
as configured). **The first planning run wrote `plan.md` but no task
file** ("a single task, no split needed"), so no `workrun-` topic opened;
a follow-up post as the Developer asking for `task1.md` got a second
planning run that wrote it and opened `work-r2-6/workrun-task1-r2-6`.
Planning ran on sonnet both times, so this is a superdirector variance,
not a codex one — the identical agy mission got its task file first time.

One post there started the task at 16:53:55; it was reported done at
16:54:56:

- `main/CODEX_TRIAL.md` exists **in the project workspace** with the two
  requested lines, committed as `ca0c1ae codex trial`, pushed
  (`## main...origin/main`, no divergence). So `danger-full-access`
  commits, pushes, and reaches Zulip from a listener-started run, and
  `-C <cwd>` put the file where it belongs.
- **The progress display worked through the translation**: the topic
  shows ten `🔧` lines — `shell: /bin/zsh -lc "git status …"`,
  `apply_patch: …/main/CODEX_TRIAL.md`, `shell: … git commit -m 'codex
  trial' …` — and four `💬` lines, the claude-shaped form autolab's
  `RunProgress` reads. `file_change` came through as `apply_patch` with
  its path.
- `agautolab/.local/agent/supercoder/run-0217.json`: `harness codex`,
  `outcome done`, 59.9 s, `num_turns 9`, usage input 173206 / cached
  151040 / output 1508 / reasoning 305, no cost.
- The 20-minute timeout was not exercised (60 s run). Codex has no
  timeout flag; a killed run would fall through run_harness's timeout
  branch, which is tested but not seen live.
- `~/.codex/sessions/` held the same 201 files before and after all
  three runs: `--ephemeral` holds under launchd too.

Two observations, not harness problems:

- **The supercoder posted its result twice**: once itself, with
  `agentchat send` into its own `workrun-` topic (message 4873, "Completed
  and pushed … Commit: ca0c1ae…"), and once through the listener's normal
  reply (4875, naming the Developer). The guide says the run's answer
  *is* the reply; codex read "report the hash back to the requester" as an
  instruction to post. Harmless here, and it did **not** close its own
  task ("the task remains open until …") — the opposite of the agy run.
- `num_turns` is the tool-item count (9 here, 4 for the window, 2 for
  front), the unit report1 chose; a reader comparing with claude_code's
  API turns or agy's user-message count should know the units differ.

## The failure path

Seen in report1 through `run_harness` with the real CLI from this shell:
`openai/no-such-model-xyz` → exit 1, `outcome: failed`, `subtype
turn_failed`, `failure: "codex exited 1: {"type":"error","status":400,…
model is not supported when using Codex with a ChatGPT account."}` — the
400 quoted. Not repeated through a listener: that would need a bad model
committed to `agents.toml`.

## Not done

- The Ollama route (`--oss --local-provider ollama`) was left out in step
  1 (`openai` only), so no `summarizer` run on `ollama/…`.
- No read-only role ran live; `summarizer`'s `-s read-only` is covered by
  report1's real-CLI probe (writes denied, model explains) and role_run's
  test.

## State left behind

- Both overlays are back to `sonnet`; the codex lines are commented next
  to the agy ones, ready to uncomment.
- `pj-runsmoke2` has mission R2-6 (In Progress, task open, awaiting the
  requester's acceptance) and `work-r2-6`; harmless smoke residue beside
  R2-4 and `work-r2-4`.
- One extra `front` sonnet run was paid for the restart backlog, as step 5
  warned; and one extra sonnet planning run for the missing task file.
