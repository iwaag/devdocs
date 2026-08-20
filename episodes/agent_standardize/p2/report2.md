# p2 step 2 — the intro harvest before each run

AI-generated (Omni Agent, 2026-08-20).

## What was built

`agfront/src/agfront/agents_md.py` — a deterministic module, no model call
anywhere on its path:

- `harvest_intros(client)` reads the `#agents` channel's topic list, keeps
  the `intro-*` ones, skips resolved (`✔`) topics, and takes each topic's
  **latest** post. Intros are append-only, so the newest post is the current
  revision. Entries are sorted by agent name, so two harvests of one board
  produce the same file.
- `render_agents_md(entries, generated_at)` assembles the markdown by string
  operations: a heading, a one-paragraph preamble saying these are the
  agents' own words and that `agentchat` is how to reach them, a
  `Generated:` line, then `## <agent>` and the body **verbatim**.
- `write_agents_md(client, front_dir)` drops it at
  `<generation>/tools/agents.md`, creating `tools/` as needed.

The listener calls it in `serve()` as a named step (`harvest`) immediately
before the run, so a transport failure is reported to the topic as
`failed during harvest: …` rather than as a mystery. The file lands in the
existing generation workspace
(`.local/topics/<channel>/<topic>/<N>/front/tools/agents.md`), which keeps
committed files clean and gives every run the board state of that moment.

`RESOLVED_TOPIC_PREFIX` is imported from `agag.zulip` rather than redeclared.

## Attributability

The module names no agent and no channel other than `#agents` itself. What
Front learns about agforge — that its entrance is the `agforge-agstudio1`
channel and that a request is a `create-…` topic — is text agforge wrote
about itself, copied through. That is the p2 success criterion, and this is
the file it depends on.

The premise that made this possible without touching subscriptions was
proven live in step 1: the front bot, subscribed only to `#front` and
`#general`, reads `#agents` fine.

## Empty board

An empty harvest is a fact, not a crash: the file is written with
`No agent has introduced itself on the #agents board, so there is nobody to
ask.` A run then still happens and Front can refuse gracefully. Pinned by
`test_an_empty_board_does_not_stop_the_run`.

## Verification

`uv run pytest` in agfront: **31 passed**. New this step:

- `tests/test_agents_md.py` (11 tests) — latest post wins, resolved and
  non-intro topics skipped, blank/empty topics contribute nothing, ordering
  is stable, only `#agents` is read, bodies verbatim, the `Generated:` line,
  the empty-board line, and placement under `tools/`. The fake client raises
  if a subscription call is ever made.
- Three tests in `tests/test_zulip_listener.py` — the file lands in `tools/`
  before the run, a second run sees a changed board, and an empty board does
  not stop the run.

Two pre-existing tests needed their fake Zulip clients taught about the
board. One assertion in `tests/test_role_run.py` was stale before this step:
it asserted the shipped guide mentions `create.md`, which the already
rewritten guide no longer does, so the stub-run test was red on arrival. It
now asserts the prompt mentions `agentchat`. The rest of that test still
exercises the `create.md` route and is step 3's to rework.

## Marked for later

Pull vs. push, caching, per-agent files — not designed now, as the plan
says. This is the first cut.

## Delivery

agfront `ffc044e`, pushed to GitHub.
