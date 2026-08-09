# p2 step 5 — backend selectability sweep (Agent ≠ Model)

AI-generated (Omni Agent, 2026-08-09). Status: **done**, with one honest gap
named below (the assistant's `claude` backend is wired, documented and
recorded, but was never run — this machine has no Anthropic API key).

## Where each agent stands

| agent | switch | default | verified |
|---|---|---|---|
| autolab window | `AUTOLAB_WINDOW_BACKEND` | `ollama` | **both backends, real runs** (step 2) |
| agforge | `AGFORGE_AGENT_BACKEND` | `ollama` | **both backends, real runs** (below) |
| assistant | `ASSISTANT_BACKEND` | `ollama` | ollama run real; claude path reaches the API-key check and records `failed` |
| cagent | `CAGENT_OPENCODE_MODEL` | `openai/gpt-5.6-luna` | rendered + served, restart verified |
| autolab delegate | `job.yaml adapter:` | per job | already conformed; re-confirmed against evidence |

## agforge — verify one run per backend

Already conformant, so this step was measurement, not code. The **same
desire** ("a small watercolour of a lighthouse at dusk") on each backend, both
`done` with a URL that passed the runner's GET check:

| backend | `cost_usd` | `duration_ms` | `num_turns` |
|---|---|---|---|
| `ollama` | 0.0 | 31151 | 5 |
| `claude` | 0.1341148 | 18249 | 4 |

The switch is visible in the record, which is the acceptance criterion.
Documented in `agforge/README_DEV.md` (Agent backends), and the measured
0.134 USD replaced the card's estimated range.

## assistant — the "engine-agnostic seam" implemented

`server.mjs` carried a comment promising a seam; there wasn't one — `handleChat`
called ollama inline. Now `handleChat` builds the system prompt and hands
`(system, messages)` to a backend, and only the `BACKENDS` entries know what
answers. `ASSISTANT_BACKEND` resolution copies agforge's shape: one default,
unknown value is an error rather than a silent fallback.

- **`ollama` (default)** — unchanged behavior, plain `fetch`, no dependency.
- **`claude`** — the official `@anthropic-ai/sdk`, `claude-opus-5` at `low`
  effort (this assistant answers from a snapshot and a card, not from hard
  reasoning), imported lazily so the default path never pays for it and a
  missing package breaks only the backend that needs it. `refusal` is handled
  before reading content, and only `text` blocks become the reply.
- The image now runs `npm install` (new `assistant/package.json`); compose
  passes `ASSISTANT_BACKEND`, `ANTHROPIC_API_KEY` and `CLAUDE_MODEL` through,
  with empty meaning "use the default" — the `||`-not-`??` trap this file
  already documents for `AUTOLAB_NODES`.

**Records**: one JSON line per reply on stdout (`kind: "assistant.run.v1"`) —
id, backend, backend_model, outcome, duration, tokens, and on failure the
backend's verbatim words. The container has no writable volume, so its log
*is* the record store; `ASSISTANT_RECORDS_DIR` optionally writes files too.

**The gap, stated plainly.** There is no `ANTHROPIC_API_KEY` and no `ant`
credential on this machine, so the claude backend has never produced an
answer here. What *was* verified live: flipping `ASSISTANT_BACKEND=claude`
changes the recorded backend to `claude` and produces a `failed` record whose
`failure` is `ANTHROPIC_API_KEY is not set in the assistant container`, with a
502 carrying the same words. The plan called upgrading this engine optional
and required only the switch; the switch is real and recorded, and one API key
is all that stands between this and a measured second row.

## cagent — model as explicit config

The model was hardcoded in the committed `config.json.template`. It is now
`__MODEL__`, rendered by `start.sh` from `CAGENT_OPENCODE_MODEL` (default
unchanged, `openai/gpt-5.6-luna`) and printed in the startup line beside the
config and data dirs. Verified by restarting and reading back the rendered
`opencode.json`. Documented in `cagent/README.md`.

Per-request switching was not required and was not added: like `AGENTS.md`,
the model is fixed at OpenCode process start. Cagent still records **no**
per-request cost — OpenCode reports none back to the API — which is exactly
what its card means by "unknown".

## autolab delegate — nothing to do, re-confirmed

`job.yaml`'s `adapter:` is the switch and `adapter_result.json` is the record.
Re-checked against `snake-web-b/evidence/iter-0002`: `total_cost_usd`
0.549051, `num_turns` 19, `duration_ms` 81095, `exit_code` 0, and a
`modelUsage` map naming both models that served it
(`claude-haiku-4-5-20251001`, `claude-sonnet-5`). No change made.

## ENT — the stale binary pointer, fixed at the root this time

Step 2 lost a run to `.local/agent/claude_bin` pinning a version-numbered path
into the VS Code extension directory. **This step hit the identical failure a
second time**, in a different workspace: agforge's `AGFORGE_CLAUDE_CMD` was
pinned at extension 2.1.223 while the installed one is 2.1.226. Two
occurrences in one phase, same mechanical cause, same misleading symptom (`No
such file or directory` — reads as infra, is config).

So it was fixed rather than re-patched: `claude_bin()` now accepts a **glob**
in either `AUTOLAB_CLAUDE_BIN` or the pointer file and resolves the newest
match per call. The machine-specific part stays in `.local/`
(`…/anthropic.claude-code-*-darwin-arm64/…`), and the next extension update
resolves itself. A plain path is still returned as written, deliberately — a
genuinely wrong path must keep failing loudly with the path in the message,
which is what made this diagnosable in one read both times. Three tests pin
it; autolab suite **73 passed**.

Half the fix is still outstanding: **agforge's `AGFORGE_CLAUDE_CMD` has no
resolver** to harden — it flows straight from `local_env` into `build_argv`.
Its value was re-pointed and the trap is now called out in
`agforge/README_DEV.md`, but a third occurrence there is still possible. That
is the obvious next ENT item.

## Notes

- Documentation acceptance ("each agent's docs name the backend switch") is
  met in `agautolab/agent/README.md`, `agforge/README_DEV.md`,
  `agdevworld/README_DEV.md`, and `cagent/README.md` — plus each capability
  card, which is where a *caller* rather than an operator reads it.
- **DEM note**: implemented the assistant's backend seam and cagent's model
  config for those agents — handoff candidate.
