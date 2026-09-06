# operation_room p1 implementation plan (investigation phase)

p1 is an investigation phase. The goal is to establish, per signal source, how
far execution state can be detected, and to gather the material for deciding
operation_room's design. Any code written here is an experiment or probe
(throwaway), not a production feature.

## Deliverables

report.md must contain:

1. **A capability matrix per signal source**: source / what it tells you /
   latency & freshness / attributable to a task? / work needed to use it
2. **Proposed state definitions**: the braindump's
   [planned only / running / awaiting reply post / done] redefined against
   what turned out to be observable (collapse states that cannot be detected)
3. **A cannot-do list**: everything established as undetectable or
   unattributable, stated explicitly

## Known facts at planning time (no investigation needed; copy into the report)

Confirmed while writing this plan, to avoid duplicate investigation.

### Run records are written only at run end
- `pyagag/src/agag/harness.py:870` `write_run_record()` does a single
  `write_text`. The call site is around `agag/agent.py:271` — **after**
  `run_harness()` returns. A run in progress therefore leaves no
  `run-NNNN.json`. Not even the number is reserved (`agag/topics.py:102`
  `next_record_path()` scans for a free number).
- The record has no start/end timestamps, only `duration_ms`. The file mtime
  is the only end time.
- Fields: `schema: ag.agent-run.v1`, `role, profile, harness, provider, model,
  duration_ms, cost_usd, usage, num_turns, transcript, outcome, failure`, etc.
  → **Excellent source for completion state and backend attribution; useless
  for detecting "running".**
- Location: `.local/agent/<role>/run-NNNN.json` in each repository.

### No lockfile / pidfile / status file marks a run in progress
- Nothing under `agag/` writes a lockfile, pidfile, or in-progress marker.
- `agag-status.json` (`agag/status.py:45`, `agag.status.v1`) is the
  *listener's* poll-health file and is **only updated on a successful poll**.
  While a harness run blocks the listener the file goes stale, so staleness
  cannot distinguish "busy" from "dead". A trap for any observer.

### forge's generation-job state is readable as file transitions
- `agforge/src/agforge/assetrun_topic.py:87,152` —
  `.local/agentws/<work id>/generator/pending.json` (submitted, unwatched)
  is replaced by `watching.json` in `start_watching()` (same file, :342).
  Fields: `{prompt_id, note}` plus `trigger` added at watch time.
- This is **currently the only path that attributes a backend job to a task
  (work id)**.

### ComfyUI
- URL comes from `AGFORGE_COMFYUI_URL` (agforge's `.local/.env`).
- Existing read code: `agforge/src/agforge/comfy_video.py:105`
  (`GET /queue` → `queue_running` / `queue_pending`), :152 and
  `comfy_async.py:74` (`GET /history/<prompt_id>`). Equivalent helpers in
  `comfynotify/src/comfynotify/comfy.py:16-34`.
- SwarmUI has no live referencing code (only env names in a past episode).
  Think in terms of ComfyUI directly.

### ollama
- base_url is configured per workspace in `.local/agents.local.toml`
  `[local.provider.ollama]` (parser: `agag/agent_config.py:289`); default in
  `agag/agcode.py:34` = `http://localhost:11434`.
- **Nothing anywhere calls `/api/ps`** (the nctl health probes use
  `/v1/models` and `/api/tags` only). Session observation is entirely new
  ground.
- ollama sessions carry no requester information, so task attribution is
  expected to be impossible in principle.

### The routine list already has a GUI seam
- The source of truth is `pj-agdev/.local/rtschedule/schedule.json` (a clone
  of a Gitea repository). Schema: `pj-agdev/devenv/routine/dispatch.py:50-95`
  (`requests[{id, said_at, until, text}]` /
  `events[{id, at, from, kind, routine, fired_at, logical_at}]`).
- The roster of routine names is the `routine-*` topics in `#front`; a fire is
  a post into the `front-routine-<name>` topic (`devenv/routine/trigger.sh`).
- **`com.agdev.routine-gui.plist.in` already serves the rtschedule directory
  with `python3 -m http.server 8093`.** Whether operation_room reuses or
  replaces this is a design decision — include it in the report.
- The launchd residents (`com.agdev.agforge-zulip` and the other listeners,
  comfy-notifier) make `launchctl list` a cheap "what is up" source.

### The selfnote parsers already exist
- `pyagag/src/agag/selfnote.py` — `SELFNOTE_MARKER` (:56), `parse_served()`
  (:176) → `(Conversation, message id)`, `parse_rootchat` (:162),
  `without_selfnotes()` (:210), `last_real_speaker` (:224).
- Invariant (:42): a selfnote must never count as "who spoke last". Any
  observer must run `without_selfnotes` before deriving activity.

## Investigation items

What the known facts above do not settle. In priority order.

### A. Reconstructing task state from Zulip (most important)
- Verify on real data that "a callback that named an agent but has not been
  served" can be detected by matching against
  `[selfnote][served] <channel>/<topic> <message id>` — including whether the
  matching works across the ✔ rename (this is exactly the re-detection of
  p9's 26-minute stall).
- Confirm that [awaiting reply post] and [stalled (named X minutes ago,
  unserved)] can be derived from this. **Stalled detection is the core value
  of this screen**, so verify with the intent of adding it to the four-state
  proposal.
- The agent_room relay (`agdevworld/agentroom/room.py`) can serve as the base
  for probes, but in p1 do not add production code to the relay itself —
  throwaway scripts are fine.

### B. Experiments in approximating "running"
With no run records and no lockfiles, there are three candidates. Check
detection rate and false positives with throwaway probes for each:
1. Watching the transcript file grow (written by `run_harness` while
   streaming)
2. `pgrep` for the harness CLIs (claude / codex / agy / gemini / agcode)
3. `agag-status.json` staleness (with its busy/dead ambiguity, only in
   combination with other signals)

If the conclusion is "running cannot be caught event-driven; only a coarse
polling approximation exists", that is a fine result — report it as the basis
for the design principle (state is reconstructed from traces).

### C. Attributing backend activity to tasks
- Verify that matching ComfyUI `/queue` prompt_ids against forge's
  `pending.json` / `watching.json` can actually answer "which work's
  generation is running now".
- Hit ollama's `/api/ps` once and record what comes back (just model names
  and counts?). If attribution is impossible, put that on the cannot-do list.

### D. Reconstructing the state definitions
Based on A–C, redefine the states in two layers:
- **Conversation/task layer** (derived from Zulip): planned only / awaiting
  reply / stalled / done (✔)
- **Process/backend layer** (real observation): harness-run approximation,
  ComfyUI queue depth, ollama sessions, launchd liveness
Do not force these into one state machine. Write the design direction on the
premise that the only join between the layers today is forge's prompt_id path.

## Constraints (minimal)

1. **Probes must not post to Zulip.** A bot post triggers paid agent runs and
   misdirects conversations (a known incident pattern). Work read-only. If a
   posting experiment is truly needed, create a dedicated test channel that no
   agent watches.
2. Keep experimental code somewhere clearly disposable (under `.local/`, an
   explicit `experiments/`, etc.). Production changes to the relay,
   PanelGridScene, etc. wait for p2+.
3. Do not copy credentials or `.local` values into reports or commits
   (naming env vars and file paths is fine).

Everything else (probe language, verification order, depth) is the
implementer's discretion. Anything established early as "cannot be obtained"
should go on the cannot-do list without deep pursuit — move on.
