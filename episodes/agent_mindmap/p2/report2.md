# p2 step 2 — window backend and run records

AI-generated (Omni Agent, 2026-08-09). Status: **done**. The code landed in
the same commit as step 1 (it is the same route); this report covers the
backend layer and the records.

## Backend selection

agforge's pattern copied in shape, not shared as a library — the plan asked
for the copy, and a shared module between two workspaces that deploy
separately would buy nothing and couple them.
`gateway.local_env(name)` resolves process env first, then
`agautolab/.local/.env`, exactly as `agforge/service/agent_run.py:local_env`
does; `window_backend()` mirrors `agent_backend()` including the "unknown
value is an error, not a silent default" behavior.

| variable | default | meaning |
|---|---|---|
| `AUTOLAB_WINDOW_BACKEND` | `ollama` | `ollama` \| `claude` |
| `AUTOLAB_WINDOW_MODEL` | `gemma3:latest` / `claude-sonnet-5` | model for the chosen backend |
| `AUTOLAB_OLLAMA_URL` | `http://127.0.0.1:11434` | ollama endpoint |

`ollama` is the default, per the plan: a small local model, so idle chatter
with the window is free. `AUTOLAB_OLLAMA_URL` exists because a node without
a local ollama (agautolab1) has to point somewhere; the claude backend
reuses the existing `claude_bin()` resolution the summarizer already uses.

Documented in `agautolab/agent/README.md` ("The conversational window →
Backend").

## Records

Per `devpolicy/agent_records.md`, one file per answer at
`.local/agent/window/run-NNNN.json` — a sibling under `.local/`, which the
plan allowed. Fields:

- **id** — `window/run-NNNN`
- **backend** / **backend_model** — e.g. `claude` / `claude/claude-sonnet-5`
- **outcome** — `done` | `failed`
- **cost / time** — `duration_ms` always (harness-measured); `cost_usd`,
  `num_turns` from claude; `prompt_tokens` / `output_tokens` from ollama,
  with `cost_usd: null` because ollama reports no price and inventing one is
  forbidden
- **on failure** — `failure`, the backend's words verbatim

The directory is under `.local/`, already git-ignored (`git check-ignore`
confirms), so questions and answers stay off Git.

One honest deviation, worth naming: the policy asks for a failure report
"in the agent's own words". When a window run fails, the agent has usually
not spoken at all — the launch failed, or the endpoint refused. The record
carries the *failing party's* verbatim text instead (`could not launch
claude (…): No such file or directory`). The harness still fixes only the
path, never the wording.

## Acceptance — the switch demonstrably changes the record

Same question, same node, same minute, backend flipped by process env:

| backend | record `backend_model` | `cost_usd` | `duration_ms` | answer |
|---|---|---|---|---|
| `ollama` (default) | `ollama/gemma3:latest` | `null` (2529 prompt / 63 output tokens) | 1494 | partly wrong — called converged jobs "running" |
| `claude` | `claude/claude-sonnet-5` | `0.0903679` | 10116 | correct: six converged jobs, most expensive snake-web-b at $1.348516 |

Records: `.local/agent/window/run-0007.json` and `run-0010.json`.

## Notes

- **ENT — a stale pointer file cost a run.** The first `claude`-backend
  attempt failed with `could not launch claude
  (…/anthropic.claude-code-2.1.224-…/claude)`: `.local/agent/claude_bin`
  pins an absolute, *version-numbered* path into the VS Code extension
  directory, and the extension had updated to 2.1.226. This breaks the
  existing summarizer the same way, silently, on every extension update.
  Repointed the file (local-only, one line). The durable fix — resolve the
  newest match rather than a pinned path, or install a stable symlink —
  belongs to whoever next touches `claude_bin()`; it is not this phase's
  step. The failure record made this diagnosable in one read, which is the
  first small return the record policy has paid.
- The window's own cost figure was written back into `agent/GUIDE.md` after
  measuring it, replacing the guessed "a fraction of a cent" with the
  measured 0.09 USD — the card should not carry a number the node can
  disprove.
