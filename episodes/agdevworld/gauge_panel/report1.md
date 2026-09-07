# gauge_panel step 1 — the record describes itself

Every new `ag.agent-run.v1` record now carries `started_at` / `ended_at`
(epoch seconds) and, when the run served a conversation, `channel` / `topic`
/ `generation`.

## What changed (pyagag `5ca63ea`)

- `harness.run_harness` stamps `started_at = time.time()` into `meta` at
  the same place it takes the monotonic start, so every exit path —
  timeout, launch failure, normal return — carries it.
- `harness.write_run_record` copies the five new keys and writes
  `ended_at` as the time of writing (the record is written as soon as the
  harness returns; that is the end of the run to within the write).
- `topics.workspace_identity(cwd)` is the inverse of `generation_dir`: it
  finds the `.local/topics/<channel>/<topic>/<N>/` pair in the run's cwd.
  `agent.run_role` merges it into `meta` before the caller's `extra_meta`,
  so no caller changed: agfront, entrance, arxivsage and forge's topic roles
  all run *in* the generation directory. Runs that do not (autolab
  `coding`/`director` in a project clone, forge's `generator`) stay
  unattributed, which is the truth.
- The names are directory names (after `_safe_topic_component`), which is
  what the relay's `/inflight` compares too.

Tests: 456 pass; two exact-equality assertions on `result.meta` in
`test_harness.py` now pop `started_at` first.

## Roll-out

pyagag is a git dependency (`branch = "main"`) of every agent, pinned by
`uv.lock`. Bumped and pushed in agfront `9165a1c`, agautolab `110d6ee`,
agforge `95613ea`, arxivsage `ff269aa`; `uv sync` each; then
`launchctl kickstart -k` on `agfront-zulip`, `agautolab-zulip`,
`agforge-zulip`, `agforge`, `arxivsage-zulip` with every instance idle
(`/inflight/ghtrends` showed no run in flight). All five came back;
recovery sweeps spent 3–6 Zulip calls each and served nothing. agecho was
not bumped (not running).

Not proven yet: a real record with the new fields. The next fire of any
routine writes one; step 4 checks it.
