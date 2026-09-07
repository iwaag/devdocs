# gauge_panel — external-backend cost on one screen

One page, opened in its **own browser window** beside the operation room,
that shows what the agents' external backends (claude_code, codex, agy /
Antigravity, gemini_cli, plus the free local ones) have cost — per harness,
per day, per instance/role, and per routine session.

This is a destructive phase in a private experimental environment. No
backward compatibility: change record shapes, relay APIs and frontend
modules as useful. The steps describe outcomes, not recipes. The only
things to keep are the two existing invariants of the relay — **never poll
Zulip for this** (the cost page needs no Zulip at all) and **an absence is
rendered as unknown, never as zero**. UI text is English.

Decided in the advice that preceded this plan (2026-09-07): no external
service. Provider dashboards are one-per-vendor, Antigravity and Codex are
subscriptions with no USD, ccusage-style tools read only Claude's JSONL, and
proxy meters (LiteLLM, Helicone) cannot sit in front of the CLI harnesses.
The primary data already exists on this host; the relay already reads the
directories it lives in.

## What is already there (measured 2026-09-07)

- **1150 `ag.agent-run.v1` records** under `<instance>/.local/agent/<role>/run-NNNN.json`
  (agfront 551, agautolab 485, agforge 114; pj-clusterintent has a few too).
  Writer: `pyagag/src/agag/harness.py:870` `write_run_record`, called from
  `pyagag/src/agag/agent.py:288`; agforge wraps the same writer in
  `pj-agdev/agforge/src/agforge/agent_run.py:131`.
- Harness mix: claude_code 1072 · agcode/ollama 30 · gemini_cli 27 ·
  opencode/ollama 14 · agy/antigravity 4 · codex/openai 3.
- **`cost_usd` exists only for claude_code** (sum so far ≈ 291.7 USD) and as
  `0.0` for opencode. codex, agy, agcode report tokens only; gemini_cli's
  27 runs are all failed (`exit 41`, no usage). Token key names differ per
  harness — codex `cached_input_tokens`, agy `cache_read_tokens`, claude
  `cache_read_input_tokens`, opencode `input`/`output`/`cache_read`.
- **No timestamp in the record** (only `duration_ms`; `started` exists on 19
  opencode records). File mtime is the end time today.
- **No channel/topic in the record.** Attribution to a conversation is
  today only the `/inflight` mtime trick: the generation dir
  `.local/topics/<ch>/<topic>/<N>/<role>/` is created ~5 s before the run,
  the record is written after it (p1: 94% unambiguous at 5 s slack).
- Relay `AGENTROOM_AGENT_ROOTS` is configured and live: `/inflight/ghtrends`
  answers `configured: true` with `front-agstudio1` and `autolab-agstudio1`
  known. Routine work is served by the **local** autolab instance, so the
  records that matter for routines are on this Mac. The VM `agautolab1`'s
  records are not here; they stay *unknown*, not zero.
- Models in use: `pj-agdev/agfront/agents.toml` `[models.*]` table
  (sonnet-5, gpt-5.6-terra, gemini-3.8-flash-medium, claude-sonnet-4-6 via
  antigravity, gemini-2.5-flash, ollama qwen). There is no price anywhere yet.

## Step 1: make the record self-describing (pyagag)

Add to every new record, in `write_run_record` / its callers:

- `started_at`, `ended_at` (epoch seconds, UTC) — set in `run_harness` or
  around it; stop relying on mtime for new records.
- `channel`, `topic`, `generation` when the run serves a topic. The
  `serve_topic` handler (`pyagag/src/agag/topics.py:339`) receives a context
  that knows all three; pass them through `extra_meta` (already stamped into
  the record at `agent.py:284`). Roles without a topic (autolab `coding`,
  forge `generator`) simply omit them.

No migration of old records: the relay treats a missing `ended_at` as file
mtime and a missing topic as *unattributed*. Re-post nothing to Zulip; this
is file-only. Restart the listeners that embed pyagag afterwards
(`nctl status` / `nctl drift` tell you which are polling).

## Step 2: relay `/cost` (agentroom)

New read route beside `/inflight`, same roots, no Zulip:

- Walk `AGENTROOM_AGENT_ROOTS`, parse every `run-*.json`. 1150 files is
  milliseconds; an mtime-keyed in-memory cache is enough. Poll-safe like
  `/inflight`.
