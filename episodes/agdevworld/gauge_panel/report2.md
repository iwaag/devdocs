# gauge_panel step 2 — relay `/cost`

`GET /cost` on the agentroom relay (agdevworld `agentroom/src/agentroom/cost.py`,
route in `server.py`, wiring in `main.py`, table `agentroom/prices.json`,
README section "`/cost` — what the backends cost"). Host files only; the
ops engine is asked for the roster and each routine's sessions, which it
answers from links it already holds — no Zulip call is spent by a poll.

## What it does

- Walks the same `AGENTROOM_AGENT_ROOTS` as `/inflight` (five roots on
  agstudio: front 551 records, autolab 484, forge 114, arxivsage 8, agecho 2;
  one autolab file is unreadable JSON and is skipped). Incremental: a file
  is re-read only when its mtime moves; a price-table edit reprices all.
- One row per record with a **unified token quartet** (`input` uncached,
  `cached_input`, `cache_write`, `output`, plus informational `reasoning`).
  codex's `input_tokens` includes its cached part and is split; opencode's
  `input`/`cache_read` and agy's `cache_read_tokens` are mapped.
- **Cost kind per harness** from `prices.json`: claude_code `reported`,
  gemini_cli `metered` → `estimated` when the table prices the model,
  codex and agy `subscription` (codex here is on a ChatGPT login —
  `~/.codex/auth.json` `auth_mode: chatgpt` — so USD is not a fact the run
  has), agcode/opencode `local` (0). No usage at all → `unknown`, never 0.
- Old records: end = file mtime, start = end − `duration_ms`, conversation
  = the generation directory whose mtime span overlaps the run's (30 s
  slack, each directory spent once, nearest start wins). The first rule I
  wrote — "the run's end falls inside the directory's span" — attributed
  nothing for agfront, because agfront writes nothing into the workspace at
  the end of a run, so the span ends at the start. Overlap fixed it:
  front 550/551, autolab 406/484, forge 88/114 attributed by window; the
  rest are roles that run outside a topic workspace, as expected.
- Aggregates: `totals.{today,days7,days30,all}` with `by_harness`, `days[]`
  (14 local days), `table[]` (instance × role × harness × model),
  `routines[].sessions[]` with per-agent breakdown and an `attribution`
  count, `unattributed`, `recent[]`, `roots[]`, `missing[]`, `prices`.

## Measured

- `board()` over 1149 real records: 139 ms cold, 90 ms warm (all of it
  the attribution pass, which is recomputed every scan).
- Live after `launchctl kickstart -k`: today = 7 runs, 1.185 USD reported;
  routine `ghtrends` session `2026-09-07T07:00Z` = those same 7 runs
  (front 4 in the run topic, autolab superdirector 1 in `workplan-trend4`,
  supercoder 2 in `workrun-task1-g-9`), all `window`-attributed since they
  predate step 1. All-time: 291.67 USD reported, all of it claude_code;
  codex 3 and agy 4 runs are `subscription`; the 27 gemini_cli runs are
  `unknown` (every one failed before counting).
- Tests: `tests/test_cost.py`, 5 tests on records copied from the real
  shapes; suite 119 pass.

## Left as is

`prices.json` prices only `google/gemini-2.5-flash`, from memory, with a
note to verify; nothing on this host has ever produced a priced gemini run,
so no estimate is currently shown anywhere. `AGENTROOM_PRICES` is not set
in the plist: the tracked default is used.
