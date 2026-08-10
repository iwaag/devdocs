# Legacy names and paths later phases will delete or replace

Compiled 2026-08-10 from a code survey of the three projects (paths are
relative to `pj-agdev/`). This is the Phase-1 evidence list the roadmap
asks for; the deleting happens in Phases 2–5.

## agforge (Phase 2)

- Backend value `"ollama"` meaning OpenCode+Ollama, and `"claude"`
  meaning Claude Code: `agforge/service/agent_run.py` (validation
  tuple, default, dispatch), `service/GUIDE.md`, `README_DEV.md`,
  `.local/devenv.md`.
- Env vars: `AGFORGE_AGENT_BACKEND`, `AGFORGE_OPENCODE_CMD`,
  `AGFORGE_OPENCODE_MODEL`, `AGFORGE_CLAUDE_CMD`, and the global bypass
  `AGFORGE_AGENT_CMD` (replace with a `fake`-harness profile).
- Hardcoded `CLAUDE_MODEL = "claude-sonnet-5"` (`agent_run.py`) — no
  env override exists today; becomes a canonical model in a profile.
- Pseudo-backend values leaking status into identity: `"override"` and
  `"error"` in the recorded `backend` field.
- Dead vars in `agforge/.local/.env` (from the removed
  `service/interpret.py` pipeline): `AGFORGE_INTERPRET_BACKEND`,
  `AGFORGE_OLLAMA_URL` (value also wrong vs the live opencode config),
  `AGFORGE_OLLAMA_MODEL`. Deletable immediately.
- Naming collision to keep, but disambiguate in docs: `--model` /
  `AGFORGE_SWARMUI_MODEL` name an SDXL image checkpoint, not an LLM.

## agautolab (Phase 3)

- Backend value `"ollama"` meaning a direct `/api/chat` call and
  `"claude"` meaning Claude Code CLI: `agautolab/agent/gateway.py`
  (`WINDOW_DEFAULT_MODELS`, `WINDOW_BACKENDS`, `run_ollama`,
  `run_claude`, `backend_model` composition), `agent/README.md`,
  `agent/GUIDE.md`, `tests/test_gateway_window.py`.
- Env vars: `AUTOLAB_WINDOW_BACKEND`, `AUTOLAB_WINDOW_MODEL`,
  `AUTOLAB_OLLAMA_URL`, `AUTOLAB_SUMMARY_MODEL`, `AUTOLAB_AGENT_MODEL`,
  `AUTOLAB_CLAUDE_BIN` (and the live `.local/.env` line
  `AUTOLAB_WINDOW_BACKEND=claude`).
- Direct-Ollama window code path `run_ollama()` — removed entirely per
  roadmap Phase 3.
- Model smuggled as CLI args in every `job.yaml`
  (`adapter_config.args: ["--model", "claude-sonnet-5", ...]`) —
  becomes a profile reference.
- Claude-shaped names in generic seams: artifact filename
  `claude_output.json` (adapter + gateway cost fallback + docs), the
  `claude_bin` pointer file, three duplicate binary resolvers
  (`gateway.py`, `agent/session.sh`, `src/agautolab/binpath.py`),
  literal fallback `"claude"`.
- Already-dead: `POST /director` route and `AUTOLAB_DIRECTOR_*` vars
  (removed in git); `POST /mission`; path drift `.local/direction/` in
  `gateway.py`/`agent/README.md` vs real `.local/projects/<name>/direction/`;
  stale job keys `no_progress_limit` / `consecutive_no_progress`.
- Record archives (`.local/agent/window/run-*.json`, director runs)
  carry old `backend`/`backend_model` spellings — readers must accept
  both; the files themselves stay as evidence.

## agdevworld (Phases 4–5)

- Backend value `"ollama"` meaning direct `/api/chat` and `"claude"`
  meaning the Anthropic Messages SDK: `agdevworld/assistant/server.mjs`
  (`BACKENDS` registry, `askOllama`, `askClaude`, `toOllamaMessages`,
  `toClaudeMessages`, `getAnthropic`, boot banner), `compose.yaml`,
  `README_DEV.md`, `assistant/GUIDE.md` (prompt-injected, so behavior).
- Env vars: `ASSISTANT_BACKEND`, `OLLAMA_URL`, `OLLAMA_MODEL`,
  `CLAUDE_MODEL`, `CLAUDE_EFFORT`, `CLAUDE_MAX_TOKENS`
  (`ANTHROPIC_API_KEY` becomes a secret reference).
- Hardcoded defaults duplicated between `server.mjs` and
  `compose.yaml`: `claude-opus-5`, `glm-4.7-flash:latest`,
  `http://host.docker.internal:11434`.
- Dependency `@anthropic-ai/sdk` in `assistant/package.json` plus its
  Dockerfile install layer — removed with the direct API path
  (Phase 5, unless a new evidenced role needs an API-native harness).
- Record field `backend_model` composite in `assistant.run.v1`
  records — superseded by separate harness/provider/model fields.
- Legacy inline-action parsing `extractAssistantActions` in
  `src/chatPanel.ts` (nothing asks for it any more).
- Stale doc: `README_DEV.md` still describes 10 s fetch timeouts.

## Cross-project

- The same two words `ollama`/`claude` mean three different pairs of
  things across the projects — the collision itself is the primary
  deletion target; canonical spellings are `opencode`, `claude_code`,
  `fake` (harnesses) and `ollama/<name>`, `anthropic/<name>` (models).
- `server.mjs`'s claim of being "the same shape as
  AGFORGE_AGENT_BACKEND" is true of the selector, false of the values'
  meaning — the comment goes with the rename.