- Normalize one row per run: instance, role, harness, provider, model,
  `started_at`/`ended_at` (mtime fallback), duration, outcome, a **unified
  token quartet** (input, cached_input, output, reasoning) mapped from each
  harness's key names, `cost_usd`, and `cost_kind`:
  - `reported` — the harness gave USD (claude_code).
  - `estimated` — tokens × a price table (codex on API key, gemini_cli).
  - `subscription` — metered by the vendor's plan, show tokens/runs, **no
    USD** (agy / Antigravity; codex when it runs on a ChatGPT login — check
    which one `codex` here actually uses before deciding).
  - `local` — ollama, cost 0 and say so (opencode's `0.0` means free, not
    unknown).
  - `unknown` — no usage at all (the failed gemini runs).
- Price table: a small JSON/TOML file, path in the plist env
  (`AGENTROOM_PRICES`), keyed by the `provider/model` string, USD per
  million tokens for the quartet. Prices are not secrets; a tracked default
  under `agentroom/` is fine. Never invent a number: a model missing from
  the table is `unknown`, not 0 (`devpolicy/agent_records.md`).
- Aggregations the page needs, computed server-side so the page stays dumb:
  totals by harness for today / 7 d / 30 d, per-day series (≥ 14 days), per
  instance×role, and **per routine session**. For sessions reuse
  `ops.routine(name)` — it already knows each session's topics and the
  `/inflight` generation timestamps; join on `channel`/`topic` when the
  record has them, else by the mtime window. Mark which join was used.
- Return `generated_at`, the roots that were readable, and a `missing[]`
  list (instances in the roster that have no root here) so the page can say
  what it cannot see.

Tests: a `tests/test_cost.py` with a handful of fixture records copied from
the real shapes above (one per harness) is worth more than any mock.

## Step 3: the gauge page (agdevworld)

- A separate entry, e.g. `/?parts=gauge` in `src/main.ts`'s routing (the
  dashboard is untouched; the operation room header gets one link that opens
  it with `target="_blank"`). Reuse `operationParts.css`; no Phaser.
- Content, top to bottom: today's spend (reported + estimated, subscriptions
  shown as tokens beside it, not inside it); a 14-day bar/sparkline by
  harness; a table instance × role with runs / tokens / USD / failures; a
  routine section listing the same ≤ 3 sessions the operation room shows,
  each with its cost breakdown by agent — this is the line that lets the
  operator read "this routine run cost X" while driving it.
- Poll `/cost` every 15–30 s. It is host files only, so this is free.
- Degrade honestly: relay down → the page says so by name (same pattern as
  the agent room); `missing[]` → "not observed on this host", never 0.
- Screenshot helper: `node .local/opsshot.mjs <url> .local/shots` from
  `agdevworld/`, `OPSSHOT_STEPS` scripts clicks.

## Step 4: deploy and prove

- Relay: `launchctl kickstart -k gui/$(id -u)/com.agdev.agentroom`; log at
  `agdevworld/agentroom/.local/out/agentroom.log`. Add `AGENTROOM_PRICES` to
  `pj-agdev/devenv/launchd/com.agdev.agentroom.plist.in` and regenerate the
  installed plist (the installed one drifts from the template — diff them).
- Frontend: `docker compose up --build -d web` (`:8090`), or `:5173` for HMR.
- Proof in `report.md`: the page beside the operation room during one real
  routine fire (ghtrends fires at 07:00Z daily); the session's cost line
  matches the sum of that session's run records; one codex or agy run shows
  as subscription/estimated, not as 0.

## Hints

- zsh: quote globs in shell one-liners (`--include='*.py'`), or `noglob`.
- `/inflight`'s docstring in `agentroom/src/agentroom/inflight.py` is the
  reference for the two role vocabularies (`entrance_front` record dir vs
  `front` workspace dir) — compare against records of *any* role.
- agforge writes records through its own `agent_run.py`; if Step 1 adds
  fields in pyagag, check that path picks them up too.
- `ops.routine()` sessions are the p7 shape (one topic per run); sessions
  older than the newest three are in Zulip only — the cost page may show
  more history than the operation room does, from files alone.

## Out of scope

The VM `agautolab1`'s records (a later sync or node-side read), budget
alerts, and reconciling estimates against vendor dashboards.
