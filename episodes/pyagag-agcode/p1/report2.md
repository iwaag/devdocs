# Report 2 — agautolab → agcode

Status: **done**. agautolab suite green (78 passed). Live checks on agstudio
passed; the agautolab1 half waits for Step 6, as planned.

## Changes

- `agents.toml`: `[profiles.local] harness = "opencode"` → `"agcode"`. Model
  unchanged (`ollama/qwen3.6:35b-a3b-coding-nvfp4`); `native_model` strips the
  `ollama/` prefix for agcode, and the canonical spelling still travels in the
  run record — the live record below shows both.
- Deleted `agent/opencode-{front,mediator,coding,readonly}.json` and
  `opencode.json` (5 files).
- `role_run.py`: `_opencode_config()` and the `opencode_config=` argument are
  gone. In their place, `READONLY_ROLES = {"director", "summarizer"}` and
  `_agcode_args(role)`, which passes `--tools read-only` for those two and
  nothing for everyone else.
- `ROLE_ALLOWED_TOOLS` stays untouched — it is claude_code's grant, and three
  of the five roles are still `sonnet`.
- `README.md`: the OpenCode-permission-files bullet now describes the two
  per-harness grants.
- `.local/agents.local.toml` (ignored): dropped `[local.harness.opencode]`, and
  **changed the ollama base_url** — see below.
- `uv.lock`: agautolab consumes pyagag from GitHub, not as a path dependency,
  so `uv lock --upgrade-package pyagag` moved the pin `00a89808` → `b705fd97`.
  (The plan's "editable path dependency of every consumer" does not hold for
  agautolab; consumers that resolve pyagag from GitHub need this bump in each
  step, and they need the pyagag commit pushed first.)

## The read-only decision: kept, not dropped

The plan left `director`/`summarizer` read-only enforcement to the
implementer. It is kept, and paid for with one small addition to Step 1's
seam: agcode's CLI grew `--tools {default,read-only}` (committed to pyagag as
`b705fd9`), because agautolab drives its harness as a subprocess through
`run_harness()` and the in-process `tools=` argument cannot cross that
boundary. The CLI default is unchanged, so Step 1's "CLI keeps today's
defaults exactly" still holds.

This is strictly tighter than what it replaces. `opencode-readonly.json`
denied `edit`/`write`/`bash`; agcode never offers them.

Verified at the wire (`.local/evidence/agcode-p1/readonly-wire.jsonl`), asking
the `summarizer` role on the `local` profile to create a file:

```
REQ  tools= ['read', 'list']
RESP stop= tool_use   ["thinking", "read"]
RESP stop= end_turn   ["thinking", "text"]
→ "I don't have a tool available to create or write files. I only have the
   ability to read files and list directory contents."
```

No `write` block was ever emitted, and no file appeared in the working
directory.

**One honest caveat.** The same request at the default `--max-turns 20` did
*not* converge: the run ended `aborted: turn_budget_exhausted`, with the model
looping on reads rather than answering. Nothing was written either way, so the
door held — but the plan's hint that "there is no forbidden option to attempt"
predicts a clean early answer, and that is not what a 20-turn budget produced
here. With a 4-turn budget it answered correctly in 2 turns. Read the hint as
"the model cannot do the forbidden thing", not "the model gives up quickly".

## Live checks (agstudio)

Gateway restarted with `launchctl kickstart -k
gui/$(id -u)/com.agdev.agautolab-gateway`, then `POST :8791/window`:

```json
{"id": "window/run-0016", "outcome": "done", "schema": "ag.agent-run.v1",
 "role": "front", "profile": "local", "harness": "agcode",
 "provider": "ollama", "model": "ollama/qwen3.6:35b-a3b-coding-nvfp4",
 "duration_ms": 2031, "num_turns": 2,
 "usage": {"input_tokens": 1462, "output_tokens": 152},
 "reply": "marker-agcode-step2"}
```

The task was "read `probe.txt` and reply with its contents verbatim" against a
marker file dropped in the front workspace, so the reply is checkable rather
than a self-report. `harness: "agcode"` in the record is the migration's own
evidence — Agent ≠ Model, and the record names what served it.

Wire transcript: `.local/evidence/agcode-p1/front-wire.jsonl` (ignored). Its
meta header and both request bodies name exactly one working directory,
`.../agautolab/agent/front`.

agautolab1 is unchanged so far — it runs a deployed checkout and still has
opencode installed. Step 6 redeploys it; the second live `front` run happens
there.

## Two things the next steps need to know

1. **The ollama base_url spelling changed.** The overlay carried
   `http://127.0.0.1:11434/v1`, the OpenAI-compatible path OpenCode wanted.
   agcode posts to `{base_url}/v1/messages`, so that value would have produced
   `/v1/v1/messages`. It is now `http://127.0.0.1:11434`. Every other consumer
   overlay (agdevworld, agforge) and the Ansible-rendered
   `agents.local.toml.j2` still carry the `/v1` form — Steps 3, 4 and 6 must
   fix each one, and Step 7's `nintent_opencode_ollama_url` is the source of
   the deployed value.
2. **`run_harness(transcript_path=)` is not a wire transcript for agcode.** It
   captures the harness's stdout, which for agcode is the one-line result
   document. The real request/response capture is agcode's own `--transcript`,
   reachable through `extra_args`. Both files above were produced that way.
   Anything in Step 5 that wants "keep the wire transcripts" should use
   `--transcript` (or, in-process, `transcript_path=`) rather than the
   `run_harness` argument of the same name.

## Deus Ex Machina note

Did the agcode migration for agent `front` (agautolab) — handoff candidate.
